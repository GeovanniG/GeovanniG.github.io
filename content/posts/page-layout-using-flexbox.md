---
title: "Page Layout Using Flexbox"
date: 2023-09-24T00:00:00-07:00
draft: true
---

When first starting a web application, you may notice that your navbar, content, and footer are all smooched together. Traditionally, even if we do not have enough content to fill the page, we would still like our footer to be located at the bottom of the webpage.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>ResumeVault A.I. - @ViewData["Title"]</title>
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body class="vh-100">
    <header>
        <nav id="layout-header" class="navbar navbar-expand-sm navbar-toggleable-sm">
            <div class="container-xl">
                <a class="nav-link pe-3" asp-area="" asp-page="/Index">Web App</a>
                <button class="navbar-toggler collapsed border-0" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle px-2" id="navbarDropdownMenuLink" asp-page="/Index" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                                    Home
                                </a>
                                <ul class="dropdown-menu" aria-labelledby="navbarDropdownMenuLink">
                                    <li class="nav-item"><a class="nav-link" asp-area="" asp-page="/Index">Main Page</a></li>
                                </ul>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link px-2" asp-area="" asp-page="/Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    
    <section id="content-wrap">
        <main role="main" class="container-xl">
            @RenderBody()
        </main>
    </section>

    <footer class="footer">
        <div class="container-xl">
            &copy; 2023 - Web App - <a class="" asp-area="" asp-page="/Privacy">Privacy</a>
        </div>
    </footer>
</body>
</html>
```

```css
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>ResumeVault A.I. - @ViewData["Title"]</title>
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body class="d-flex flex-column vh-100">
    <header>
        <nav id="layout-header" class="navbar navbar-expand-sm navbar-toggleable-sm">
            <div class="container-xl">
                <a class="nav-link pe-3" asp-area="" asp-page="/Index">Web App</a>
                <button class="navbar-toggler collapsed border-0" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle px-2" id="navbarDropdownMenuLink" asp-page="/Index" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                                    Home
                                </a>
                                <ul class="dropdown-menu" aria-labelledby="navbarDropdownMenuLink">
                                    <li class="nav-item"><a class="nav-link" asp-area="" asp-page="/Index">Main Page</a></li>
                                </ul>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link px-2" asp-area="" asp-page="/Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    
    <section id="content-wrap" class="flex-fill">
        <main role="main" class="container-xl h-100">
            @RenderBody()
        </main>
    </section>

    <footer class="footer">
        <div class="container-xl">
            &copy; 2023 - Web App - <a class="" asp-area="" asp-page="/Privacy">Privacy</a>
        </div>
    </footer>
</body>
</html>
```