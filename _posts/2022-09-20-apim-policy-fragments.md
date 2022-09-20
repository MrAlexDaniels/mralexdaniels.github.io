---
title: Policy Fragments With APIM
author: Alex Daniels
date: 2022-09-20 12:10:00 +0800
categories: [Azure API Management, APIM]
tags: [apim, azure api management]
render_with_liquid: false
---


In this article we are going to talk about policy fragments and use cases. Let's start by understanding what a policy fragment is. 

Policy fragments help you configure policies consistently and maintain policy definitions without the need to repeat or retype XML code. 


## Scenario 
In this policy we are looking at a JWT token and checking the roles that were included in the token. We are limiting the request methods they can perform. This policy is not going to be added to the All APIs level, but it must be applied to 3/7 of the APIs. Before the release of policy fragments, we would have had to duplicate this policy across all 3 APIs. When you have duplicate policies we must ensure any changes take place in all the duplicated area, leaving room for error and inconsistency. To fix this we can use a policy fragment! One policy that can be reused across multiple APIs.  

![alt text](/img/policy_frag.png)


## Creating a Policy Fragment
We are going to name the fragment and drop the policy from the image above into it.

![alt text](/img/policy_frag2.png)

After the creation of it we can use the fragment anywhere we need it. Allowing for us to edit in one location instead of multiple. 

```xml
<include-fragment fragment-id="request-jwt-roles" />
```

![alt text](/img/policy_frag3.png)


## Limitations
* A policy fragment can't include a policy section identifier (`<inbound>`, `<outbound>`, etc.) or the `<base/>` element.
* Currently, a policy fragment can't nest another policy fragment.