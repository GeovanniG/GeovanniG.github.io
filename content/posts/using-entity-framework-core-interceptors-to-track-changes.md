---
title: "Using Entity Framework Core Interceptors to Track Changes"
date: 2023-07-16T00:00:00-07:00
draft: false
---

It is a good practice to track row modifications and creations within a table. Tracking these modifications can serve as audits, and can help us determine exactly when the data was last modified. To accomplish this, we can create columns within a table to serve as audit columns.

Our next task will then be to fill these audit columns as we create and modify rows within our tables. With Entity Framework Core, we can accomplish this using interceptors.

## What is an Entity Framework Core Interceptor?

An interceptor acts on operations, allowing us to further modify or suppress an action within Entity Framework Core. For example, with an interceptor, we can modify our data before or after it has been saved to the database. As you may have guessed, we can use such an interceptor to fill our audit columns.

Before continuing, we would like to note that interceptors can be added in 2 different locations: `AddDbContext` or `OnConfiguring`. The choice of were to add it is based on personal preference. Later in this article, we will show how to add an interceptor using both methods.

## Creating an Auditable Base Entity

To begin tracking our row changes, we must first add extra properties to our entities. To accomplish this, we can create a class `BaseAuditableEntity` with `CreatedOn`, `CreatedBy`, `ModifiedOn`, and `ModifiedBy` properties:

```c#
public abstract class BaseAuditableEntity : BaseEntity
{
    public DateTime CreatedOn { get; set; } = DateTime.UtcNow;
    public string CreatedBy { get; set; } = null!;
    public DateTime? ModifiedOn { get; set; }
    public string? ModifiedBy { get; set; }
}

public class BaseEntity
{
    public string Id { get; set; } = Guid.NewGuid().ToString();
}
```

To start tracking our modifications and creations, we must remember to inherit from `BaseAuditableEntity`, when creating new entities. For example, if we had a `Car` entity, we will need to inherit from `BaseAuditableEntity` to take advantage of our interceptor. We will see why in the next section:

```c#
public class Car : BaseAuditableEntity
{
    public int Wheels { get; set; }
}
```

With our properties in place, lets create our first interceptor.

## Creating an Interceptor to Track Changes

Before creating an interceptor, we must first decide which operations we would like our interceptor to modify. All interceptor implement the [IInterceptor](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.diagnostics.iinterceptor?view=efcore-7.0) interface. Therefore, all interceptor that can be overridden must implement this interface. To locate all available interceptors, we can visit the [IInterceptor documentation](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.diagnostics.iinterceptor?view=efcore-7.0). 

Using the link, we see the following options:

* DbCommandInterceptor
* DbConnectionInterceptor
* DbTransactionInterceptor
* IDbCommandInterceptor
* IDbConnectionInterceptor
* IDbTransactionInterceptor
* IgnoringIdentityResolutionInterceptor
* IIdentityResolutionInterceptor
* IInstantiationBindingInterceptor
* IMaterializationInterceptor
* IQueryExpressionInterceptor
* ISaveChangesInterceptor
* ISingletonInterceptor
* SaveChangesInterceptor
* UpdatingIdentityResolutionInterceptor

As we can see from the list above, Entity Framework Core reveals an interface and a class for each type of operation. Depending on the operation you are planning to modify, you will need to implement or inherit the appropriate interceptor. In general, if you do not need to implement every method within the interface, it is best to inherit the class. 

For instance,`ISaveChangesInterceptor` and `SaveChangesInterceptor` are interceptors used to further modify data before or after it is saved to the database. If we do not need to implement every method within the `ISaveChangesInterceptor`, it is best to use `SaveChangesInterceptor`.

We can showcase an example utilizing `BaseAuditableEntity`, created above. This example will update `CreatedBy`, `CreatedOn`, `ModifiedBy` and `ModifiedOn`, when we add a new row, and update `ModifiedBy` and `ModifiedOn`, when we edit an existing row. Here, we have decided to inherit the operation class, as we did not need to implement every method within `ISaveChangesInterceptor`:

```c#
public class AuditableEntitySaveChangesInterceptor : SaveChangesInterceptor
{
    private readonly ICurrentUserService _user;

    public AuditableEntitySaveChangesInterceptor(
        ICurrentUserService user)
    {
        _user = user;
    }

    public override InterceptionResult<int> SavingChanges(DbContextEventData eventData, InterceptionResult<int> result)
    {
        UpdateEntities(eventData.Context);

        return base.SavingChanges(eventData, result);
    }

    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(DbContextEventData eventData, InterceptionResult<int> result, CancellationToken cancellationToken = default)
    {
        UpdateEntities(eventData.Context);

        return base.SavingChangesAsync(eventData, result, cancellationToken);
    }

    public void UpdateEntities(DbContext? context)
    {
        if (context == null)
        {
            return;
        }

        foreach (var entry in context.ChangeTracker.Entries<BaseAuditableEntity>())
        {
            if (entry.State == EntityState.Added)
            {
                entry.Entity.CreatedBy = _user.Id ?? string.Empty;
                entry.Entity.CreatedOn = DateTime.UtcNow;
            }

            if (entry.State == EntityState.Added || entry.State == EntityState.Modified || entry.HasChangedOwnedEntities())
            {
                entry.Entity.ModifiedBy = _user.Id;
                entry.Entity.ModifiedOn = DateTime.UtcNow;
            }
        }
    }
}

public static class EntityEntryExtensions
{
    public static bool HasChangedOwnedEntities(this EntityEntry entry) =>
        entry.References.Any(r =>
            r.TargetEntry != null &&
            r.TargetEntry.Metadata.IsOwned() &&
            (r.TargetEntry.State == EntityState.Added || r.TargetEntry.State == EntityState.Modified));
}

public class CurrentUserService : ICurrentUserService
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public CurrentUserService(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public string? Id => _httpContextAccessor.HttpContext?.User?.FindFirstValue(ClaimTypes.NameIdentifier);
}
```

Notice how `UpdateEntities` relies on `BaseAuditableEntity`; this is why it is important to have our entities inherit from this class. With this interceptor defined, even if we forget to update the audit fields manually, this method will perform this task on our behalf!

However, before this interceptor can take effect, we must first register it with our database context.

## Registering Interceptor

As hinted above, we have 2 locations where we can add an interceptor: `AddDbContext` or `OnConfiguring`.

Let's showcase both:

### Registering Interceptor in `OnConfiguring`

This approach is the preferred approach according to the [Entity Framework Core documentation on interceptors](https://learn.microsoft.com/en-us/ef/core/logging-events-diagnostics/interceptors#registering-interceptors).

In this approach, we add our interceptor by calling `AddInterceptors` within `OnConfiguring`:

```c#
public class ApplicationDbContext : DbContext
{
    private readonly AuditableEntitySaveChangesInterceptor _auditableEntitySaveChangesInterceptor;

    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options,
        AuditableEntitySaveChangesInterceptor auditableEntitySaveChangesInterceptor)
        : base(options)
    {
        _auditableEntitySaveChangesInterceptor = auditableEntitySaveChangesInterceptor;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.AddInterceptors(_auditableEntitySaveChangesInterceptor);
}

public static class ConfigureServices
{
    public static IServiceCollection AddServices(this IServiceCollection services, IConfiguration configuration)
    {
        _ = services.AddScoped<AuditableEntitySaveChangesInterceptor>();
        return services;
    }
}

```

### Registering Interceptor in `AddDbContext`

An alternative approach is to call `AddInterceptors` within `AddDbContext`. We perform this action when registering our database context:

```c#
public static class ConfigureServices
{
    public static IServiceCollection AddServices(this IServiceCollection services, IConfiguration configuration)
    {
        _ = services.AddScoped<AuditableEntitySaveChangesInterceptor>();

        var serviceProvider = services.BuildServiceProvider();

        _ = services.AddDbContext<ApplicationDbContext>(options =>
            {
                _ = options
                .AddInterceptors(
                    (AuditableEntitySaveChangesInterceptor)serviceProvider
                .GetRequiredService(typeof(AuditableEntitySaveChangesInterceptor)));
            });


        return services;
    }
}

public class ApplicationDbContext : DbContext
{
    private readonly AuditableEntitySaveChangesInterceptor _auditableEntitySaveChangesInterceptor;

    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
}
```

As you can see, this approach is not as elegant as using `OnConfiguring`. With this approach, we needed to rely on the ServiceProvider to register `AuditableEntitySaveChangesInterceptor` and cast our interceptor.  In most cases, it is preferred to register interceptors in `OnConfiguring`.

## Summary

In this article we described how interceptors can be used to perform modifications either before or after an operation takes place. 

By utilizing the `SaveChangesInterceptor`, we showed how we can update audit columns before they are saved to the database. We also illustrated 2 ways of registering an interceptor, with the preferred approach being within `OnConfiguring`.

With the example given, I hope it gave you some ideas on how to take advantage of interceptors within your database context.

As always, thanks for reading!