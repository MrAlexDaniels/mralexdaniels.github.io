---
title: Advanced Policies With APIM
author: Alex Daniels
date: 2022-09-20 12:10:00 +0800
categories: [API Management (APIM)]
tags: [apim, azure api management]
render_with_liquid: false
---

# Welcome Azure API Management

Azure API Management is a hybrid, multicloud management platform for APIs across all environments. This article provides an overview of common scenarios and key components of API Management.

```xml
<fragment>
	<choose>
		<when condition="@(context.Request.Method == "POST" && !((Jwt)context.Variables["jwt"]).Claims["roles"].Contains("api.post"))">
			<return-response>
				<set-status code="403" reason="Forbidden" />
			</return-response>
		</when>
		<when condition="@(context.Request.Method == "PUT" && !((Jwt)context.Variables["jwt"]).Claims["roles"].Contains("api.put"))">
			<return-response>
				<set-status code="403" reason="Forbidden" />
			</return-response>
		</when>
		<when condition="@(context.Request.Method == "PATCH" && !((Jwt)context.Variables["jwt"]).Claims["roles"].Contains("api.patch"))">
			<return-response>
				<set-status code="403" reason="Forbidden" />
			</return-response>
		</when>
		<when condition="@(context.Request.Method == "GET" && !((Jwt)context.Variables["jwt"]).Claims["roles"].Contains("api.get"))">
			<return-response>
				<set-status code="403" reason="Forbidden" />
			</return-response>
		</when>
		<when condition="@(context.Request.Method == "DELETE" && !((Jwt)context.Variables["jwt"]).Claims["roles"].Contains("api.delete"))">
			<return-response>
				<set-status code="403" reason="Forbidden" />
			</return-response>
		</when>
	</choose>
</fragment>
```