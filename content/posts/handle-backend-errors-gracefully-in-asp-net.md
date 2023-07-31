---
title: "Handle Backend Errors Gracefully In ASP.NET"
date: 2023-07-30T00:00:00-07:00
draft: false
---

There is no way around it, your users will experience errors within your application. How you handle these errors will show your users how professional and robust your application really is. In this article, we will delve into a few techniques for handling errors, gracefully.

## Built-in Error Handling

ASP.NET provides many options for handling backend errors gracefully. In fact, all projects built using the default ASP.NET project template have error handling middleware incorporated, by default. If we navigate to the `Program.cs` file, we see the following middleware:

```c#
...
if (!app.Environment.IsDevelopment())
{
    ...
    _ = app.UseExceptionHandler("/Error");
    ...
}
...
```

This middleware is very simple. Assuming we are not in development, when the application receives an error, that was not handled correctly by the application, the user will be redirected to the __Error__ page. This is a great first step. 

Using this approach, we are not revealing any application errors to the user. If we give the user too much context on the error, they may be able to use it to their advantage, and explicitly seek out vulnerabilities within our system. 

However, a downside to this approach, is the errors can be too generic. If the error is transient or self-correcting, performing the action once more may resolve the issue altogether. In this case, displaying a user-friendly error message is preferred.

## Displaying User-Friendly Error Messages

To display a more user-friendly error message, we have the option of accessing the exception, using [IExceptionHandlerPathFeature](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-7.0&preserve-view=true#access-the-exception). For example, if a file is not found exception is thrown, we have the option of filtering on this exception, and displaying an appropriate error message to the user:

```c#
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
[IgnoreAntiforgeryToken]
public class ErrorModel : PageModel
{
    public string? RequestId { get; set; }

    public bool ShowRequestId => !string.IsNullOrEmpty(RequestId);

    public string? ExceptionMessage { get; set; }

    public void OnGet()
    {
        RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier;

        var exceptionHandlerPathFeature =
            HttpContext.Features.Get<IExceptionHandlerPathFeature>();

        if (exceptionHandlerPathFeature?.Error is FileNotFoundException)
        {
            ExceptionMessage = "The file was not found.";
        }
    }
}
```

```html
@page
@model ErrorModel
@{
    ViewData["Title"] = "Error";
}

<h1 class="text-danger">Error.</h1>
<h2 class="text-danger">An error occurred while processing your request.</h2>

@if (Model.ShowRequestId)
{
    <p>
        <strong>Request ID:</strong> <code>@Model.RequestId</code>
    </p>
}

<p>
    <strong>Error Message:</strong> <code>@Model.ExceptionMessage</code>
</p>
```

## Displaying Custom Error Message

We can take the above method one step further. Instead of using `IExceptionHandlerPathFeature` directly within the page model, we can move it into its own static class. 

For example, we can create a custom error handler class around `IExceptionHandlerPathFeature`, and filter out the exception that was thrown. Knowing the exception, we can then display an appropriate error message to the user:

```c#
public static class CustomExceptionHandlerPathFeature
{
    private static readonly Dictionary<Type, Func<Exception, ProblemDetails>> _exceptionHandlers;

    static CustomExceptionHandlerPathFeature()
    {
        // Register known exception types and handlers.
        _exceptionHandlers = new()
            {
                { typeof(ValidationException), HandleValidationException },
                { typeof(NotFoundException), HandleNotFoundException },
                { typeof(UnauthorizedAccessException), HandleUnauthorizedAccessException },
                { typeof(ForbiddenAccessException), HandleForbiddenAccessException },
            };
    }

    public static ProblemDetails TryHandle(IExceptionHandlerPathFeature exceptionHandlerPathFeature)
    {
        var exceptionType = exceptionHandlerPathFeature.Error.GetType();

        if (_exceptionHandlers.TryGetValue(exceptionType, out var value))
        {
            return value.Invoke(exceptionHandlerPathFeature.Error);
        }

        return HandleDefaultException(exceptionHandlerPathFeature.Error);
    }

    private static ProblemDetails HandleValidationException(Exception ex)
    {
        var exception = (ValidationException)ex;

        return new ValidationProblemDetails(exception.Errors)
        {
            Status = StatusCodes.Status400BadRequest,
            Title = "Validation Errors",
            Type = "https://tools.ietf.org/html/rfc7231#section-6.5.1",
            Detail = DictToString(exception.Errors, "{0}='{1}' ")
        };
    }

    private static string DictToString<T, V>(IEnumerable<KeyValuePair<T, V>> items, string format)
    {
        format = string.IsNullOrEmpty(format) ? "{0}='{1}' " : format;
        return items.Aggregate(new StringBuilder(),
            (sb, kvp) => sb.AppendFormat(CultureInfo.InvariantCulture, format, kvp.Key, kvp.Value)).ToString();
    }

    private static ProblemDetails HandleNotFoundException(Exception ex)
    {
        var exception = (NotFoundException)ex;

        return new()
        {
            Status = StatusCodes.Status404NotFound,
            Type = "https://tools.ietf.org/html/rfc7231#section-6.5.4",
            Title = "The specified resource was not found.",
            Detail = exception.Message
        };
    }

    private static ProblemDetails HandleUnauthorizedAccessException(Exception _) => new()
    {
        Status = StatusCodes.Status401Unauthorized,
        Title = "Unauthorized",
        Type = "https://tools.ietf.org/html/rfc7235#section-3.1",
        Detail = "Please login and try again."
    };

    private static ProblemDetails HandleForbiddenAccessException(Exception _) => new()
    {
        Status = StatusCodes.Status403Forbidden,
        Title = "Forbidden",
        Type = "https://tools.ietf.org/html/rfc7231#section-6.5.3",
        Detail = "Forbidden access."
    };

    private static ProblemDetails HandleDefaultException(Exception _) => new()
    {
        Status = StatusCodes.Status500InternalServerError,
        Title = "Internal Server Error",
        Type = "https://tools.ietf.org/html/rfc7231#section-6.6.1",
        Detail = "We are experiencing issues."
    };
}
```

To use this static class, we will need to make improvements to our page model:

```c#
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
[IgnoreAntiforgeryToken]
public class ErrorModel : PageModel
{
    public string? RequestId { get; set; }

    public bool ShowRequestId => !string.IsNullOrEmpty(RequestId);

    public string? ExceptionMessage { get; set; }

    private readonly ILogger<ErrorModel> _logger;

    public ErrorModel(ILogger<ErrorModel> logger)
    {
        _logger = logger;
    }

    public ActionResult OnGet()
    {
        HandleError();
        return Page();
    }

    public ActionResult OnPost()
    {
        HandleError();
        return Page();
    }

    public void HandleError()
    {
        RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier;
        var problemDetails = CustomExceptionHandlerPathFeature.TryHandle(
            HttpContext.Features.Get<IExceptionHandlerPathFeature>());
        ExceptionMessage = problemDetails.Detail;
    }
}
```

With this simple change, we can display a more direct error message when a user receives an error. If the error message is informative enough, the user may figure out the appropriate steps to resolve the issue on their own.

The potential downside to this approach is that we must register each exception of interest within `CustomExceptionHandlerPathFeature`. This may be a time-consuming process at first. Still, if the error messages are meaningful and descriptive enough, the time initially invested, may pay itself back, in the form of fewer support tickets.

Thanks for reading!