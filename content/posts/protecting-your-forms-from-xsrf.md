---
title: "Protecting Your Forms From Cross Script Request Forgery"
date: 2023-05-07T00:00:00-07:00
draft: true
---

Forms are one of the primary ways users interact with your back-end service. This makes them a vulnerability and extra care should be taken when receiving data from a form.

One key area of concern is data coming from an untrusted source. We should always verify that data from a from is coming from a legitimate source. The legitimate source being your site!

When you do not verify the source of the data, you are opening your back-end to cross script request forgery (CSRF/XSRF) attacks. This can have catastrophic consequences such as executing malicious code on behalf of the user on your back-end server!

To illustrate this, let's consider an example:

1. Let's say you have a website with a domain of `www.goodies.com`. A loyal user logs into your site. 

2. Unknowingly, they visit another website with the following code:
```
<h1>Hi! Welcome!</h1>
<form action="https://goodies.com/api/account" method="post">
    <input type="hidden" name="Operation" value="deleteProfile" />
    <input type="submit" value="Click to log in!" />
</form>
```
Notice that the form's action posts to your site, not to the malicious site. This is the "cross-site" part of CSRF.

3. The user selects the submit button. The browser will make the request to `www.goodies.com`, your site!

4. The request runs on the `www.goodies.com` server with the user's authentication context and can perform any action that an authenticated user is allowed to perform!

This is why it is extremely important that your back-end service is always verifying where the form data is coming from.

## Validation Check to Prevent Cross Script Request Forgery (CSRF/XSRF)

One way to verify that the form submission is coming from your front-end application is with the use of a special token. This technique is usually referred to as the Synchronizer Token Pattern (STP). This technique works as following:

1. The server sends a token to the client with the current user's identity.
2. When the client submits a form request, the client will include this token to the server.
3. The server will accept or deny the request depending if the token matches the authenticated user's identity.

Now, if we replay the scenario above, the malicious site will no longer be able to submit requests on behalf of the user. This is because the malicious site request's will not have this token and so they will be rejected by your back-end server.

This is great! We have mitigated CSRF hacks.

## Preventing CSRF/XSRF in .NET Core applications

Many modern frameworks have built-in protection from CSRF/XSRF attacks. In .Net Core we need to set up a few configuration options and we are all set.

First, we need to register one of the following services and middleware is added to Program.cs:
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
Adding one of these services, depending on your project, and your application is ready to generate antiforgery tokens.

The next step involves embedding the antiforgery token within your forms.
.Net Core makes this simple. We include `method="post"` without an `action` attribute and 

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