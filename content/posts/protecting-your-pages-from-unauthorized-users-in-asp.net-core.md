---
title: "Protecting Your Pages From Unauthorized Users In ASP.NET Core"
date: 2023-08-27T00:00:00-07:00
draft: false
---

ASP.NET Core has built-in support for protecting your pages. Add the following middleware to your startup code:

```c#
_ = app.UseAuthentication();

_ = app.UseAuthorization();
```

and authentication and authorization capabilities will be added to your project. To take advantage of this middleware, we can use the `AuthorizeAttribute`. When applying this attribute to a route, like so:

```c#
[Authorize]
public class IndexModel : PageModel
{
    public void OnGet()
    {
    }
}
```

if a user is not logged in and they attempt access the page, they will not be granted access. Instead, they will be redirected to the dreaded Access Denied Page:

```c#
@page
@model AccessDeniedModel
@{
    ViewData["Title"] = "Access denied";
}

<header>
    <h1 class="text-danger">@ViewData["Title"]</h1>
    <p class="text-danger">You do not have access to this resource.</p>
</header>
```

However, we can take this one step further. We can protect a page depending on the role or policy assigned to the user.

## Role-based / Claim-based Authorization

To protect a page via a claim or role, we can use the `AuthorizeAttribute` with the `Policy` or `Role` properties, respectively.

For example, we can use the `UserManager` to add a role or claim to a user:

```c#
public class IdentityService : IIdentityService
{
    private readonly UserManager<IdentityUser> _userManager;
    private readonly SignInManager<IdentityUser> _signInManager;

    public IdentityService(
        UserManager<IdentityUser> userManager,
        SignInManager<IdentityUser> signInManager)
    {
        _userManager = userManager;
    }

    public async Task<bool> IsInRoleAsync(string userId, string role)
    {
        var user = _userManager.Users.SingleOrDefault(u => u.Id == userId);

        return user != null && await _userManager.IsInRoleAsync(user, role);
    }

    public async Task<bool> AddToRole(string userId, string role)
    {
        var user = _userManager.Users.SingleOrDefault(u => u.Id == userId);

        if (user == null)
        {
            return false;
        }

        var result = await _userManager.AddToRoleAsync(user, role);

        return result.Succeeded;
    }

    public async Task<bool> IsInClaimAsync(string userId, Claim claim)
    {
        var user = _userManager.Users.SingleOrDefault(u => u.Id == userId);

        if (user == null)
        {
            return false;
        }

        var result = await _userManager.GetClaimsAsync(user);

        return result.Any(c => c.Type == claim.Type && c.Value == claim.Value);
    }

    public async Task<bool> AddToClaimAsync(string userId, Claim claim)
    {
        var user = _userManager.Users.SingleOrDefault(u => u.Id == userId);

        if (user == null)
        {
            return false;
        }

        var result = await _userManager.AddClaimAsync(user, claim);

        if (result.Succeeded)
        {
            await _signInManager.RefreshSignInAsync(user);
        }

        return result.Succeeded;
    }
}
```

Once the user has been assigned a role or claim, we update the `AuthorizeAttribute` to only accept users with these properties:

```c#
[Authorize(Roles = "Admin")]
public class IndexModel : PageModel
{
    public void OnGet()
    {
    }
}
```

The example given above uses an "Admin" role. Nevertheless, the role can be any name meaningful to your application.

Claim authorization follows a similar format with a few extra conditions. Instead of using the `Roles` property, we supplant it with the `Policy` property:

```c#
[Authorize(Policy = "EmployeeOnly")]
public class IndexModel : PageModel
{
    public void OnGet()
    {
    }
}
```

The application has no clue what this policy entails. We provide those details in the application's startup:

```c#
_ = _services.AddAuthorization(options =>
{
   options.AddPolicy("EmployeeOnly", policy => policy.RequireClaim("EmployeeNumber"));
});

```

Within `AddAuthorization`, we can decide what conditions are necessary to satisfy the policy.

## Custom Policy-based Authorization

Using claim authorization provides great flexibility. In some cases, this enough. When it is not enough, we have another authorization tool we can turn to: Policy-based Authorization.

With Policy-based Authorization, we can define our own requirements using `IAuthorizationRequirement`, and the conditions needed to satisfy our new requirements with `AuthorizationHandler<IAuthorizationRequirement>`. 

For example, we can have a `MinimumAgeRequirement` with a `MinimumAge` Property:

```c#
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public MinimumAgeRequirement(int minimumAge) =>
        MinimumAge = minimumAge;

    public int MinimumAge { get; }
}
```

After creating our requirement, we must specify the conditions to satisfy the requirement. In our case, the requirement will only be satisfied if the user's age is greater than the `MinimumAge`.

We can create this condition by inheriting from `AuthorizationHandler`:

```c#
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, MinimumAgeRequirement requirement)
    {
        var dateOfBirthClaim = context.User.FindFirst(
            c => c.Type == ClaimTypes.DateOfBirth && c.Issuer == "http://helloworld.com");

        if (dateOfBirthClaim is null)
        {
            return Task.CompletedTask;
        }

        var dateOfBirth = Convert.ToDateTime(dateOfBirthClaim.Value);
        int calculatedAge = DateTime.Today.Year - dateOfBirth.Year;
        if (dateOfBirth > DateTime.Today.AddYears(-calculatedAge))
        {
            calculatedAge--;
        }

        if (calculatedAge >= requirement.MinimumAge)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

Inheriting from `AuthorizationHandler` allows us to specify the conditions to satisfy our requirements.

With our handler and requirements created, we are ready to register our handler and add a new policy using our newly created requirement, `MinimumAgeRequirement`:

```c#
 _ = services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();

 _ = services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast18", policy =>
         policy.Requirements.Add(new MinimumAgeRequirement(18)));
});
```

In our case, our policy is named "AtLeast18", and the condition needed to satisfy this requirement is the user must have an age that is at least 18.

We can use this policy by assigning the policy's name to the `Policy` property in the `AuthorizeAttribute`:

```c#
[Authorize(Policy = "AtLeast18")]
public class IndexModel : PageModel
{
    public void OnGet()
    {
    }
}
```

## Summary

There we have it. Three different ways to authorize a user. There are other techniques such as creating [Custom Authorization attributes](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider?view=aspnetcore-7.0#custom-authorization-attributes), or [Resource-based authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased?view=aspnetcore-7.0), which is an extension of Policy-based authorization. However, in most cases, the techniques provided here should handle most cases.

Thanks for reading!

