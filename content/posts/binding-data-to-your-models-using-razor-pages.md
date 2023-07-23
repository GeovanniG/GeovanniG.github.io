---
title: "Binding Data to Your Models Using Razor Pages"
date: 2023-07-23T00:00:00-07:00
draft: false
---

Having worked primarily with ASP.NET MVC, learning Razor pages has been a great pleasure. Razor pages introduce new model binding principles, that are not available in MVC.

In this article, I will discuss 3 ways to bind data to our models using Razor pages.

## 3 Ways to Bind Data to Models

Razor pages allows three main ways to retrieve data from our forms:
* Using the `Request.Form` property,
* Passing the data as parameters, or
* Using `BindProperty` attribute.

To explain how the different methods work, we will use the following Razor page:

```html
@page
@model CarModel
@{
    ViewData["Title"] = "AddCar";
}

<form method="post">
    <section>
        <div asp-validation-summary="ModelOnly" class="text-danger">
            <span>Please correct the following errors</span>
        </div>

        <label for="Brand" class="form-label">Brand: </label>
        <input name="Brand" type="text" class="form-control" />

        <label for="Model" class="form-label">Model: </label>
        <textarea name="Model" class="form-control"></textarea>

        <div class="form-check">
            <input type="checkbox" name="IsOperational" class="form-check-input" />
            <label for="IsOperational" class="form-check-label">Operational</label>
        </div>
    </section>

    <button id="submit-car" class="btn btn-primary">Save</button>
</form>
```

The Razor page will have the following handlers:

```c#
public class AddCarModel : PageModel
{
    private readonly ILogger<AddCarModel> _logger; 

    public AddCarModel(
        ILogger<AddCarModel> logger
    )
    {
        _logger = logger;
    }

    public void OnGet()
    {
        _logger.LogInformation($"Triggered {nameof(OnGet)}");
    }

    public void OnPost()
    {
        _logger.LogInformation($"Triggered {nameof(OnPost)}");
    }
}
```

This page is very simple. When we click the _Save_ button, the `OnPost` handler will fire. We will overwrite the `OnPost` as we introduce the model binding principles.

Let's start our discussion with the `Request` property.

### Model Binding Using the Request Property

When we submit a form, we have access to the request and all of it's properties via the `Request` property. The `Request` property even has access to the form data in the `Request.Form` property.

Using this property, we can retrieve all form data, located within `<input>` tags, that also have a `name` attribute.

Our form created above, satisfies these conditions. For example, here is the form data we have access to:

```html
<input name="Brand" type="text" class="form-control" />
<textarea name="Model" class="form-control"></textarea>
<input type="checkbox" name="IsOperational" class="form-check-input" />
```

To access the data, we can pass the `name`'s value to the `Request.Form` array. The following example, shows how to retrieve the form data:

```c#
public class AddCarModel : PageModel
{
    private readonly ILogger<AddCarModel> _logger; 

    public AddCarModel(
        ILogger<AddCarModel> logger
    )
    {
        _logger = logger;
    }

    public void OnGet()
    {
        _logger.LogInformation($"Triggered {nameof(OnGet)}");
    }

    public void OnPost()
    {
        _logger.LogInformation($"Triggered {nameof(OnPost)}");
        _logger.LogInformation($"Brand: {Request.Form["Brand"]}, Model: {Request.Form["Model"]}, IsOperational: {Request.Form["IsOperational"]}");
    }
}
```

As you may have guessed, using this method is very error prone. If we enter the wrong key, we receive a `null` value in exchange. Working with string is also very cumbersome, especially since the keys are case sensitive. To handle these issues we can use an alternative approach.

### Model Binding Using Parameters

Another approach we can take, is binding our form data to function parameter's. Similarly, the parameters will bind to `input` fields that have a `name` attribute.

With the example given earlier, we can read the data, using the following `OnPost` method:

```c#
public class AddCarModel : PageModel
{
    private readonly ILogger<AddCarModel> _logger; 

    public AddCarModel(
        ILogger<AddCarModel> logger
    )
    {
        _logger = logger;
    }

    public void OnGet()
    {
        _logger.LogInformation($"Triggered {nameof(OnGet)}");
    }

    public void OnPost(string brand, string model, bool isOperational)
    {
        _logger.LogInformation($"Triggered {nameof(OnPost)}");
        _logger.LogInformation($"Brand: {brand}, Model: {model}, IsOperational: {isOperational}");
    }
}
```

A great benefit of using this approach, is that, we do not need to cast our data. When retrieving data using `Request.Form`, we are given data as `StringValues`, and it is our responsibility to cast the objects. Using this approach, the data is automatically casted by middleware.

The downside of this approach, is if we decide to add a more form data, we will have to update our `OnPost` method with new parameters. This is unmaintainable if our form is comprised of many data fields. To resolve this, we have one more approach to consider.  

### Model Binding Using the BindProperty Attribute

To remove the model binding dependency from our `OnPost` handlers, we can take advantage of the `BindProperty` attribute.

Using this approach has many upsides. For one, we can make use of tag helpers such as `asp-for`, and `asp-validation-for`, if you are using jQuery unobtrusive validation. Using these methods, we can statically type into our Razor pages. 

For example, if we introduce a `CarModel` with the following properties:

```c#
public class CarModel
{
    public string Brand { get; set; }
    public string Model { get; set; }
    public bool IsOperational { get; set; }
}
```
We can rewrite and improve our forms using tag helpers:

```html
@page
@model CarModel
@{
    ViewData["Title"] = "AddCarModel";
}

<form method="post">
    <section>
        <div asp-validation-summary="ModelOnly" class="text-danger">
            <span>Please correct the following errors</span>
        </div>

        <label asp-for="Car.Brand" class="form-label">Brand: </label>
        <input asp-for="Car.Brand" type="text" class="form-control" />
        <span asp-validation-for="Car.Brand" class="text-danger"></span>

        <label asp-for="Car.Model" class="form-label">Model: </label>
        <textarea asp-for="Car.Model" class="form-control"></textarea>
        <span asp-validation-for="Car.Model" class="text-danger"></span>

        <div class="form-check">
            <input type="checkbox" asp-for="Car.IsOperational" class="form-check-input" />
            <label asp-for="Car.IsOperational" class="form-check-label">Operational</label>
            <span asp-validation-for="Car.IsOperational" class="text-danger"></span>
        </div>
    </section>

    <button id="submit-car" class="btn btn-primary">Save</button>
</form>
```

When we generate our forms, the `asp-for` located within the `input` tags will be converted into `name` attributes. 

To complete the binding, we must add our model to the Razor page, with an attribute of `BindProperty`. This will instruct Razor pages to bind the form data to our model.

```c#
public class AddCarModel : PageModel
{
    private readonly ILogger<AddCarModel> _logger;

    [BindProperty]
    public CarModel Car { get; set; } = new();

    public AddCarModel(
        ILogger<AddCarModel> logger
    )
    {
        _logger = logger;
    }

    public void OnGet()
    {
        _logger.LogInformation($"Triggered {nameof(OnGet)}");
    }

    public void OnPost()
    {
        _logger.LogInformation($"Triggered {nameof(OnPost)}");
        _logger.LogInformation($"Brand: {Car.Brand}, Model: {Car.Model}, IsOperational: {Car.IsOperational}");
    }
}
```

When binding data to our models, this is the preferred approach. This approach combines the best of the two previously discussed approaches. We take advantage of static analysis properties, while also removing the interdependency from our handlers.

The downside being that the developers must be familiar with how `BindProperty` works, and must also have some knowledge with Razor pages.

Thanks for reading!