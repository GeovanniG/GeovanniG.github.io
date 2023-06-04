---
title: "Keeping .NET Core Projects Consistent By Enforcing Code Styling Rules"
date: 2023-06-04T00:00:00-07:00
draft: false
---

As projects grow and gain new developers, code tends to lose its consistency. Developers may have different preferences and unique ways of expressing their thoughts in code. If these differing preferences slips into the codebase, future developers will have a harder time understanding the code and may even have a hard time knowing which styles to adhere to. This is why a tool to check for these inconsistency is absolutely necessary. 

When setting up a new project it is important to layout the coding standards for the code. The policies do not need to be exhaustive and can be improved on as the code grows. However, they should lay the foundation to keep the code consistent and readable. Later, when it comes to refactoring/scanning the code, or bringing a developer up-to-speed, future developers will thank you for how well maintained the code is.

In this article, we will describe how to setup code-styling (IDExxxx) rules for .NET core projects.

## Setting up an .EditorConfig file

How do we go about keeping our code consistent? One great way is with the use of an EditorConfig file.

An EditorConfig file is a language agnostic file where we can define the coding styles and guidelines for a project.

A sample can be found at [EditorConfig.org](https://editorconfig.org/).
```
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
end_of_line = lf
insert_final_newline = true

# Matches multiple files with brace expansion notation
# Set default charset
[*.{js,py}]
charset = utf-8

# 4 space indentation
[*.py]
indent_style = space
indent_size = 4

# Tab indentation (no size specified)
[Makefile]
indent_style = tab

# Indentation override for all JS under lib directory
[lib/**.js]
indent_style = space
indent_size = 2

# Matches the exact files either package.json or .travis.yml
[{package.json,.travis.yml}]
indent_style = space
indent_size = 2
```

As we can see, the rules defined are very readable, and as you may have guessed, we are applying indentation styles to files with certain file extensions.

Taking this a step further, Microsoft has constructed their own properties specifically for C# and .NET. Their [styling guideline](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/naming-rules) describes and gives examples on how to construct .NET naming rules. 
> A full list of these custom naming rules for C# and .NET can be found in the [code-style rules documentation](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/).


## Enforcing Code Styling Rules

Now, after creating an EditorConfig file and defining some rules, our next task is to enforce them throughout the project. 

In .NET 5+ this can be accomplished by setting the MSBuild property `EnforceCodeStyleInBuild` to `true`:
``` XML
<PropertyGroup>
  <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
</PropertyGroup>
```

With this property enabled, all code-style analysis (IDExxxx) rules with a severity of warning or error will cause the build to fail. 


For example, the following rule states that access modifiers are required.
```
[*.{cs,vb}]

# IDE0040: Accessibility modifiers required (escalated to a build warning)
dotnet_diagnostic.IDE0040.severity = warning
```
If we fail to include an access modifier, the build will fail.

An alternative approach is to enable all code-style categories as warnings or errors, by default. Then, we can selectively silent rules that are not of interest.
```
[*.{cs,vb}]

# Default severity for analyzer diagnostics with category 'Style' (escalated to build warnings)
dotnet_analyzer_diagnostic.category-Style.severity = warning

# IDE0040: Accessibility modifiers required (disabled on build)
dotnet_diagnostic.IDE0040.severity = silent
```

Either of the 2 ways work and the choice is more of a preference. Nevertheless, choosing one is better then choosing none.

> As of this writing, there is an open [bug](https://github.com/dotnet/roslyn/issues/49439#issuecomment-821506457) where the C# and .NET naming rules, described in the preceding section, do not cause the build to fail. 
> 
> The 2 workarounds I have found are to: 
> 1. Set the severity of all styling rules to a severity of error: 
>```
>dotnet_analyzer_diagnostic.category-Style.severity = error
>``` 
>2. Include the diagnostic severity for each naming style as in:
>```
>csharp_prefer_simple_using_statement = true
>dotnet_diagnostic.IDE0063.severity = warning # diagnostic severity
>```

## Summary

In this article we discussed the benefits of keeping our code consistent, and introduced the EditorConfig file to solve this inconsistency in our code. We described its use and illustrated how to define code-style (IDExxxx) rules. We also mentioned extra C#/.NET styling rules that can be added to the EditorConfig. Lastly, we showed how to enforce code-style rules on build time with the MSBuild property `EnforceCodeStyleInBuild`. 

If your project is lacking consistency, introducing an EditorConfig may be the tool your project needs. For existing projects, these rules can be added gradually until all desired rules are introduced.

In future articles, we will discuss how to enforce code-quality (CAxxxx) rules. Until then, thanks for reading!