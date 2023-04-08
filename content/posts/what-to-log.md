---
title: "What to Log"
date: 2023-04-07T19:41:09-07:00
draft: true
---

Many web frameworks have the ability to log application and user events details. However, very few explain what information should be logged. In this article, we will discuss what information is useful to log, and which information should not be logged. 

However, before we begin that discussion, let us take a quick detour and start with integrating logging into ASP.NET applications.

## Logging in ASP.NET
Some frameworks make it very easy to integrate logging. Fortunately, for us, ASP.NET is one of those frameworks. 

In ASP.NET, logging is built-in by default. As soon as we create our [HostBuilder](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.host.createdefaultbuilder), default logging providers are registered into the application.
```
using Microsoft.Extensions.Hosting;

using IHost host = Host.CreateDefaultBuilder(args).Build();

// Application code should start here.

await host.RunAsync();
```
The code preceding code above will add the following logging providers by default into the application:
* Console
* Debug
* EventSource
* EventLog (Windows only).

This is great as we have very little to do to get logging to work. Adding additional logging providers is just as easy:
```
var builder = WebApplication.CreateBuilder();
builder.Host.ConfigureLogging(logging =>
{
    logging.AddApplicationInsightsTelemetry();
});

\\ or

var builder = WebApplication.CreateBuilder(args);
builder.Logging.AddApplicationInsightsTelemetry();
```
And that's it, your application is ready to start logging events. Now that we have logging in-place, let us discuss what to log.

## What Information to Log

To determine what information is useful to log, we must first determine what we would like to do with the data. Logging is useful because we can go back in time and replay events as they occurred. This is what makes logging useful. Most of the time, we need to go back in time because some issue occurred in our service.

In other words, we should log any information that will help you later debug an issue. For example, say there was an issue with your text messaging service where user's are claiming that they are not receiving messages.

The first place to see if this is the case is the logs! The logs should have information on the paths taken for each application request. Only then can you determine if an error is actually occurring.

Continuing our text messaging example, if the service did receive the call but is erroring out, our logs should be have enough information to pinpoint the issue.

In summary, we should log enough information to track the progression of an event and any exceptions that may be occurring.

## What Information Not to Log

The question then becomes what should we not log. This is not always as straight-forward, as the more logs we have the better. However, if you log too much data, we will create extra noise that will distract us from the issue at hand. Therefore, we must find a balance.

However, one this that should never be logged is sensitive data, such as api keys and secrets.

## Summary

Logging is crucial to the success of any application. It assists in recording the health of a service and can help in determine holes within the application. 