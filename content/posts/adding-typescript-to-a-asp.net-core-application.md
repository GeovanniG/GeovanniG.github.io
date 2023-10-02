---
title: "Adding Typescript To a ASP.NET Core Application"
date: 2023-10-01T00:00:00-07:00
draft: false
---

Typescript is a powerful language with many great features such as static typing, support for generics, interfaces, union types, and the list goes on and on. For C# developers, Typescript is a welcoming language. In this article, I will show how to add Typescript to your existing ASP.NET Core application. I will also describe some of the problems I have encountered, and how to overcome them.

## Adding Typescript to your Project

Adding Typescript to your project requires two components:

1. The first is the [Microsoft.TypeScript.MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/) NuGet package. This package will be responsible for converting TypeScript into JavaScript at build time.

2. Next, we need to configure the TypeScript compiler options. This is performed using a `tsconfig.json` file. This file must be created at the root of your asp.net project and must adhere to the https://json.schemastore.org/tsconfig.json JSON schema.

For simplicity, our `tsconfig.json` will look like the following:

```json
{
  "compileOnSave": true,
  "compilerOptions": {
    "noImplicitAny": false,
    "noEmitOnError": true,
    "removeComments": false,
    "sourceMap": true,
    "target": "es5",
    "outDir": "wwwroot/js"
  },
  "include": [
    "wwwroot/scripts"
  ]
}
```

With these configurations, the compiler will look for TypeScript files in `wwwroot/scripts` and output the compiled JavaScript files to `wwwroot/js`. The compiled files will adhere to the `es5` standard, so older browser versions will be supported. To import the compiled JavaScript files into our pages, we can use the `script` HTML as usual:

```html
<script src="~/js/app.js"></script>
```

> To learn more about the `tsconfig.json` file, please visit the [official documentation](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).

## Working With TypeScript

By following the previous steps, we are ready to use TypeScript in our project. To test that TypeScript is working, we can create a TypeScript file, `loading-modal.ts`, in `wwwroot/scripts/shared` with the following content:

```ts
document.addEventListener('DOMContentLoaded', () => {
    const bootstrap = window['bootstrap'];

    const modal = new bootstrap.Modal(document.getElementById('loadingDataModal'));
    modal.show();
})
```

Notice how we are also using bootstrap 5 to open a modal.

Next, we add the associated HTML. The last lines add the bootstrap 5 javascript file and the compiled javascript file, not the typescript file.

```html
...
<!-- Modal -->
<div class="modal fade" id="loadingDataModal" tabindex="-1" aria-labelledby="loadingDataLabel" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content text-center">
            <div class="modal-body">
                <div class="spinner-grow" role="status">
                    <span class="visually-hidden">Loading...</span>
                </div>
            </div>
        </div>
    </div>
</div>

...
 <environment include="Development">
     <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.js"></script>
 </environment>
 <environment exclude="Development">
     <script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.0/js/bootstrap.bundle.min.js"
             asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"
             asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
             crossorigin="anonymous"
             integrity="sha384-geWF76RCwLtnZ8qwWowPQNguL3RmwHVBC9FhGdlKrxdiJJigb/j/68SIy3Te4Bkz"></script>
 </environment>

<script src="@Url.Content("~/js/shared/loading-modal.js")"></script>
...
```

Running our application, we will see a modal displayed on the screen.

## Loading Third-party libraries using TypeScript (Optional)

In our previous example, we obtained bootstrap from the `window` object. This approach is great for simple applications, but it does not utilize TypeScript's full potential. For example, `bootstrap` has a type of `any`, which is impossible to infer the methods available to the object. It would be better if we can use TypeScript both for our code and for third-party libraries.

This can be accomplished by installing the [npm](https://www.nuget.org/packages/Npm/3.5.2?_src=template) NuGet package. Followed by creating a `package.json` file with the following content:

```json
{
  "version": "1.0.0",
  "name": "asp.net",
  "private": true,
  "dependencies": {

  },
  "devDependencies": {
    "@types/bootstrap": "^5.2.7"
  }
}
```

> For those interested, the full JSON schema can be found at https://json.schemastore.org/package.json.

Lastly, we need to install the dependencies using `npm install`.

With the addition of bootstrap type definitions, our TypeScript file can be simplified to the following:

```ts
document.addEventListener('DOMContentLoaded', () => {
    const modal = new bootstrap.Modal(document.getElementById('loadingDataModal'));
    modal.show();
})
```

Notice how we removed `const bootstrap = window['bootstrap'];`. By including the type definitions in this way, `bootstrap` is no longer of type `any`. Furthermore, we are able to have auto-complete features that we have grown accustom to.

Thanks for reading!