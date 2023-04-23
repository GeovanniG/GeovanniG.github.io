---
title: "Url Encoding: When to Encode Your Urls"
date: 2023-04-22T00:00:00-07:00
draft: true
---

When sending an HTTP request, we normally include may different bits of information. The request is composed of a URL, a method, headers, and at times a body. The question then becomes what data is necessary to encode when submitting HTTP requests.

## Background

There have been many times in my career where we have failed to encode some information in the URL; this resulted in issues such as failing to authenticate a user. On further investigation, it was observed that either the username or password contained special characters. When the username or password were added to the URL as query parameters, e.g. `?username=test&password=!test#`, it was not processed correctly, and the user was not able to login.

## URL Special Characters

In URLs, certain characters have special meanings. For example, `?` and `&` are used for appending query parameters to the URL. Similarly, `#` is used as a fragment identifier that will direct you to a portion of the web page. For a complete list, please visit [URL encoding of special characters](https://documentation.n-able.com/N-central/userguide/Content/Further_Reading/API_Level_Integration/API_Integration_URLEncoding.html).

When submitting any HTTP request, these special characters must be URL encoded if they are used for any reason other then their intended use. Otherwise, the URL will be misinterpreted. For example, if we have an endpoint such as the following
```
GET /users/{username}
```
and we have a username `Tom#e`. We must URL encode this username to `Tom%23e` when submitting this request, otherwise we may have a hard time finding the correct user. 

Note: Modern browsers may encode this username on your behalf, but it's good practice to manually encode the data yourself to ensure compatibility with all browsers and services, especially older services.

As for the headers and body, that data can be left as is and does not need to be encoded. We are free to include spaces and all characters that should normally be encoded, as is. A perfect example of this is the Authorization header, this header contains spaces and special characters with no enoding necessary. The body does however need to conform to the format specified in the `Content-Type` header.

## URL Encoding Example in .NET

In .NET, URL encoding is normally done using the `HttpUtility.UrlEncode` static method. For example to encode the query parameters of a URL, we can use the following method that utitlizes `HttpUtility.UrlEncode`:

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

From my experience, normally the values of the query parameters usually cause URL misinterpretation. The above method should handle this case. Feel free to adjust the method to suit your needs.

## Summary

When submitting HTTP requests, it is necessary to encode the URL. This is especially true when submitting requests from your backend server. Note that web browser do automatically URL encode URLs on your behalf. However, it's worth noting that while browsers will automatically encode URLs, it's still a good practice to manually encode URLs that you generate in your own code, to ensure that they are properly formatted and compatible with all web browsers and servers.

Thanks for reading!