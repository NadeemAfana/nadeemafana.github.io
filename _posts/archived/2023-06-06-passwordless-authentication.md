---
layout: post
title: What is Passwordless Authentication? Passwordless Authentication Explained
excerpt: "This post explains what passwordless authentication is, how it works and its advantages as well as Google passkeys."
tags:
- passwordless
- passkeys
- auth
---

<span style="display: none;" class="excerpt">This post explains what passwordless authentication is, how it works and its advantages as well as Google passkeys.</span>

<a href="https://app.upend.cc" target="_blank"><img alt="" src="/images/posts/archived/upend-ad.svg"></a>

## What is Passwordless Authentication?
Passwordless authentication is the process of verifying the identity of a user without using a password.
Passwordless authentication is a safer and easier alternative to passwords.
Passwordless is a very generic term, and there are many different strategies and implementations that revolve around it which makes it sound overwhelming. 

## Is passwordless better than password?
Passwords get breached, shared, reused, and are hard to manage and remember for the end users. 
Additionally, 63% of data breaches involved leveraging weak, stolen or default passwords.
Passwords also add conversion friction to your app(s) as users must remember them or store them, and not to mention rotate them regularly.
With the increasing amount of data breaches involving credentials, IT security is looking for alternatives and thus organizations are slowly moving away from passwords. 

Passwords are more susceptible to cyberattacks. For example, <a href="https://en.wikipedia.org/wiki/Credential_stuffing" target="_blank">credential stuffing</a> is a form of cyberattack in which a bad actor obtains stolen usernames and passwords from one organization to access user accounts at another. The credentials are normally stolen during a data breach or purchased from the dark web. <a href="https://en.wikipedia.org/wiki/Phishing" target="_blank">Phishing</a> is another cyberattack where a bad actor sends SMS messages or emails to trick a victim into replying with their credentials.

## How does passwordless authentication work?
There are different methods of authenticating users without requiring a password. 
For example, magic links. A link is emailed to the user that will authenticate them when it is opened. 
Another passwordless method is sending users a one-time security code to log in. Because the security code is generated dynamically and is valid for a limited time range, it minimizes the opportunity of account hijacking.

Open standards such as W3C <a href="https://www.w3.org/TR/webauthn-2/" target="_blank">WebAuthn</a> and <a href="https://fidoalliance.org/specs/fido-v2.0-id-20180227/fido-client-to-authenticator-protocol-v2.0-id-20180227.html" target="_blank">CTAP2</a> exist today to help us migrate to the passwordless authentication world. I suspect it will be a while before we transition. 

WebAuthn relies on <a href="https://en.wikipedia.org/wiki/Public-key_cryptography" target="_blank">Public-key cryptography</a>. A public and private key pair (aka credential) is generated on the user side.
The private key is bound to the user and device being used whereas the public key is shared with the public such as a third-party application the user is trying to log into. 
Note the credential or key pair is unique to the user device and the third party application. 
If the same user wants to log into another third-party application, they will need another credential. 
This makes the credential only usable for a particular application. This security feature is inherently implemented by the browser and/or operating system.
Furthermore, a hacker cannot obtain the private key from the public key itself. 

Companies including Microsoft and Google are adopting passwordless authentication. For example, Microsoft Windows Hello uses WebAuthn to log in users, and Microsoft Edge browser stores the private key securely in Windows Hello on Windows.

Google on the other hand uses <a href="https://developers.google.com/identity/passkeys"  target="_blank">passkeys</a>. 
A passkey is a specialized use case of WebAuthn. It is a credential, tied to a user account and a website or application. 
The passkey is stored securely in different places depending on the OS. For example, the passkey is stored in Google Password Manager on Android and stored in Windows Hello on Windows.

The WebAuthn credential does not have to be stored locally on that device, and in that case, it is called a "roaming authenticator" as it can roam between different devices. The communication between the device and the remote authenticator is done using CTAP2 protocol which also entails transports such as NFC and Bluetooth.


In WebAuthn, 
**client** is the user. 
A **relying party** is the application that the user is trying to register for or sign into. Usually, the interaction is mediated via a browser.
The **client device** is the hardware that hosts WebAuthn implementation. Laptops and phones are examples of client devices. In this instance, the browser device is the client device. 
A **client device** and a **client** together make up a **client platform**.

An **authenticator** is the software or hardware that can register the user and verify the credential. 
The authenticator is responsible for storing the credential and signing the challenge with the private key when needed.
An authenticator is generally assumed to have a single user to it.  
An authenticator can be a **platform authenticator** which resides on the same client device (eg browser device). Examples of platform authenticators include fingerprint recognition technology that is built-into a laptop. 
An authenticator can also be a **roaming authenticator**  which can roam between different client devices using cross-platform transports.
In this instance, the authenticator is a roaming authenticator which is the client phone device because it is separate from the browser (client) device.

The following diagram illuistrates how all the pieces fit together.

![](/images/posts/archived/passwordless-auth-webauthn.svg)

1. The user attempts to log into an external app
1. The app through the browser UI prompts the user for their existing WebAuthn credentials. The app stores the public key, and the user has the private key stored securely on their device (in this case, it is stored on their phone, their roaming authenticator device. In other cases, it could well be stored on the same device as the browser.)
1. The app sends some information including a challenge to the user's browser. 
1. The browser asks the authenticator to sign the challenge which will trigger some notification on the user's phone device. If the user consents, the browser forwards the signed assertion to the app to validate. 
1. Once the app validates the signature using the public key, the user is considered authenticated.

## What is the difference between 2FA and passwordless?
They are two different things. Two-factor authentication (2FA) is an identity and access management security method that requires two forms of identification to access resources and data. These two forms of identification could be passwordless or not.


## Is passwordless authentication bad?
Even with passwordless authentication, some attacks can still happen. For example, one-time passcodes (OTPs) can be intercepted using malware. Also, infected web browsers with viruses can be used to intercept magic links.


## Finally
Reduce the risk of your exposure by switching to passwordless authentication.


## See also 
- <a href="https://app.upend.cc" target="_blank">Low-code passwordless solution for web apps</a>
- <a href="https://developers.google.com/identity/passkeys" target="_blank">Google Passkeys</a>
- <a href="https://www.w3.org/TR/webauthn-2" target="_blank">WebAuthn</a>
- <a href="https://fidoalliance.org/specs/fido-v2.0-id-20180227/fido-client-to-authenticator-protocol-v2.0-id-20180227.html"  target="_blank">CTAP2</a>