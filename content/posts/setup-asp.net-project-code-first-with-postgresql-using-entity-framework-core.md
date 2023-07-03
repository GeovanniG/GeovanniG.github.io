---
title: "Setup ASP.NET Project Code-First With PostgreSQL Using Entity Framework Core"
date: 2023-07-02T00:00:00-07:00
draft: false
---

When developing an ASP.NET CRUD application, we have two main ways of communicating with our database server. We can take a database-first approach or a code-first approach. 

With a database-first approach, we take full advantage of the database features and optimize our queries by communicating with the databases using stored procedures. Usually, our database has existed long before our application has and it contains logic for querying the data in the form of stored procedures. With this direct form of communication, our reads and writes are much faster, however, with our logic tied to databases, switching to another database in the future requires considerable refactoring.

With a code-first approach, we really more heavily on an ORM. Whether the ORM is Entity Framework Core or Dapper, our main form of communication with the database is in the language of the ORM; which in C# is normally LINQ. A great benefit of using an ORM is we are not locked into a particular database. The database is abstracted away and developers can focus more on the application logic. Of course, the developer must be proficient in LINQ, as performance can become a bottleneck if inefficient LINQ is used. As hinted, performance is not as great as taking a database-first approach, as we are no longer using native features of the database; however, we do reduce complexity of the application, which has it's own benefits.

Now that we have some context on the difference between the two approaches, I would like to discuss how to take advantage of a code-first approach by developing an ASP.NET application using PostgreSQL as our database provider. A database-first approach will be discussed in a future article, as it requires an existing database.

## Setting up PostgreSQL with EntityFramework Core

To begin our discussion, we will showcase how to configure our application with Postgres with the help of Entity Framework Core. Entity Framework is an ORM developed by MicroSoft and makes it easy to integrate with 3rd-party database providers.

To get started, we need three main packages:

```bash
dotnet add package Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

To give some context on the functionality of each package:

* The first package is for adding health checks to our database via `AddHealthChecks`. On application start up, this package will check the health of our database; if there are any issues, it will inform us immediately. It also provides a `CheckHealthAsync` method, in `DefaultHealthCheckService`, that we can periodically call to assess any peculiarities in our database.

* The second package reveals detailed database errors via `AddDatabaseDeveloperPageExceptionFilter`. As the method signature suggests, we should only enable this feature in development; otherwise, we run the risk of revealing sensitive application data to our users, such as our database schema.

* The last package is responsible for adding Postgres to our application. With this package, we have all the packages necessary to add Postgres as our database provider. Technically, the first two packages are optional, however, they do make debugging our application that much simpler.

Putting this all together, we end up with the following registered services:

```c#
public static class ConfigureServices
{
    public static IServiceCollection AddServices(this IServiceCollection services, IConfiguration configuration)
    {
        _ = configuration.GetConnectionString("DbConnection")
            ?? throw new InvalidOperationException("Connection string 'DbConnection' not found.");

        _ = services.AddHealthChecks()
            .AddDbContextCheck<AppDbContext>();

        var serviceProvider = services.BuildServiceProvider();
        var env = serviceProvider.GetRequiredService<IWebHostEnvironment>();

        if (env.IsDevelopment())
        {
            _ = services.AddDatabaseDeveloperPageExceptionFilter();
        }

        _ = services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(configuration.GetConnectionString("DbConnection"),
                builder => builder.MigrationsAssembly(typeof(AppDbContext).Assembly.FullName)));

        services.AddScoped<IAppDbContext>(provider => provider.GetRequiredService<AppDbContext>());

        return services;
    }
}
```

From the above code, we can see that we are looking for a connection string called `DbConnection`. We need to add this to our `appsettings.json` in the `ConnectionStrings` section:

```json
{
    ...
    "ConnectionStrings": {
        "DbConnection": "Server=127.0.0.1;Database=<database>;Port=5432;User Id=<username>;Password=<password>"
    }
    ...
}
```

With these services and configurations configured, we move on to the database context.

## Configuring the Database Context

Setting up the database context is straightforward. In fact, the same code given below can used for all database providers, whether it be Postgres, SQL Server, or any other supported database. If you have ever configured a database context before, the following will look familiar:

```c#
public class AppDbContext : DbContext, IAppDbContext
{
    public AuthDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }
}

public interface IAppDbContext 
{
}
```

With the database context setup, our application is ready to facilitate communication with our database. To test the connection, we will need to add an entity and create a migration. Migrations are a fascinating topic and will be discussed more thoroughly in a future post.

## Extras: Adding ASP.NET Core Identity

If your application involves user registration, we will need add ASP.NET Core Identity to our project. To do so, we will need the following packages:

```
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
```

These packages make the default identity provider available to our database context. 

To configure our application to take advantage of the identity provider, we need to update `AddServices` by adding `AddDefaultIdentity` and `AddEntityFrameworkStores`. `AddRoles` is optional, it only becomes necessary when our application incorporates roles for authorization.

The final product is the following:

```c#
public static class ConfigureServices
{
    public static IServiceCollection AddServices(this IServiceCollection services, IConfiguration configuration)
    {
        ...

        if (env.IsDevelopment())
        {
            _ = services.AddDatabaseDeveloperPageExceptionFilter();
        }

        _ = services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
            .AddRoles<IdentityRole>()
            .AddEntityFrameworkStores<AppDbContext>();

        _ = services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(configuration.GetConnectionString("DbConnection"),
                builder => builder.MigrationsAssembly(typeof(AppDbContext).Assembly.FullName)));

        ...
    }
}
```

Lastly, we will need to update `AppDbContext` to inherit from `IdentityDbContext`, instead of `DbContext`. This will add the identity tables and schema to our database context.

```c#
public class AppDbContext : IdentityDbContext, IAppDbContext
{
    public AuthDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }
}
```

After creating a migration and updating our database, we will see the newly created identity tables.

## Recommendations: Use Postgres Naming Convention, snake_case

Postgres has a naming convention similar to Python. The community prefers to use `snake_case` for the names of databases, tables, columns, etc. In Entity Framework, there are numerous ways to adhere to this pattern, from using `DataAnnotations`, to using Fluent API. 

To showcase the naming convention for the identity tables created above, we will use Fluent API in `OnModelCreating`. To do so, we use the `ToTable` method to give our tables a proper name:

```c#
public class AppDbContext : IdentityDbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        _ = builder.Entity<IdentityUser>().ToTable("users");
        _ = builder.Entity<IdentityRole>().ToTable("roles");
        _ = builder.Entity<IdentityUserRole<string>>().ToTable("user_roles");
        _ = builder.Entity<IdentityUserClaim<string>>().ToTable("user_claims");
        _ = builder.Entity<IdentityUserLogin<string>>().ToTable("user_logins");
        _ = builder.Entity<IdentityUserToken<string>>().ToTable("user_tokens");
        _ = builder.Entity<IdentityRoleClaim<string>>().ToTable("role_claims");
    }
}
```

Now if we ever query our database using a RDMS like PgAdmin, we will be able to utilize features such as autocomplete.

## Summary

In this article we discussed two different ways we communicating with our database: database-first and code-first. We then went on to configure our ASP.NET application, using Entity Framework as our ORM and Postgres as our database provider. In the examples given above, we decided to take a code-first approach to showcase how simple it is to setup a project with an ORM.

Postgres is a great database and I hope this article showcased how simple it is to incorporate into an ASP.NET application.

Thanks for reading!