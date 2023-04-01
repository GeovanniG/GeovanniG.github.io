---
title: "Dry Patterns in Asp.Net"
date: 2023-03-26T16:25:35-07:00
draft: true
---

# Dry Patterns in ASP.NET
Let’s discuss patterns to help keep your ASP.NET code nice and DRY. In most applications, there is code that is normally copied and pasted into multiple locations within the codebase. We like to refer to this common set of code as boilerplate code.

In this article, I would like to discuss techniques to remove this boilerplate code and centralize it into as few locations as possible.

After all, if we ever do have to change the boilerplate code it's better to change it in as few locations as possible. This not only reduces the likelihood of introducing new bugs but also makes all developers’ lives easier by improving the codebases’ readability and maintainability.

## Middleware
Functionality added as middleware will execute each and every request made to your application. As a result, if you have code that should be executed for every request, a great location for this code would be in the middleware. 

Common functionality that is normally added to middleware includes exception handling and logging.

ASP.NET makes this very simple by providing built in middleware for such cases:

```
// If an unhandled exception occurs in the subsequent middleware components, the middleware will catch the exception, log it, and return a custom error response with the URL /error.
app.UseExceptionHandler("/error");

// The middleware will log information about the incoming request and the outgoing response, as well as any other information that you want to log.
app.UseLogging();
```

This removes the need to add try/catch blocks to each and every controller:

This keeps your controllers and gives them a more defined purpose as they no longer have to deal with exception handling.

There's a lot more to be said about middleware. We can create our own, or use many of the built-in middlewares provided to us. But the key takeaway is, if the code needs to execute for every request, the best place for that code to reside would be as middleware.

## Filters
Another option to remove boilerplate code is to use Filters. What separates a filter from middleware is that a filter can be applied on an as-needed basis. They do not need to be applied globally, unlike Middleware, although they can be, but are normally applied on certain action methods or controllers as needed.

```
public class VerifyAgeFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // Check if age parameter is present
        if (context.ActionArguments.ContainsKey("age"))
        {
            // Get the value of age parameter
            var age = (int)context.ActionArguments["age"];
            // Verify that age is greater than 18
            if (age < 18)
            {
                // Return an error message
                context.Result = new ContentResult()
                {
                    StatusCode = (int)HttpStatusCode.BadRequest,
                    Content = "Age must be greater than 18"
                };
                return;
            }
        }

        base.OnActionExecuting(context);
    }
}

[VerifyAgeFilter]
public IActionResult Index(string name, int age)
{
    return View();
}
```

## Higher-Order Functions
One downside, or upside depending on how you look at it, is that middleware and filters do not have access to the code in which they are decorating.
For example, if we apply the filter to the following action method, the filter does not have access to parameters or data within the method. The only workaround is to use reflection, but doing this will make our filter less reusable and make the code more cumbersome to work with.


A great alternative to this situation is to use higher-order functions.

## Pipelines (MediatR)
The last option relies on a well-known package called MediatR. MediatR was created on the CQRS principles. 

