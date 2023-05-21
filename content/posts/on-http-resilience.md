---
title: "Making Your HTTP Calls Resilient"
date: 2023-05-21T00:00:00-07:00
draft: false
---

Services have outages. No service can be up 100% of the time. Sometimes the outages are transient, other times, they are persistent and can last for hours.

If your code relies on third-party services, it behoves you to consider these outages as part of the design process. In this post, I would like to discuss how to deal with such outages to ensure your code acts predictably.

## Which Status Codes To Look Out For

A service would ideally reveal its health status in the form of status codes. Common status codes indicative of a struggling service are

```java
// Handle both exceptions and return values in one policy
HttpStatusCode[] httpStatusCodesWorthRetrying = {
   HttpStatusCode.RequestTimeout, // 408
   HttpStatusCode.InternalServerError, // 500
   HttpStatusCode.BadGateway, // 502
   HttpStatusCode.ServiceUnavailable, // 503
   HttpStatusCode.GatewayTimeout // 504
};
```

When receiving these status codes, we should make the effort to call the service at least one more time. The original call may have failed due to a transient error and if so, one more call to the service would have yielded a successful result. However, care should be taken to not overdo retry logic as the service may legitimately be down.

To see how we can easily handle the scenario described in the previous paragraph, I would like to introduce a C# library built around these very concepts: [Polly](https://github.com/App-vNext/Polly).

## Polly, The C# HTTP Resilience Policy Library

Polly is an HTTP resilience library built around policies. We will show how we can handle the above scenario with 2 great resilience policies:

* Retry Policy: For transient and self-correcting faults
* Circuit-breaker Policy: For failing early when a system is seriously struggling

Let's see how we can combine these policies to create HTTP resilient requests. 

We will  begin by installing Polly and associated libraries into our project:

```bash
dotnet add package Polly
dotnet add package Polly.Contrib.WaitAndRetry
dotnet add package Microsoft.Extensions.Http.Polly
```

Now that we have the packages installed, we can showcase how to use retry and circuit breaker logic when sending external calls to a weather service. 

### Weather Service
Let's start by creating our `WeatherService` class:

```java
public interface IWeatherService
{
    Task<Forecast> GetWeatherForecastAsync(string latitude, string longitude);
}
```

```java
public class WeatherService : IWeatherService
{
    private readonly HttpClient _httpClient;

    public WeatherService(HttpClient httpClient)
    {
        _httpClient = httpClient;
        _httpClient.BaseAddress = new Uri("https://api.open-meteo.com/");
    }

    public async Task<Forecast> GetWeatherForecastAsync(string latitude, string longitude) => 
        await _httpClient.GetFromJsonAsync<Forecast>($"/v1/forecast?latitude={latitude}&longitude={longitude}&current_weather=true")
        ?? new Forecast();
    }
```

> This weather service class relies on a free weather api located at [Open-Meteo](https://open-meteo.com/). 

Since we are injecting the HTTP client into our service, we will register this service using the HTTP typed client pattern:

```java
builder.Services.AddHttpClient<IWeatherService, WeatherService>();
```

### Polly Policies

With our service configured, we can start implementing our policies. The 2 policies we are going to configure are for handling transient errors and for stopping requests when a service is seriously struggling. 

The implementation details are as follows:

```java
public static class ResiliencePolicies
{
    public static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy(int retryCount)
    {
        var delay = Backoff.DecorrelatedJitterBackoffV2(medianFirstRetryDelay: TimeSpan.FromSeconds(1), retryCount: retryCount);

        return HttpPolicyExtensions
            .HandleTransientHttpError()
            .WaitAndRetryAsync(delay);
    }

    public static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
    {
        return HttpPolicyExtensions
            .HandleTransientHttpError()
            .CircuitBreakerAsync(5, TimeSpan.FromSeconds(15));
    }
}
```
Let's discuss what each static method accomplishes, beginning with `GetRetryPolicy`. 

The `GetRetryPolicy` method is targeting all requests that return a HTTP status code of 5XX (server errors) or 408 (request timeout). If one of these status codes is returned, the request will be retried according to `DecorrelatedJitterBackoffV2`, which is one of the many elegant, exponential backoff algorithms. Other backoff methods can also be used, such as constant or linear backoffs. The retry count is determined by the variable `retryCount`.

Moving on to the next method, `GetCircuitBreakerPolicy`, this is also targeting all requests that return a HTTP status code of 5XX (server errors) or 408 (request timeout). The difference with this method is that if 5 of these status codes are returned consecutively within 15 seconds, the circuit connection to the service will "open" momentarily. The "open" circuit comes from electronics terminology and indicates that electricity (or http calls in our case) is not allowed to flow. While the circuit is open, all calls to the service will fail with an exception of `BrokenCircuitException`. This will give the service time to recover. Polly will eventually test the health of the service by placing the circuit in a half open state. If no issues occur while the circuit is half open, the circuit will close, allowing traffic to flow as normal.

With the methods created, we can register them alongside our weather service typed client:

```java
builder.Services.AddHttpClient<IWeatherService, WeatherService>()
    .AddPolicyHandler(ResiliencePolicies.GetRetryPolicy(3))
    .AddPolicyHandler(ResiliencePolicies.GetCircuitBreakerPolicy());
```

Now our `WeatherService` class is able to handle faults exhibited by third-party weather service.

 ## Summary

Services can go down. In this article, we discussed techniques to use when a service is down. Sometimes the service can be down momentarily, other times, it can be long lasting. Either way, your code should be resilient enough to handle both cases.

Thanks for reading!