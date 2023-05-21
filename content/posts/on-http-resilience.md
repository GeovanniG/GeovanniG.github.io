---
title: "Making Your HTTP Calls Http Resilient"
date: 2023-05-01T18:37:21-07:00
draft: true
---

Services have outages. No service can be up 100% of the time. Sometimes the outages are transient, other times, they are long lasting and can be down for hours. Other times, a service may not tolerate a certain number of calls and enforce a rate limiter.

If your code relies on third-party services, it behoves you to consider these outages as part of the design process. In this post, I would like to discuss how to deal with such outages to ensure your code acts predictably.

## Which Status Codes To Look Out For

When a service is struggling, ideally it would reveal this to the client in the form of status codes. Common status codes that indicate of a struggling service are

```c#
// Handle both exceptions and return values in one policy
HttpStatusCode[] httpStatusCodesWorthRetrying = {
   HttpStatusCode.RequestTimeout, // 408
   HttpStatusCode.InternalServerError, // 500
   HttpStatusCode.BadGateway, // 502
   HttpStatusCode.ServiceUnavailable, // 503
   HttpStatusCode.GatewayTimeout // 504
};
```

When receiving these status codes, it is best to make the effort to call the service at least once more time. The original call may have failed due to a transient error and if so, one more call to the service will yield a successful result. However, care should be taken to not overdo retry logic as the service may indeed be down.

To see how we can easily handle the above cases, I would like to introduce a C# Library built around these very concepts: [Polly](https://github.com/App-vNext/Polly).

## Polly, The C# HTTP Resilience Policy Library

Polly has 2 great resilience policies for dealing with the above scenario:

* Retry Policy: For transient and self-correcting faults
* Circuit-breaker Policy: For failing early when a system is seriously struggling

Let's see how we can combine these properties to create http resilient requests. Let's begin by installing Polly into our project:

```c#
dotnet add package Polly
dotnet add package Polly.Contrib.WaitAndRetry
dotnet add package Microsoft.Extensions.Http.Polly
```

Now we can create a service 