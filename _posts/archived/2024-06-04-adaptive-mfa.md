---
layout: post
permalink: /archive/adaptive-mfa
title: "What is Adaptive Multi-Factor Authentication?"
excerpt: "Adaptive Multi-Factor Authentication (MFA) is an authentication approach that utilizes business rules to decide which security factors to apply in a given situation. Adaptive Authentication is used to enforce security needs but without adding friction to the user experience. Businesses typically use Adaptive Authentication with Multi-Factor Authentication (MFA) and Single Sign-On (SSO)."
tags:
---

Adaptive Multi-Factor Authentication (MFA) is an authentication approach that utilizes business rules to decide which security factors to apply in a given situation. Adaptive Authentication is used to enforce security needs but without adding friction to the user experience. Businesses typically use Adaptive Authentication with <a href="https://en.wikipedia.org/wiki/Multi-factor_authentication" target="blank">Multi-Factor Authentication (MFA)</a> and <a href="https://en.wikipedia.org/wiki/Single_sign-on" target="blank">Single Sign-On (SSO)</a>.


## Adaptive Multi-Factor Authentication (MFA) Examples

### Remote Employees Example
For example, consider a remote employee who is working from home and another one that is working from the office. 
The network of the employee working from home is not as secure as the office's and thus you might want to require two authentication factors for employees working remotely such as SMS and password before they can use the company system. The employee that is working at the office is only required to provide one authentication factor such as their password. This policy of authentication which is based on geo-location (physical location) is an example of adaptive authentication. 

### Business Hours Example
One can set up adaptive MFA rules for the weekends or off-business hours. For example, Alice is trying to log into the system of the company she works  for around midnight. Because that hour is an unusual work hour for employees, an adaptive authentication rule requires her to provide two authentication factors.

### Adaptive Multi-Factor Authentication (MFA) Rules
In addition to geo-location (physical location) and business hours, there are many other factors that businesses consider:
- Day of week
- Trusted device
- User role or group
- IP address
- Authentication failures count

### Security and User Friction
The Adaptive Multi-Factor Authentication (MFA) rules can also be applied to re-authentication. For example, after the employee working from home has successfully authenticated using two factors, trust can be established for their home network IP address, so in the future, the employee is only required to provide one authentication factor such as their username and password to gain access to the company system. This scenario adds security but without adding friction to the user experience. 

## MFA vs Adaptive MFA
- In MFA, access control rules are the same for all users. In adaptive MFA, the access rules vary per user and per situation.
- Adaptive MFA provides better user experience whereas MFA can add more friction to the user experience.

