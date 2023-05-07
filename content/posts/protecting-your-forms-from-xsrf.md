---
title: "Protecting Your Forms From Cross Script Request Forgery (CSRF/XSRF) Attacks"
date: 2023-05-07T00:00:00-07:00
draft: false
---

Forms are one of the primary ways users interact with your back-end service. This makes them a vulnerability and extra care should be taken when receiving data from a form.

One key area of concern is the source in which we obtain the form data. We should always verify that form data is coming from a legitimate source. The legitimate source being your site!

When you do not verify the source of the data, you are opening your back-end to cross script request forgery (CSRF/XSRF) attacks. This can have catastrophic consequences such as 3rd parties executing malicious code on behalf of your users without your users' consent!

To illustrate one potential consequence, let's consider an example:

1. Let's assume we host our website at `goodies.com`.

2. A loyal user of our site logs in and begins shopping around. 

3. A few minutes later, while still logged into our site, they visit another website with the following code. Notice that the form's action posts to our site! This is the "cross-site" part of CSRF.:
```html
<h1>Hi! Welcome!</h1>
<form action="https://goodies.com/api/account" method="post">
    <input type="hidden" name="Operation" value="deleteProfile" />
    <input type="submit" value="Click to log in!" />
</form>
```

4. As soon as the user clicks the submit button, the browser will make a request to `goodies.com`, our site!

5. The request will run on our server with the user's authentication context and will perform any action that the authenticated user is allowed to perform! In this case, if their form data was built correctly, it will go ahead and delete the user's profile.

This is why it is extremely important that you always verify the source of the form data.

## Validation Check to Prevent Cross Script Request Forgery (CSRF/XSRF)

One way to verify the source of the form submission is through the is use of a token. This technique is often referred to as the Synchronizer Token Pattern (STP) and works as follows:

1. The server sends a token to the client containing the user's identity, usually in the form of a cookie.
2. When the client submits a form request, the client will include this token with the request, either as a header property or as a part of the form data.
3. The server will accept or deny the request depending on whether the token matches the authenticated user's identity.

Now, if we replay the scenario above, the malicious site will no longer be able to submit requests on behalf of the user. This is because the malicious site request's will not have this token and so they will be rejected by our back-end server. 

This is great! This is how CSRF/XSRF hacks are normally mitigated.

## Preventing CSRF/XSRF in .NET Core applications

Many modern frameworks have built-in protection from CSRF/XSRF attacks. In .NET Core we need to set up a few configuration options and we are set.

First, we need to register one of the following services and middleware into `Program.cs`:
```c#
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

// For MVC applications
builder.Services.AddMvc();

// For Razor Pages applications
builder.Services.AddRazorPages();

// Add middleware to the application

var app = builder.Build();

// For Razor Pages applications
app.MapRazorPages();

// For MVC applications
app.MapControllerRoute();

// For Blazor Server applications
app.MapBlazorHub();

// For SPA applications like Blazor Wasm, React, or Angular
var antiforgery = app.Services.GetRequiredService<IAntiforgery>();

app.Use((context, next) =>
{
    var requestPath = context.Request.Path.Value;

    if (string.Equals(requestPath, "/", StringComparison.OrdinalIgnoreCase)
        || string.Equals(requestPath, "/index.html", StringComparison.OrdinalIgnoreCase))
    {
        var tokenSet = antiforgery.GetAndStoreTokens(context);
        context.Response.Cookies.Append("XSRF-TOKEN", tokenSet.RequestToken!,
            new CookieOptions { HttpOnly = false });
    }

    return next(context);
});
```
After adding the correct service/middleware, your application is ready to generate antiforgery tokens.

The next step involves embedding the antiforgery token within your forms. .NET Core makes this simple. 

As long as your forms contain an include `method="post"` without an `action` attribute, your forms will autogenerate a hidden token. It can also be explicitly add with the HTML helper method `@Html.AntiForgeryToken()`:

```c#
// Implicit
<form asp-action="Index" asp-controller="Home" method="post">
    <!-- ... -->
</form>

// Explicit
<form asp-action="Index" asp-controller="Home" method="post">
    @Html.AntiForgeryToken()
    <!-- ... -->
</form>
```

The final step involves safe-guarding your endpoints. There are two ways to do this: 

1. By using `AutoValidateAntiforgeryToken`:

```c#
// Action  level
[HttpPost]
[AutoValidateAntiForgeryToken]
public IActionResult Index()
{
    // ...
    return RedirectToAction();
}

// Controller level
[AutoValidateAntiforgeryToken]
public class CsrfProtectedController : Controller
{
    // ...
}

// Middleware level
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});
```

2. By using `ValidateAntiForgeryToken`:
```c#
// Action  level
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Index()
{
    // ...
    return RedirectToAction();
}

// Controller level
[ValidateAntiforgeryToken]
public class CsrfProtectedController : Controller
{
    // ...
}

// Middleware level
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new ValidateAntiforgeryTokenAttribute());
});
```

Both work very similar, the main difference being that `ValidateAntiForgeryToken` is more encompassing and applies to both `GET` and `POST` requests. 

> Microsoft's [recommendation](https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery?view=aspnetcore-7.0#automatically-validate-antiforgery-tokens-for-unsafe-http-methods-only) is to apply `AutoValidateAntiforgeryToken` broadly for non-API scenarios. This will ensure that all `POST`s require an antiforgery token by default. The alternative being applying `ValidateAntiForgeryToken` to each individual action method, with the possibility of a `POST` action method left unprotected by mistake, leaving the app vulnerable to CSRF attacks.

And there you have it, your .NET Core application is protected from CSRF/XSRF attacks.


## Summary

Great care should be taken to protect your site from CSRF/XSRF attacks. Luckily, many frameworks have built in support for such attacks.

Unless you have a good reason not to, protect all `POST` endpoints that receive form data.
The least thing you want is exposing your services to CSRF/XSRF attacks. 

Thanks for reading!