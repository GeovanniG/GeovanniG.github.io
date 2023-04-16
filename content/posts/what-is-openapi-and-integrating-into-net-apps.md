---
title: "What Is OpenAPI, and integrating into ASP.NET applications"
date: 2023-04-14T14:40:26-07:00
draft: true
---

OpenAPI is a specification for building and documenting your APIs. By following the OpenAPI specification, your APIs are easier to understand and making them easily consumable by external applications.

## Easy Integration of OpenAPI

For example, in ASP.NET, importing an API that follows the OpenAPI specification is a few button clicks at most. 

To import the API using visual studio 2022, we can right click our project and select Add -> Service Reference ![Screenshot of Visual Studio locating service reference option](/what-is-openapi/what-is-openapi-and-integrating-into-net-apps.png)

This will redirect us to Connected Reference tab, were we can add a Service Reference 
![Screenshot of service reference modal](/what-is-openapi/openapi-selection-option.png)

Once we fill out the details,
![Screenshot of service reference modal with details filled out](/what-is-openapi/openapi-selection-option-filled-out-details.png)
we will get a generated namespace with the classes and methods generated according to the OpenAPI specification.

In the above example, we used the API service freely available on [National Weather Service](https://www.weather.gov/documentation/services-web-api). We gave our service reference a name of ``Api.Weather.Gov``. If we look for this namespace in Object Browser, we can view all of the classes and methods generated on our behalf!

![Screenshot of generated code from importing service reference](/what-is-openapi/openapi-object-browser-generated-code.png)

With this neat trick, much of the boilerplate for building our service layer code for integrating this API has been reduced. In fact, most of it has been built for us!

## Making a .NET Web API OpenAPI compliant

Making a .NET Web API OpenAPI adhere to OpenAPI specifications is straightforward. 

In fact, when the Web API is generated for use either via using Visual Studio or 
```
dotnet new webapi
```


## Alternatives to OpenAPI

Before there was OpenAPI, we had WCF