---
title: "Minify and Bundle Static Assets In ASP.NET Core"
date: 2023-06-18T00:00:00-07:00
draft: false
---

As the internet evolves and improves, so do user's expectations. Users are now expecting web pages to load within a reasonable amount of time. If they do not, users will gladly take their business elsewhere.

In this article, I will describe a simple technique to improve the speed of your application. We will be focusing on static files (e.g. JavaScript or CSS) and show how to minify and bundle them so your application can load and retrieve them faster.

## What are Bundling and Minification?

Bundling and minification are static asset performance optimizations with different roles. Bundling is the act of concatenating multiple static assets into a single asset thereby reducing the number of calls a browser needs to make to a server. Minification is a separate process of reducing the size of a static asset by removing comments, renaming variables, etc. Combined together, these two techniques have the potential of decreasing loading times of static assets by 50%+!

## Setting Up a Project For Bundling And Minification

At the time of this writing, ASP.NET Core doesn't provide a native bundling and minification solution. However, they do recommend the use of an open-source bundling and minification package called [WebOptimizer](https://github.com/ligershark/WebOptimizer). We will use this package and explain how to incorporate it into our project to take advantage of bundling and minification.

Let's begin by install the NuGet package into our project. For convenience, we will use the `dotnet` command line tool to install the NuGet package `LigerShark.WebOptimizer.Core`:

```
dotnet add package LigerShark.WebOptimizer.Core 
```

Next, we need to add the WebOptimizer services into our services container. This is accomplished by adding `AddWebOptimizer` to `IServiceCollection`:

```c#
public void ConfigureServices(IServiceCollection services)
{
    ...
    var serviceProvider = services.BuildServiceProvider();
    var env = serviceProvider.GetRequiredService<IWebHostEnvironment>();

    if (env.IsDevelopment())
    {
        _ = services.AddWebOptimizer(minifyJavaScript: false, minifyCss: false);
    }
    else
    {
        _ = services.AddWebOptimizer();
    }
    ...
}
```

For debugging purposes, we have decided to not minify static assets while in development.

Thirdly, we add the WebOptimizer as middleware. We do this by calling `UseWebOptimizer` on `IApplicationBuilder`:

```c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...
    app.UseWebOptimizer();
    app.UseStaticFiles();
    ...
}
```

Please note that order matters; `UseWebOptimizer` must come before `UseStaticFiles`. With the middleware added, static files will be minified and bundled before they are submitted to the client.

Lastly, we register the `@addTagHelper *, WebOptimizer.Core` Tag Helpers into _ViewImports.cshtml:
```c#
@addTagHelper *, WebOptimizer.Core
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

These tag helpers will target the `<script>` and `<link>` tags and enable cache busting. Cache busting will grant features such as hot-reloading and will remove outdated static assets from the client's computer for new assets when a change in the asset is detected. 

With the above steps completed, our project is setup for minification. We can run a quick test by adding comments to one of our existing JavaScript files.

{{< image
src="/minify-and-bundling/minified-javascript.png"
width="90%"
height="90%"
alt="Screenshot of minified javascript using WebOptimizer" >}}

Notice how the minified files have extracted much of the extra white space and have also removed all comments!

With minification added to our project, we will now move onto bundling our static assets.

## Bundling Static Assets

As described above, bundling is the act of concatenating multiple files into a single static file. Bundling can be accomplish by adding `AddJavaScriptBundle` or `AddCssBundle` to the WebOptimizer pipeline.

```c#
services.AddWebOptimizer(pipeline =>
{
    pipeline.AddJavaScriptBundle("/js/tests-combined.js", "/js/test.js", "/js/test-2.js");
});
```

In the code above, we are combining `/js/test.js` and `/js/test-2.js` into a single minified and bundled file called `/js/tests-combined.js`:

{{< image
src="/minify-and-bundling/bundled-javascript.png"
width="90%"
height="90%"
alt="Screenshot of bundled javascript using WebOptimizer" >}}

With bundling and minification combined, we now have the tools necessary to greatly reduce the load time of our website.

## Conclusion

Bundling and minifying your static assets is a great and simple way to improve the load time of your application. In this article we showed how to incorporate such tactics using [WebOptimizer](https://github.com/ligershark/WebOptimizer). This package takes much of the work, and with a few configuration properties, our project is ready to take advantage of these techniques. By incorporating these techniques, you improve the speed of your application and showcase to your visitors that you take their time seriously.

Thanks for reading!