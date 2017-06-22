---
layout: post
title: Create a Wizard in ASP.NET MVC 3
redirect_from:
- "/post/create-wizard-in-aspnet-mvc-3.aspx.html"
- "/post/create-wizard-in-aspnet-mvc-3.aspx/"
tags: [aspnetmvc, aspnet, mvc, wizard, jquery]
---
**[Download Code](/attachments/posts/archived/create-wizard-in-aspnet-mvc-3.zip)**

### Introduction
 Wizards can improve web sites usability by breaking an overwhelming list of questions into multistep simpler forms. A wizard collects information and submits all of it at the end, the user can go back and modify their answers when they want. Because ASP.NET MVC is so flexible, there are many ways to create wizards in MVC. In this post, I will show you a nice technique to create a wizard with the aid of `jQuery`.

The intrinsic idea is to divide a single form by wrapping each set of questions in one container (e.g. `<DIV>`) , and all the rest of the work including navigation is done in javascript. 

### Creating the Wizard

Let's see how we can create this wizard using ASP.NET MVC 3 and Razor. Although you are not restricted to Razor, it's the recommended view engine by many professional web developers because of its fluid, concise, and robust nature.

Click "File->New Project" menu command within Visual Studio to create a new ASP.NET MVC 3 Project. We'll create a new project using the "Internet Application" template. 

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-1.png)

###  Creating the Model

We'll need a model to create our Wizard. Add a class named "User" to the "Models" folder 

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-2.png)

Note that we added some validation using DataAnnotations.

Creating a WizardController

Now right click on the controllers folder and then choose the "Add->Controller" context menu command. This will bring the "Add Controller" dialog:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-3.png)

We'll name the new controller "WizardController". When we click the "Add" button, Visual Studio will add a default "Index" action:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-4.png)

### Creating a View Template

Now we will implement the View associated with the WizardController's Index action. Now to implement the view right-click within the "WizardController.Index()" method and select the "Add View" command to create a view template for our home page: 


![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-5.png)

We will need to modify some of the settings above. Choose the option "Create a strongly-typed view" and choose the User class we created before. Also, choose the "Create" menu item from the "Scaffold template" drop-down box.

When we click the "Add" button, a view template of our "Create" view (which renders the form) looks like the one below:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-6.png)

Our Wizard will have 3 steps. The first ask for Username and email address, the second step asks for First and Last name, and the third asks for Age and Biography. Now we need to wrap every step into a <div> container like the one below:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-7.png)

We'll need to make these containers use class "wizard-step", this will mark these containers so we can track them using javascript later. Also we'll need to hide the steps when the view renders to the end user, so define the following CSS class in the same Index.cshtml template, or you can do it in a .CSS file:


![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-8.png)


We have to add navigation buttons to the template. Add the following two buttons to the end of the template:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-9.png)

### Adding javascript logic

We will need to handle the button clicks on the client side. The "Next" button takes the user to the next step, while the "Back" button takes them to the previous step. The button behind the scenes uses javascript to hide the current step panel (ie <div>) and show the next one, or the previous one (in case of back button).

We will need to add the following code to the .cshtml template:

```javascript
 $(function ()
    {

        $(".wizard-step:first").fadeIn(); // show first step


        // attach backStep button handler
        // hide on first step
        $("#back-step").hide().click(function ()
        {
            var $step = $(".wizard-step:visible"); // get current step
            if ($step.prev().hasClass("wizard-step")) { // is there any previous step?
                $step.hide().prev().fadeIn();  // show it and hide current step

                // disable backstep button?
                if (!$step.prev().prev().hasClass("wizard-step")) {
                    $("#back-step").hide();
                }
            }
        });


        // attach nextStep button handler       
        $("#next-step").click(function ()
        {

            var $step = $(".wizard-step:visible"); // get current step
                        
            var validator = $("form").validate(); // obtain validator
            var anyError = false;
            $step.find("input").each(function ()
            {
                if (!validator.element(this)) { // validate every input element inside this step
                    anyError = true;
                }

            });

            if (anyError)
                return false; // exit if any error found
             
            

            
            if ($step.next().hasClass("confirm")) { // is it confirmation?
                // show confirmation asynchronously
                $.post("/wizard/confirm", $("form").serialize(), function (r)
                {
                    // inject response in confirmation step
                    $(".wizard-step.confirm").html(r);
                });

            }
                        
            if ($step.next().hasClass("wizard-step")) { // is there any next step?
                $step.hide().next().fadeIn();  // show it and hide current step
                $("#back-step").show();   // recall to show backStep button
            }
            
            else { // this is last step, submit form
                $("form").submit();
            }


        });

    });
```

### Creating a POST Action

Whenever the user clicks on the submit button, the controller must handle the request. We will need an action that handles the wizard data. Add a new action like below:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-10.png)

If all data is valid, then we will show the "Complete" template. We will add the "Complete" template by right-clicking within the new POST action, and then choosing "Add View":


![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-11.png)

In the "Complete.cshtml" template, we will just show the following message to the end user:


![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-12.png)

The wizard is ready now. Let's launch the site and try it. Navigate to "/wizard":

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-13.png)

Notice how client-side validation is still working nicely. We did not need to enable client-side validation manually in the template as before, this is because ASP.NET MVC 3 already has it enabled by default. We did also hide the "Back" button on first step using javascript. We cannot submit the form until all information is correct. Even if a user bypasses javascript validation (which can be easily done), the form will be re-displayed again.

### Further Enhancements

Prevent User from Navigating to Next Step when errors found

You may have noticed that the user can navigate to the next step even if the previous step contains errors. Let's prevent the user from navigating to the next step by applying some logic into our javascript code.

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-14.png)

Since client-side validation uses jQuery validation behind the scenes, we obtained a validator's instance and applied validation to every input element on the current step. This will cause alert messages to appear like below:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-15.png)

Now we cannot move forward to step 2 until we enter a valid email address. This won't change the final result as the form won't post back until all fields are OK, however, it will improve user's experience.

### Show a Confirmation Dialog Before Submission

Wizards normally show users a summary of all the details they enter before completion. This will give the user a chance to modify any mistake they made when filling out the form:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-16.png)

If any of the above information is incorrect, we can go back and modify it. This can be handy for large forms, as the user sometimes forgets to fill out some answers.

First, let's add the controller action that we will call to return the template above. In the Wizard controller, add the following action:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-17.png)

Now let's add the "Confirm.cshtml" template that is show in the screenshot above. Right click within the newly added action method, and choose "Add View":

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-18.png)


This time we created a partial view since we will use call the action asynchronously (ie using AJAX). This is a limitation because if we do not use AJAX, all the form data (input fields) will be lost due to the stateless HTTP (There are workaround though). Anyway, let's modify the rendered template so it looks like below:


![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-19.png)

We added the "ValidationSummary" to show errors that cannot be detected on the client-side like duplicate usernames, or bypassed validation rules. If any errors found, the template will show them.

If we will use AJAX to load that template, we will need to inject it somewhere in the main template (Index.cshtml). Let's add it after the final step in the wizard, we will also mark it with both the CSS classes "wizard-step" and "confirm" 


![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-20.png)

The HTML code returned from the "Confirm" action will be injected in that container. The only missing thing now is the javascript code to load the template from the server. Modify the "next step" button by adding the following code:

![](/images/posts/archived/create-wizard-in-aspnet-mvc-3-21.png)

You can download all the code by clicking the link "Download code" at the top.

### Summary

Wizards improve web sites usability by breaking a long list of questions into multistep simple forms. If your site contains such a long form, it is recommended that you use a wizard, especially if it's the registration form. Users will become reluctant if they see a huge list of questions on one page. We learnt how to create a wizard with the aid of javascript. You can do further enhancements to this wizard by adding more navigation links, etc.