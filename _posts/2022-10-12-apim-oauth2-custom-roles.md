---
title: Using AAD For Custom Roles In APIM
author: Alex Daniels
date: 2022-10-12 12:10:00 +0800
categories: [Azure API Management, APIM, AAD]
tags: [apim, azure api management, active directory]
render_with_liquid: false
---

# What are we trying to accomplish? 
When we want to limit what a consumer of an API can or cannot do we can accomplish this by creating custom roles in RBAC and issue them within our token. Once issued in our token we can read and apply these roles from within our APIM polcies. 

## Use case
A user has access to consume API endpoints but we do not want them to be able to perform `DELETE` operations. By creating custom roles we are able to limit this access. 

## Walkthrough
If you have not already created a backend app registration amd configured it with APIM please check out this resource to do so.
[Protect API's using OAuth 2.0 in APIM](https://techcommunity.microsoft.com/t5/azure-paas-blog/protect-api-s-using-oauth-2-0-in-apim/ba-p/2309538 )

### App Registration
Within the app registration created for our APIM enviroment we need to configure a few things. 

This is where we can create our custom roles that are assigned to each user. In this example we are creating roles that grant access to `GET`, `POST`, `PATCH`, `PUT`, `DELETE`, `ADMIN` 

![alt text](/img/AppReg1.png)

### Enterprise Application
Now lets go into the Enterprise Application for our app. From here we can assign a role to a user/group. 
- Add user/group
- Select user(s) or group(s)
- Select role
- Assign

![alt text](/img/EnterpriseAppReg1.png)

We now have everything configured for the token to pass these roles!

## Getting/Viewing the token 
I found the easiest way to grab the issued token is to get it from the Developer Portal. From here we can use the Authorization to generate a Bearer Token that we can take a peek at. Copy the token and lets go to [https://jwt.ms](https://jwt.ms/) to take a look at it! 

![alt text](/img/DevPortal1.png)

<sup>Note: Remove "Bearer" from the token when pasting into [https://jwt.ms](https://jwt.ms/)</sup>

![alt text](/img/Jwt1.png)

If everything is configured correctly you should not see the roles assigned to that user.

## Limiting Access with a Policy

![alt text](/img/PolicyRoles.png)

We have a couple important things going on here. First, we are validating the Bearer Token. Second we are outputting the token as a variable called `jwt`.

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid." output-token-variable-name="jwt">
```

Now that we have the token stored as a context variable we can access it. Using `context.Variables` 


So what are we doing? What does this line of code do? 
```xml
<when condition="@(context.Request.Method == "POST" && !((Jwt)context.Variables["jwt"]).Claims["roles"].Contains("api.post"))">
    <return-response>
        <set-status code="403" reason="Forbidden" />
    </return-response>
</when>
```


We are looking at the `Request.Method`, if its a `POST` we need to check the token roles to determine if the user can perform that action. 
```xml
((Jwt)context.Variables["jwt"]).Claims["roles"].Contains("api.post"))
```

If the user does not have the correct role they will get a 403 Forbidden returns to them. 


![alt text](/img/DevPortal403.png)