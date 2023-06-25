---
title: "Correctly Reference Static Files Within a .Net Core Project"
date: 2023-06-25T00:00:00-07:00
draft: true
---

Have you ever noticed that the default razor page or razor view, `Shared/_Layout.cshtml`, for freshly created .NET web projects always contains references to the following files:
```html
...
<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
...
<script src="~/lib/jquery/dist/jquery.min.js"></script>
<script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
...
```

These are references to JQuery and Bootstrap are located in wwwroot:
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

This approach works well for development, however, as soon as the application is ready for production, our approach to retrieving these static assets should change.

JQuery and Bootstrap are not the most lightweight downloads. According to Bundlephobia, bootstrap@5.3.0 takes 6x the amount to download when compared to react@18.2.0, and jquery@3.7.0 can take up to 10x to download!

This is not to scare you away from JQuery or Bootstrap, as we do have options to improving the download speed of these packages.

## Using a CDN

One great way to improve the download speed of JQuery and Bootstrap is to use a CDN. When using a CDN, these static assets will be downloaded from the closest edge server to the user.