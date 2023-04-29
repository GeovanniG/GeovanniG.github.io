---
title: "Dry Patterns in ASP.NET Core, Removing Boilerplate Code"
date: 2023-04-02T16:25:35-07:00
draft: false
---

Let’s discuss patterns to help keep your ASP.NET code nice and DRY. 

In most applications, there is code that is normally copied and pasted into multiple locations within the codebase. We refer to this code as boilerplate code.

In this article, I would like to discuss techniques to remove this boilerplate code and centralize it into as few locations as possible.

After all, if we ever do have to change the boilerplate code it's better to change it in as few locations as possible. This not only reduces the likelihood of introducing new bugs, but also makes all developers’ lives easier by improving the codebases’ readability and maintainability.

## Middleware
Our first technique to reduce boilerplate code is to add the code as middleware.

Functionality added as middleware will execute on each and every request made to your application. As a result, if you have code that should be executed for every request, a great location for this code would be in the middleware. 

Common functionality that is normally added to middleware includes exception handling and logging.

ASP.NET makes this very simple by providing built in middleware for such cases:

```csharp
// If an unhandled exception occurs in the subsequent middleware components, the middleware will catch the exception, log it, and return a custom error response with the URL /error.
app.UseExceptionHandler("/error");

// The middleware will log information about the incoming request and the outgoing response, as well as any other information that you want to log.
app.UseLogging();
```

This removes the need to add try/catch blocks to each and every controller:

```csharp
public bool TestEndpoint()
{
    try 
    {
        // do something
        return true;
    }
    catch (Exception ex)
    {
        _log.LogError(ex, "Meaningful message");
    }
    return false;
}
```
and can be replaced with the following:

```csharp
public bool TestEndpoint()
{
    // do something
    return true;
}
```

This keeps your controllers lightweight and gives them a more defined purpose as they no longer have to deal with exception handling.

There's a lot more to be said about middleware. We can create our own, or use many of the other built-in middlewares provided. But the key takeaway is, if the code needs to execute for every request, the best place for that code to reside would be as middleware.

## Filters
Another option to remove boilerplate code is to use Filters. What separates a filter from middleware is that a filter can be applied on an as-needed basis. They do not need to be applied globally -- unlike Middleware, although they can be -- but are normally applied on certain action methods or controllers as needed.

For example, the following filter can validate if the user is a certain age. It uses reflection and assumes the controller has a parameter named ``age``:
```csharp
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
This filter serves a specific purpose, but filters can be made as generic or specialized as needed and applied accordingly. 

## Higher-Order Functions
One downside, or upside depending on how you look at it, is that middleware and filters do not have direct access to the code in which they are decorating.

For example, if we apply a filter an action method, the filter does not have access to parameters or data within the method. The only workaround is to use reflection, like we did in the above example, but doing so will make our filter less reusable and make the code more cumbersome to work with.

A great alternative to this situation is to use higher-order functions. The above method can be transformed into the following and wrapped in a ``Func``:
```csharp
public IActionResult VerifyAge(int age, Func< IActionResult> func)
{
    if (age < 18)
    {
        // Return an error message
        return context.Result = new ContentResult()
        {
            StatusCode = (int)HttpStatusCode.BadRequest,
            Content = "Age must be greater than 18"
        };
    }
    return func();
}

public IActionResult Index(string name, int age)
{
    VerifyAge(age, () => View());
}
```
This technique does take some time to get used to but allows you to take full advantage of type safety features.

## Records
One last technique to remove boilerplate code is to use ``records``, introduced in C# 9. ``records`` remove the need to declare properties within the ``record`` and allow you to specify them upfront when creating the ``record``. 

For example, say we have a ``Person`` ``class`` such as the following:

```csharp
public class Person 
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
}
```

As a ```record```, it can be transformed into

```csharp
public record Person(string FirstName, string LastName, int Age);
```

Now, these 2 representations are not 100% equivalent, as ```records``` are immutable by default, however, when your ``class`` is immutable by design, ```records``` are a great alternative.

Note: One great thing to look out for in C# 12 is partial constructors. Partial constructors will allow us to represent ``Person`` as:
```csharp
public class Person(string FirstName, string LastName, int Age);
```
And will even allow us to inject interfaces into partial constructors:
```csharp
public class Car(ITire Tire);
```
This is a feature I am looking forward to, as it removes the need to declare private readonly variables for all injected services:
```csharp
private readonly ITire _tire;

public class Car(ITire tire)
{
    _tire = tire;
}
```

## Conclusion

In this article we described 4 techniques to remove boilerplate code. The less boilerplate code we have in our applications, the easier it will be to read and maintain. 

If you have anymore techniques to reduce boilerplate code, please let us know.

Thanks for reading!