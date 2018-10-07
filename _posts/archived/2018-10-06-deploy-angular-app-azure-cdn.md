---
layout: post
title: Deploying Angular App using CircleCI to Azure CDN with no Downtime
tags:
- angular
- azure
- cdn
- javascript
- circleci
- angularcli
---
## Deploying Angular App using CircleCI to Azure CDN with no Downtime
Recently, I migrated an Agular app from Azure Web App to Azure CDN from Verizon. There are important benefits to using CDN over a regular web app such as cost and delivery speed. I shall save the migration process to another post.

In this post, I am going to build a CircleCI script that deploys an Angular app to Azure CDN with no downtime. Of course, the same behavior can be achieved using a different CI system.


The deployment script has the following features
* **Deploys files to Azure Blob container** (I assume that's where the site is hosted).
* **Purges the CDN cache** after deployment to provide fresh content to the end users.
* **Auto-deletes old assets** from Azure Blob Storage. As you might already know, Angular CLI fingerprints assets, and over time the old assets will linger in the Azure Blob container.

## How It Works
The key to avoiding a downtime boils down to the order in which the files are deployed. First, the `index.html` file needs to be deployed last. The reason for that is the following: Suppose you are deploying your site assets and only some of them have been deployed so far including `index.html`, and some user tries to access the site at that moment, they might get 404 because the new `index.html` references assets that are not available yet from the CDN.

## CircleCI config.yml
The CircleCI `config.yml` file looks like the following. You might have to adjust it to match your deployment settings:
```
version: 2
jobs:
  build:
    working_directory: ~/my-project
    docker:
      - image: circleci/node:8.11-browsers
    steps:
      - setup_remote_docker
      - checkout
      - restore_cache:
          key: npm-dependency-cache-{{ checksum "package.json" }}
      - run:
          name: Install npm packages
          command: npm install
      - save_cache:
          key: npm-dependency-cache-{{ checksum "package.json" }}
          paths:
            - node_modules
      - run:
          name: Test
          command: npm test -- --watch=false
      - store_artifacts:
          path: coverage
          prefix: coverage
      - run:
          name: Build and Deploy
          command: |
            echo Installing AzureCLI...
            echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
            sudo tee /etc/apt/sources.list.d/azure-cli.list
            curl -L https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
            sudo apt-get install apt-transport-https
            sudo apt-get update && sudo apt-get install azure-cli
            
            if [[ "${CIRCLE_TAG}" == *release* ]]
            then
                echo Releasing to Live...
                azureConnectionString=${PRODUCTION_STORAGE_CONNECTIONSTRING}
                userName='c53f1a9b-a334-4152-ad19-c9612e715a35'
                resourceGroup='my-project-live-rg'
                cdnEndpointName='my-project-live-cdn'
                cdnProfileName=$cdnEndpointName
                subscription='Prod Subscription'
                npm run build -- --prod
            elif [[ "${CIRCLE_BRANCH}" == "master" ]]
            then
                echo Releasing to Staging...
                azureConnectionString=${STAGING_STORAGE_CONNECTIONSTRING}
                userName='14898d6c-bae1-4c5c-b55a-9caf1121de96'
                resourceGroup='my-project-staging-rg'
                cdnEndpointName='my-project-staging-cdn'
                cdnProfileName=$cdnEndpointName
                subscription='Staging Subscription'
                npm run build -- -c=staging
            elif [[ "${CIRCLE_BRANCH}" == "test" ]]
            then
                echo Releasing to Test...
                azureConnectionString=${TEST_STORAGE_CONNECTIONSTRING}
                userName='dfb87956-4959-46c1-b63b-aa82969c769a'
                resourceGroup='my-project-test-rg'
                cdnEndpointName='my-project-test-cdn'
                cdnProfileName='my-project-test-cdn-profile'
                subscription='Test Environment'
                npm run build -- -c=test
            fi

            # Avoid a downtime by uploading index.html last.
            mv ./dist/index.html ./index.html
            az storage blob upload-batch --source ./dist --destination site --connection-string $azureConnectionString
            mv ./index.html ./dist/index.html
            az storage blob upload --file ./dist/index.html --container-name site --name 'index.html'  --content-cache-control "no-cache" --connection-string $azureConnectionString
            echo Deleting files older than 3 months...
            oldFilesDateTime=$(date -d "-3 months"  +"%Y-%m-%dT%H:%MZ")
            az storage blob delete-batch --source site --if-unmodified-since $oldFilesDateTime  --connection-string $azureConnectionString
            echo Purging CDN caches...
            az login --service-principal --username "$userName" --password $CircleCIServicePrincipalPassword --tenant "3e64d903-d42e-4a8d-8c6d-430299e3b8d8"
            az account set --subscription "$subscription"
            foldersToFlush=$(cd ./dist && find ./ -type d | awk '{print $0 suffix}' suffix="/*" | tr  '\n' ' ')
            foldersToFlush=$(echo $foldersToFlush | sed 's/\.//g' | sed 's/\/\//\//g')
            az cdn endpoint purge --resource-group $resourceGroup --name $cdnEndpointName --profile-name $cdnProfileName  --content-paths $foldersToFlush

            
workflows:
  version: 2
  build_and_test:
    jobs:
      - build:
          filters:
            branches:
              only: 
                - master
                - test
            tags:
              only: 
                - /release-.*/
```

The script above supports three environments: Test, Staging and Live but these do not matter. 
I will explain the relevant pieces of the code one by one. 

First, make sure you have all the environment variables set as needed (eg `${PRODUCTION_STORAGE_CONNECTIONSTRING}`). Additionally, you need an Azure service principal in order to deploy the site. This can be created easily in Azure. 

Here is an example that creates a service principal using Azure CLI
```
az ad sp create-for-rbac --name CircleCIServicePrincipal --password 19gxxc3@@8h44
```
**NOTE**: You might need to create one service principal for each subscription.

Then, AzureCLI is installed:
```
echo Installing AzureCLI...
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
sudo tee /etc/apt/sources.list.d/azure-cli.list
curl -L https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo apt-get install apt-transport-https
sudo apt-get update && sudo apt-get install azure-cli
```

Now, the angular project is built using Angular CLI for the current environment.
```
echo Releasing to Live...
azureConnectionString=${PRODUCTION_STORAGE_CONNECTIONSTRING}
userName='c53f1a9b-a334-4152-ad19-c9612e715a35'
resourceGroup='my-project-live-rg'
cdnEndpointName='my-project-live-cdn'
cdnProfileName=$cdnEndpointName
subscription='Prod Subscription'
npm run build -- --prod
```
In the code above, a few variables are set for the next phase. Always store sensitive information such as connection strings in environment variables. 

**NOTE**: The `userName` variable holds the service principal `appId` value.

```
# Avoid a downtime by uploading index.html last.
mv ./dist/index.html ./index.html
az storage blob upload-batch --source ./dist --destination site --connection-string $azureConnectionString
mv ./index.html ./dist/index.html
az storage blob upload --file ./dist/index.html --container-name site --name 'index.html'  --content-cache-control "no-cache" --connection-string $azureConnectionString
```
This deploys all the files under the `dist` folder to Azure Blob and makes sure `index.html` is deployed last. The code  assumes your site content is inside the `site` container in Azure Blob storage. This can be changed easily.

If a user hits the site before the new `index.html` is uploaded, they will NOT get an error because the old version is still there along with all its assets (I noticed images do not get finger printed but these are highly unlikely to cause a downtime for a site).

Notice that  `index.html` has `no-cache` value set for the `Cache-Control` header. This is important for the users to always get the fresh version of that file.

I mentioned that some files will linger since Angular CLI fingerprints assets. Although there is no need for cleanup, I personally like to keep the Azure Blob storage clean. In my case, I delete all assets older than 3 months. This value can be adjusted to suit your needs. However, to avoid a downtime, I recommend keeping at least the two most recent versions.0

```
echo Deleting files older than 3 months...
oldFilesDateTime=$(date -d "-3 months"  +"%Y-%m-%dT%H:%MZ")
az storage blob delete-batch --source site --if-unmodified-since $oldFilesDateTime  --connection-string $azureConnectionString
```

In order for the end users to see the immediate results, the CDN cache must be purged.
```
echo Purging CDN caches...
az login --service-principal --username "$userName" --password $CircleCIServicePrincipalPassword --tenant "843dce7c-ff36-4164-a9aa-0616af27aeeb"
az account set --subscription "$subscription"
foldersToFlush=$(cd ./dist && find ./ -type d | awk '{print $0 suffix}' suffix="/*" | tr  '\n' ' ')
foldersToFlush=$(echo $foldersToFlush | sed 's/\.//g' | sed 's/\/\//\//g')
az cdn endpoint purge --resource-group $resourceGroup --name $cdnEndpointName --profile-name $cdnProfileName  --content-paths $foldersToFlush
```
The value of `tenant` comes from the service principal.
The code above simply enumerates all the folders under `./dist` and feeds them into the Azure CLI command since there is no bulk purge feature in Azure CLI yet.