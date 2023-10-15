---
title: "Feature Management in C# Blazor Wasm"
date: 2023-10-15T00:00:00-07:00
draft: false
---

In Microsoft's tutorial [Use feature flags in an ASP.NET Core app](https://learn.microsoft.com/en-us/azure/azure-app-configuration/use-feature-flags-dotnet-core?tabs=core6x#mvc-views), they outline how to use feature flags in an MVC application.

Nowhere is there mention of using feature flags in a Blazor Wasm application. In this project, we will create a `FeatureRender` component. The component is inspired by the MVC tag helper `feature`.

To see how the inner workings of the component, please see: [BlazorFeatureManagement](https://github.com/GeovanniG/BlazorFeatureManagement#feature-management-in-blazor)