---
title: "Using a CDN to Improve the Performance of a .Net Core Project"
date: 2023-06-25T00:00:00-07:00
draft: false
---

Have you ever noticed that the default layout, `<project-root>/Shared/_Layout.cshtml`, always contains the following file references:

```html
...
<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
...
<script src="~/lib/jquery/dist/jquery.min.js"></script>
<script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
...
```

These HTML tags are referring to JQuery and Bootstrap files located under `wwwroot`:

```
wwwroot
└───lib
    └───jquery
        └───dist
                jquery.min.js
    └───bootstrap
        └───dist
            └───css
                    bootstrap.bundle.min.css
            └───js
                    bootstrap.bundle.min.js
```

As long as the stylesheets and scripts are located under `wwwroot`, this approach works well. However, JQuery and Bootstrap are not the most lightweight downloads, and as the size of your application grows, we will eventually need to consider alternative approaches.

In fact, according to Bundlephobia, [bootstrap@5.3.0](https://bundlephobia.com/package/bootstrap@5.3.0) can take 6x the amount to of time download when compared to [react@18.2.0](https://bundlephobia.com/package/react@18.2.0), and [jquery@3.7.0](https://bundlephobia.com/package/jquery@3.7.0) can take up to 10x the amount of time to download! This is not to scare you away from JQuery or Bootstrap, but more so to inform you on the impact these files can have on performance. Luckily,  we do have options to improving the browser download speed of these libraries.

## Retrieving Static Files From a CDN

One great way to improve the browser download speed of JQuery and Bootstrap is to use a CDN. When using a CDN, these static assets will be downloaded from the closest edge server to the user. So instead of obtaining the files from our servers, the files will be obtained from a CDN that should be much closer to the user. Given that the proximity of the files are closer, the files should theoretically download much faster. Another great benefit is that the work is offload from our server, reducing blockage of other scripts and allowing our server to process more user requests.

A great benefit of using a CDN for JQuery and Bootstrap, is that we have many available options. For our example, we will demonstrate how to retrieve JQuery and Bootstrap from the CDN [`cdnjs`](https://cdnjs.com/). 

Instead of the above HTML, we can replace the code with the following:

```html
...
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.0/css/bootstrap.min.css"
    asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
    asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
...
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.0/jquery.min.js" 
    asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
    asp-fallback-test="window.jQuery"
    crossorigin="anonymous"
    integrity="sha384-NXgwF8Kv9SSAr+jemKKcbvQsz+teULH/a5UNJvZc6kP47hZgl62M1vGnw6gHQhb1"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.0/js/bootstrap.bundle.min.js"
    asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"
    asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
    crossorigin="anonymous"
    integrity="sha384-geWF76RCwLtnZ8qwWowPQNguL3RmwHVBC9FhGdlKrxdiJJigb/j/68SIy3Te4Bkz"></script>
```

In this example, we are retrieving JQuery and Bootstrap from [`cdnjs`](https://cdnjs.com/) and have provided fallback options in case our application experiences issues fetching the files from the CDN. 

To elaborate further, the stylesheets are using the following fallback options:
* `asp-fallback-href`: the location of the fallback stylesheet
* `asp-fallback-test-class`: the css class to test. This test class should only exist in the stylesheet loaded by `href`
* `asp-fallback-test-property`: the css property within the css test class, `asp-fallback-test-class`
* `asp-fallback-test-value`: the value the css test property must equal. This will verify the css loaded correctly, otherwise, it will load the stylesheet in `asp-fallback-href`.

Similarly, for scripts, we are using the following fallback options:
* `asp-fallback-src`: the location of the fallback script
* `asp-fallback-test`: the test to perform to validate the script loaded correctly, otherwise, it will load the script in `asp-fallback-src`
* `integrity` & `crossorigin`: to validate the integrity of CDN scripts. This will verify that the scripts have not been tampered with. To learn more visit [srihash.org](https://www.srihash.org/)

With this configuration, we are now taking full advantage of a CDN. The CDN will allow the static assets to download much quicker, leading to quicker performance of our application, and hopefully, a more satisfied customer.

## Retrieving Static Files From wwwroot, Development Setup

While developing our application, it would preferred to use the static assets located in `wwwroot`. These files are hosted directly on our machines and hence should download many times faster then from the CDN.

For this purpose, ASP.NET has an [`environment`](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/environment-tag-helper?view=aspnetcore-7.0) tag helper in which we can specify what HTML to render based on the current hosting environment.

With this approach, we can render the scripts from `wwwroot` while in `Development`, and when not in `Development`, our static assets will be loaded from a CDN:

```html
...
<environment include="Development">
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
</environment>
<environment exclude="Development">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.0/css/bootstrap.min.css"
        asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
        asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
</environment>
...
<environment include="Development">
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
</environment>
<environment exclude="Development">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.0/jquery.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery"
            crossorigin="anonymous"
            integrity="sha384-NXgwF8Kv9SSAr+jemKKcbvQsz+teULH/a5UNJvZc6kP47hZgl62M1vGnw6gHQhb1"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.0/js/bootstrap.bundle.min.js"
            asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"
            asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
            crossorigin="anonymous"
            integrity="sha384-geWF76RCwLtnZ8qwWowPQNguL3RmwHVBC9FhGdlKrxdiJJigb/j/68SIy3Te4Bkz"></script>
</environment>
...
```

This approach will yield the best of both worlds in each respective environment.

## Summary

In this article we described how to improve the performance of our site by using a CDN to load commonly available libraries. We also showed how to continue to use the files within `wwwroot` when in development. 

If your application is loading 3rd party static assets from `wwwroot`, you may want to research if the libraries are hosted on a CDN. This has the potential to greatly improve the speed of your application, thereby improving the satisfaction of your customers.

Thanks for reading!