---
title: "Logging Basics: What to log"
date: 2023-04-09T19:41:09-07:00
draft: false
---

Many web frameworks have the ability to log application and user event data. However, very few explain what information should be logged. In this article, we will discuss what information is useful to log and what information should never be logged. 

Before we begin our discussion, let us take a quick detour and start with integrating logging into ASP.NET applications.

## Logging in ASP.NET Core
Some frameworks make it very easy to integrate logging. Fortunately, for us, ASP.NET is one of those frameworks. 

In ASP.NET, logging is built-in by default. As soon as we create our [HostBuilder](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.host.createdefaultbuilder), default logging providers are registered into the application.
```csharp
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

This is great as we have very little to do to get logging to work. Adding additional logging providers is also just as simple:
```csharp
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

To determine what information to log, we must first determine what we would like to do with the logs. Logging is useful because we can go back in time and replay events as they occurred. Most of the time, we need to go back in time to diagnose some issue that occurred in our service.

In other words, we should always log any information that will later help us debug an issue. 

For example, say there was an issue with our text messaging service where we are receiving complaints that messages are not being sent out.

The first place to verify this would be the logs! The logs should have enough information on the paths taken for each application request. This will allow us to track the exact path taken by the requests. Only then can you determine if an error is occurring.

Continuing our text messaging example, if the service did receive the call but it is erroring out, our logs should be have enough information to pinpoint the root cause. Preferably, the logs should have the stacktrace, if there is any.

In summary, we should log enough information to track the progression of an event and any exceptions that may be occurring.

## What Information Not to Log

The question then becomes what should we not log. This is not always be as straightforward as the more logs we have the better. However, if we log too much data, we will create extra noise that will distract us from the issue at hand. Therefore, we must find a balance.

However, there is one thing that should never be logged and that is sensitive data. Sensitive data such as api keys and secrets should remain as secrets.

Logs are widely and easily accessible by developers. Therefore, logs should never contain any sensitive, private, or personal information.

Logs are for assisting in diagnosing issues and auditing application events, not for storing sensitive data.

## Summary

Logging is crucial to the success of any application. They assist in recording the health of a service and can help in determine weaknesses within the application. Nonetheless, logs are not a place to store sensitive information. Sensitive information should be stored in secure areas, far away from application logs.