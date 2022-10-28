---
title: Creating APIM API using Azure Function Swagger/OpenApi
author: Alex Daniels
date: 2022-10-28 12:10:00 +0800
categories: [Azure API Management, APIM, AAD, AZURE FUNCTIONS]
tags: [apim, azure api management, azure functions]
render_with_liquid: false
---

# What are we trying to accomplish? 
When importing an API into APIM with swagger/OpenApi it helps to have everything as detailed as possible to eliminate manual changes after import. To do this we can decorate our swagger as much as possible, in this article we will cover how to use OpenApi decorators in Azure Functions. 

## Lets look at the problem
As you can see in the image below, we are missing a few details that may be handy for a consumer of this API to know. The field `name` doesn’t have a description. 
Wouldn’t it be nice if you could put that into the swagger.json file instead of manually entering it within APIM? How do we mark a field as required? 

![alt](/img/apim_swagger/DevPortal1.png)


## Walkthrough
If you have not already created an Azure Function project, I would recommend you do so to be able to walkthrough this as easy as possible. 

In this scenario we will be using Visual Studio and .NET Functions

### Let's get started by creating a new Azure Function

Not required but I would suggest creating one with OpenApi already configured. 
![alt](/img/apim_swagger/vs1.png)

Click Add and we should see some generated code looking like the image below.

```c#
[FunctionName("Function3")]
[OpenApiOperation(operationId: "Run", tags: new[] { "name" })]
[OpenApiSecurity("function_key", SecuritySchemeType.ApiKey, Name = "code", In = OpenApiSecurityLocationType.Query)]
[OpenApiParameter(name: "name", In = ParameterLocation.Query, Required = true, Type = typeof(string), Description = "The **Name** parameter")]
[OpenApiResponseWithBody(statusCode: HttpStatusCode.OK, contentType: "text/plain", bodyType: typeof(string), Description = "The OK response")]
public async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req)
{
    _logger.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    string responseMessage = string.IsNullOrEmpty(name)
        ? "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."
        : $"Hello, {name}. This HTTP triggered function executed successfully.";

    return new OkObjectResult(responseMessage);
}
```

I am going to skip and configure this to look how I want my endpoint. 

```c#
[FunctionName("PostCharactersEndpoint")]
[OpenApiOperation(operationId: "POST Character", tags: new[] { "Character" })]
[OpenApiRequestBody("application/json", typeof(CharacterModel), Description = "Creating a new Character", Required = true)]
[OpenApiResponseWithBody(statusCode: HttpStatusCode.OK, contentType: "text/plain", bodyType: typeof(CharacterModel), Description = "This is my description for 200 response documentation")]
[OpenApiResponseWithBody(statusCode: HttpStatusCode.BadRequest, contentType: "text/plain", bodyType: typeof(ValidationResult), Description = "ValidationError Response")]
public async Task<IActionResult> RunPost(
    [HttpTrigger(AuthorizationLevel.Function, "post", Route = "character")] HttpRequest req)
{
    _logger.LogInformation("C# HTTP trigger function processed a request.");
    var body = await req.GetBodyAsync<CharacterModel>();
    return new OkObjectResult(body.Value);
}
```

So whats going on here? 

We are naming the `operationId` to be anything but `run`. The reason being is we'll have conflicts when deploying to APIM if every endpoint we create has the same name. 

____________

```c#
[OpenApiRequestBody("application/json", typeof(CharacterModel), Description = "Creating a new Character", Required = true)]
```

We are telling swagger the request body for this `POST` is `JSON` and we are defining that body with `CharacterModel` (We will circle back to `CharacterModel` in a bit) and we have a basic description

____________
```c#
tags: new[] { "Character" }` 
```

Tagging is a way for us to put these endpoints into categories that will be used within the APIM Developer Portal. 

____________
```c#
[OpenApiResponseWithBody(statusCode: HttpStatusCode.OK, contentType: "text/plain", bodyType: typeof(CharacterModel), Description = "This is my description for 200 response documentation")]
```

If the operation returns a 200 this is the model we expect back

__________

```c#
[OpenApiResponseWithBody(statusCode: HttpStatusCode.BadRequest, contentType: "text/plain", bodyType: typeof(ValidationResult), Description = "ValidationError Response")]
```

If that status code is a bad request we are going to be returning ValidationResults

_________

Now we have the tricky task of assigning descriptions to each field in our `POST` model and if that field should be required or not. From the example above we are expect `CharacterModel` to be the `POST Body`.

Let's take a look at that. 

```c#
public class CharacterModel
{
    [JsonRequired]
    [JsonProperty(PropertyName = "name")]
    public string Name { get; set; } = string.Empty;

    [JsonProperty("birth_year")]
    public string BirthYear { get; set; } = string.Empty;

    [JsonProperty("eye_color")]
    public string EyeColor { get; set; } = string.Empty;

    [JsonProperty("gender")]
    public string Gender { get; set; } = string.Empty;

    [JsonProperty("hair_color")]
    public string HairColor { get; set; } = string.Empty;

    [JsonProperty("height")]
    public string Height { get; set; } = string.Empty;

    [JsonProperty("mass")]
    public string Mass { get; set; } = string.Empty;

    [JsonProperty("skin_color")]
    public string SkinColor { get; set; } = string.Empty;
}
```

The first this you might see is `[JsonRequired]`, this is how to define in our swagger to say which fields are required and which ones are not. In order to create a description for our `Name` field we are going to need a little extra code. I'm also going to add a few more `[JsonRequired]`.

```c#
[JsonRequired]
[JsonProperty(PropertyName = "name")]
[OpenApiProperty(Description = "This is the field description for the Name field in my model class")]
public string Name { get; set; } = string.Empty;
```
<sup>Note -- the namespace for this is going to be `Microsoft.Azure.WebJobs.Extensions.OpenApi.Core.Attributes;`</sup>

_________

### Deploy the functions project and update APIM. 

To create/update APIM use the following Powershell command
```powershell
az apim api import --api-id {} --resource-group {} --service-name {} --path /starwars --specification-format OpenApi --specification-url foofoo.azurewebsites.net/api/swagger.json
```

_______

### The Result!
Inside the Developer Portal you should see something that looks like the image below.

- The endpoint is now under the `Character` tag on the left side and directly under the endpoint name.
- `name, birth_year, gender, hair_color` are all marked as required fields.
- `name` now has a description to it.
 
![alt](/img/apim_swagger/DevPortal2.png)

