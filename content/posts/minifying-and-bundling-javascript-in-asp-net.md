---
title: "Minify and Bundle Javascript and CSS In ASP.NET Core"
date: 2023-06-18T00:00:00-07:00
draft: true
---

As the internet evolves and improves, so do user's expectations. With so many options, users are now expecting web pages to load within a reasonable amount of time. If they do not, they will gladly take their time elsewhere to a website that values their time.

In this article, I will describe a technique to improve the speed of your site. We will be focusing on static files (e.g. JavaScript or CSS) and how to minify and bundle them so your application loads them faster and reduces the amount of round-trips to your web server.

## Bundling and Minification

Bundling and minification are performance optimizations with different roles. Bundling is the act of concatenating multiple static assets into a single asset thereby reducing the number of calls a browser needs to make to a server. Minification is a separate process of reducing the size of a static asset by removing comments, renaming variables, etc. Combined together, these two techniques have the potential of decreasing loading times of such static assets by 50%+!

## Setting Up a Project For Bundling And Minification

At the time of this writing, ASP.NET Core doesn't provide a native bundling and minification solution. However, they do recommend the use of an open-source bundling and minification package, (WebOptimizer)[https://github.com/ligershark/WebOptimizer].

To start using this package, we first need to install it. Running the following dotnet command will install the NuGet package into your project.
```
dotnet add package LigerShark.WebOptimizer.Core 
```

Next, we need to add the WebOptimizer services into our services container:
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

Followed by adding WebOptimizer as middleware:
```
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...
    app.UseWebOptimizer();
    app.UseStaticFiles();
    ...
}
```
Note, order matters and `UseWebOptimizer` must come before `UseStaticFiles`. This is because the static files must be minified and bundled before they are to be rendered.

Lastly, we need to register the `@addTagHelper *, WebOptimizer.Core` Tag Helpers into _ViewImports.cshtml:
```c#
@addTagHelper *, WebOptimizer.Core
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

These tag helpers will target the `<script>` and `<link>` tags and enable cache busting, allowing features such as hot-reloading when developing, and removing outdated assets for new assets when assets are modified. 

## Testing Only the Minified Static Assets

## Testing Both Minified and Bundled Static Assets