---
title: "Setting Up Reusable .NET Project Templates For Future Projects"
date: 2023-06-11T00:00:00-07:00
draft: false
---

When creating new .NET projects, do you constantly find yourself structuring them in the same way? Maybe you have found a certain project structure that suites you or your organization well. If you have and you would like to reuse this project structure for future projects, what you need is a project template.

In this article we will describe how to create and use a project template to quickly bootstrap a new project. With a project template, a future project can leverage this template, reducing much of the boilerplate code needed to setup a new project.

Furthermore, a project template can also help maintain consistency between projects. This will make transitioning between multiple projects much smoother, hopefully, allowing developers to ramp up more quickly.

## Creating a Project Template

To create a project template, .NET requires a special folder, `.template.config`, and config file, `template.json`, to exist at the root of your project. 

Let's start by navigating to root of your project and creating a folder called `.template.config` and a config file within the folder called `template.json`. The structure should look like the following:

```
TemplateSolution
└───.template.config
        template.json
```

The config file will define what our project template will display when we execute `dotnet new list`. Keep in mind that this JSON config file should oblige to the JSON template schema: http://json.schemastore.org/template.

An example with all of the required fields is given below:

```json
{
  "$schema": "http://json.schemastore.org/template",
  "author": "Me",
  "classifications": [ "Common", "Console" ],
  "identity": "ExampleTemplate.Project",
  "name": "Example templates: console project",
  "shortName": "exampleconsoletemplate",
  "tags": {
    "language": "C#",
    "type": "project"
  }
}
```

In the example given, once we install our project template and execute `dotnet new list`, it will showcase our template like the following:

```bash
Templates                     Short Name               Language         Tags
------------------------      -------------------      ------------     ----------------------
Example templates: co...      exampleconsolete...      [C#]             Common/Console
```
Therefore, it is important to be descriptive, yet concise, when describing our project template.

Other properties that are great to include are `sourceName` and `sources`.

```json
{
    ...
    "sourceName": "NameToUseInTemplate",
    "sources": [
        {
        "source": "./",
        "target": "./",
        "exclude": [
            "README.md",
            "**/[Bb]in/**",
            "**/[Oo]bj/**",
            ".template.config/**/*",
            ".vs/**/*",
            "**/*.filelist",
            "**/*.user",
            "**/*.lock.json",
            "**/.git/**",
            "*.nuspec",
            "**/node_modules/**"
        ]
        }
    ]
    ...
}
```

These properties accomplish the following:
* `sourceName` will replace all instances of *NameToUseInTemplate* in the project with a value specified by the user on creation time.
* `sources` will ignore certain folders and files. By default, the template will include all files in the template directory, including the *obj* and *bin* folders. With this property, we can ignore such folders.

Now that we have our project setup, it's time to install it.

## Installing a Project Template

Installing your template will make it available as a .NET template option when we execute `dotnet new list`.

To install your project template, we need to run the `dotnet new install` command while in the root directory of the project:

```bash
dotnet new install .\ (windows)
dotnet new install ./ (linux)
```

If successful, this command will generate the following output

```bash
The following template packages will be installed:
   <root path>\TemplateSolution

Success: <root path>\TemplateSolution installed the following templates:
Templates                     Short Name               Language         Tags
------------------------      -------------------      ------------     ----------------------
Example templates: co...      exampleconsolete...      [C#]             Common/Console
```

As we can see, the properties that we specified in the `template.json` are showing here.

Furthermore, when we execute `dotnet new list`, we will see the template available as one of the template options.

> To make a template available for the public or for your entire organization, you will need to create a NuGet package. We will discuss how to create a NuGet packages in a future post.

## Using Our Template

Now that we have our template installed, our next step is to use it. To begin using it, execute `dotnet new` with the shortname specified in the `template.json`. 

For example:

```bash
dotnet new exampleconsoletemplate -o TestTemplate
```

This will create a new project with a name of `TestTemplate`. This new project will be based off of our project template, `exampleconsoletemplate`, created above.


## Other Options

Occasionally, you will need to make updates to the project template. To do this, first make changes to the project and then force re-install the project template with the `--force` option:

```bash
dotnet new install .\ --force (windows)
dotnet new install ./ --force (linux)
```

This will replace our old template with the newly updated version.

Lastly, if you would like to remove the template, we can uninstall it using `dotnet new uninstall` command while in the root directory of the project:

```bash
dotnet new uninstall .\ (windows)
dotnet new uninstall ./ (linux)
```

This will get rid of the template and it will no longer display when we execute `dotnet new list`.

## Summary

In this article we described the benefits of creating a project template. We also illustrated when and how to create a project template. Lastly, we created a project based off of our newly created project template.

If you constantly find yourself creating projects with the same structure, you may want to consider creating a project template before starting your next project. This will save you considerable time in the long run as you will no longer have create the same code repeatedly.

Thanks for reading!