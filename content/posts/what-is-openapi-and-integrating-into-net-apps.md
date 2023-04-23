---
title: "Making .NET Web API Apps OpenAPI Compliant"
date: 2023-04-16T00:00:00-07:00
draft: false
---

OpenAPI is a specification for building and documenting your RESTful APIs. By following the OpenAPI specification, your RESTful APIs are easier to understand, making them easily consumable by external applications.

## Easy Integration of OpenAPI

For example, in ASP.NET, importing an API that follows the OpenAPI specification is as simple as clicking a few buttons. 

To import the API using Visual Studio 2022, we can right click our project and select Add -> Service Reference
{{< image
src="/what-is-openapi/what-is-openapi-and-integrating-into-net-apps.png"
width="90%"
height="90%"
alt="Screenshot of Visual Studio locating service reference option" >}}

This will redirect us to Connected Reference tab, were we can add a Service Reference
{{< image
src="/what-is-openapi/openapi-selection-option.png"
width="90%"
height="90%"
alt="Screenshot of service reference modal" >}}

Once we fill out the details,
{{< image
src="/what-is-openapi/openapi-selection-option-filled-out-details.png"
width="90%"
height="90%"
alt="Screenshot of service reference modal with details filled out" >}}
a namespace with the classes and methods will be generated on our behalf.

In the above example, we used the API service freely available on [National Weather Service](https://www.weather.gov/documentation/services-web-api). We gave our service reference a name of ``Api.Weather.Gov``. If we look for this namespace in Object Browser, we can view all of the classes and methods generated for us!

{{< image
src="/what-is-openapi/openapi-object-browser-generated-code.png"
width="90%"
height="90%"
alt="Screenshot of generated code from importing service reference" >}}

With this neat trick, much of the boilerplate code for integrating this API has been eliminated. In fact, most of it has been built for us!

## Making a .NET Web API OpenAPI compliant

Making a .NET Web API adhere to OpenAPI specifications is also just as simple. 

In fact, starting with .NET 5, when we create a .NET Web API using either Visual Studio or 
```
dotnet new webapi
```
the OpenAPI code is automatically integrated into our start up application code by default. How sweet is that!

To elaborate more on how .NET 5+ does this, let us analyze the following code.
```csharp
// Required only for minimal APIs
builder.Services.AddEndpointsApiExplorer();

builder.Services.AddSwaggerGen();

app.UseSwagger();
app.UseSwaggerUI();
```

In the above example, we are using Swashbuckle, which can be installed running either
```
Install-Package Swashbuckle.AspNetCore -Version 6.2.3
```
or 
```
dotnet add <YOUR-PROJECT-NAME>.csproj package Swashbuckle.AspNetCore -v 6.2.3
```

With these minor additions, as soon as we start our application and go to /swagger, we will see a beautiful UI that documents our API.
{{< image
src="/what-is-openapi/generated-swagger-ui.png"
width="90%"
height="90%"
alt="Screenshot of generated swagger ui" >}}

Now, we have a standardized API with great documentation that others can easily understand. All updates made in the code will be auto-generated for us by this plugin. This makes it easier for external parties to understand our API, leading to potentially much greater adoption of our services.

There's a lot more to be said about Swashbuckle. To learn how to customize it to your likings, I will redirect to to a well written [article](https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-7.0&tabs=visual-studio-code) by Microsoft.

## Summary

We covered the basics of OpenAPI and how it conforming to this specification makes consuming apps much easier. We showed how simple it is to integrate an API that adheres to OpenAPI into a .NET application. We also discussed how to make a your .NET Web API conform to OpenAPI.

There is one main downside to OpenAPI that we did not mention. That is, OpenAPI is mainly applicable to RESTful services. In later articles, we will address alternative technologies and specification such as GraphQL.

Until then, thanks for reading!