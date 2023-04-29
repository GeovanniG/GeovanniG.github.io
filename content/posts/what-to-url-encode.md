---
title: "URL Encoding: When to Encode Your URLs"
date: 2023-04-23T00:00:00-07:00
draft: false
---

When sending an HTTP request, we normally include may different bits of information. The request is composed of a URL, a method/verb, headers, and at times a body. The question then becomes what data is necessary to encode when submitting HTTP requests.
  
## Background

There have been many times in my career where we have failed to encode some information in the URL; this resulted in issues such as failing to authenticate a user. On further investigation, it was observed that either the username or password contained special characters. When the username or password were added to the URL as query parameters, e.g. `?username=test&password=!test#`, it was not processed correctly, and the user was not able to login.

## Special Characters In a URL

In URLs, certain characters have special meanings. For example, `?` and `&` are used for appending query parameters to the URL. Similarly, `#` is used as a fragment identifier that will direct you to a portion of the web page. And these are just a few examples, for a more complete list, please visit [URL encoding of special characters](https://documentation.n-able.com/N-central/userguide/Content/Further_Reading/API_Level_Integration/API_Integration_URLEncoding.html).

When submitting an HTTP request, if these special characters are used for any reason other then their intended use, they must be URL encoded. If they are not, the URL will be misinterpreted. For example, if we have an endpoint such as the following
```
GET /users/{username}
```
and we have a username `Tom#e`. We must URL encode this username to `Tom%23e` when submitting this request, otherwise we may have a hard time finding the correct user. 

Note: Modern browsers may encode this username on your behalf, but it's good practice to manually encode the data yourself to ensure compatibility with all browsers and services, especially older services.

## URL Encoding Example in .NET

In .NET, URL encoding is normally done using the `HttpUtility.UrlEncode` static method. For example, to encode the query parameters of a URL, we can use the following method that utitlizes `HttpUtility.UrlEncode`:

```csharp
static class UrlUtilities
{
    public static string EncodeQueryParameters(string url)
    {
        string[] urlParts = url.Split('?');

        if (urlParts.Length == 1)
        {
            return url;
        }

        string baseUrl = urlParts[0];
        string queryString = urlParts[1];

        string encodedQueryString = string.Join("&", queryString
            .Split('&')
            .Select(x => string.Concat(
                HttpUtility.UrlEncode(x.Split('=')[0]),
                "=",
                HttpUtility.UrlEncode(x.Split('=')[1])
            ))
        );

        string queryEncodedUrl = string.Concat(baseUrl, "?", encodedQueryString);

        return queryEncodedUrl;
    }
}
```
Here we are spliting the URL by the starting query parameter and using Linq to URL encode the query parameter key-value pairs.

Using this method in an example,
```csharp
string url = "https://example.com/path/to/page?param1=value 1&param2=value@2";
var urlEncodedParameters = UrlUtilities.EncodeQueryParameters(url);

Console.WriteLine("Original URL: " + url);
Console.WriteLine("Encoded URL: " + urlEncodedParameters);
```
yields the following output:
```csharp
Original URL: https://example.com/path/to/page?param1=value 1&param2=value@2
Encoded URL: https://example.com/path/to/page?param1=value+1&param2=value%402
```
Now that the URL has been encoding, it will be correctly processed by the service.

## Summary

When submitting HTTP requests, it is necessary to encode the URL, otherwise, our requests may be misunderstood. This is especially true when submitting requests from a more legacy system. Some modern frameworks and web browsers will automatically URL encode URLs on your behalf. However, it's still a good practice to manually encode URLs that you generate to ensure that they are properly formatted and compatible with all web browsers and servers.

In this post we did not discuss the header and body of a request. The good news is that these can be left as is and do not need to be encoded. We are free to include spaces and all characters that should normally be encoded in a URL, as is. A perfect example of this is the Authorization header. This header contains spaces and special characters with no encoding necessary. The body does however need to conform to the format specified in the `Content-Type` header.

Thanks for reading!