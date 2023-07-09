---
title: "Automate Entity Naming Conventions Using Entity Framework Core"
date: 2023-07-09T00:00:00-07:00
draft: false
---

In a [previous post](/posts/setup-asp.net-project-code-first-with-postgresql-using-entity-framework-core), we discussed how to configure an ASP.NET application to use PostgreSQL as the database provider. In that post, we briefly touched on the topic of database naming conventions. In this post, I would like to explore this topic further.

Let's begin our discussion by going over a few database naming conventions:

## SQL Server Naming Convention

Similar to much of C#'s naming convention, SQL Server also has preference of using Pascal case for tables, columns, stored procedures and most constructs that can be created in SQL Server.

If your C# code adheres to the recommended naming convention of using Pascal case for classes and properties, then Entity Framework Core will generously abide to SQL Server's naming conventions when constructing migrations. This is great if SQL Server is your database of choice, but what about for other databases, such as Postgres.

## Postgres Naming Convention

Unlike SQL Server, Postgres has a slightly different naming convention. In Postgres, the preferred naming convention is to use snake case. This convention is normally applied to tables, columns, etc.

As described above, Entity Framework Core will create our migrations following a Pascal case naming convention. How can we address this naming convention issue for databases other than SQL Server?

## Changing Entity Framework's Default Naming Convention

Entity Framework is a very flexible framework. It allows us change many of its underlying settings, this includes the naming convention of the migrations.

We can do this alteration in one of three ways. Let's begin by discussing the manual approaches.

### Using Data Annotations

When creating our entities, we have the option of specifying a table name and column names using data annotations: `Table` and `Column`. 

The Table attribute takes 2 parameters, the first is required, while the second is optional: `[Table(string name, Properties:[Schema = string])]`. With this attribute, we can enter whatever name we would like to use for our entity. 

A complementary attribute is the Column attribute: `[Column (string name, Properties:[Order = int],[TypeName = string])]`. The column attribute takes three parameters, the first is required, while the other two are optional. Here, again, we can enter whatever name we would like to use for the column name.
 
To show these attributes in action, let's consider a `Car` entity with a column `Model`. Using these data annotations, we will rename this table to `car_master`, and rename the column to `model_type`:

```c#
using System.ComponentModel.DataAnnotations.Schema;

[Table("car_master")]
public class Car
{
    public long Id { get; set; }

    [Column("model_type")]
    public string Model { get; set; }
}
```

We can also use Fluent Api to accomplish the same task.

### Using Fluent Api

There are a few different ways to configure Fluent Api. To simplify our example, we will be applying our entity configurations in `OnModelCreating`.

To continue our example above, instead of changing the table and column names using data annotations, we will use the Fluent Api methods: `ToTable` and `HasColumnName`.

These methods work very similar to the data annotation version. Repeating the same scenario above, instead using Fluent Api, we end up with the following:

```c#
public class Car
{
    public long Id { get; set; }
    public string Model { get; set; }
}

public class CarContext: DbContext 
{
    public CarDBContext(
        DbContextOptions<AuthDbContext> options)
        : base(options)
    {
    }

    public DbSet<Car> Cars { get; set; }
    
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Car>().ToTable("car_master");
        modelBuilder.Entity<Car>()
            .Property(x => x.Model).HasColumnName("model_type");
    }
}
```

Both of these options are very readable. If we only have to perform this for a few tables and columns, then this is really not an issue. 

However, as soon as our entities grow, doing this for every table and column can become tedious. This approach is also error prone. If, for example, we accidentally forget to rename a column, our schema becomes inconsistency. This inconsistency could have been avoided from the start, if we automated the process.

### Automating Naming Conventions Using Entity's Metadata

Luckily, we do have a way of automating the table and column naming conventions. We can do this by relying on `OnModelCreating`. As the name suggests, this method is executed during the model creation process. In this method, we can access the entities, using `GetEntityTypes`, and rename the column and tables accordingly.

For instance, if we reuse the same database context above, we can use `SetTableName` and `SetColumnName`, in conjunction with a custom `ToSnakeCase` string extension method, to rename our tables and columns:

```c#
public class CarContext: DbContext 
{
    public CarDBContext(
        DbContextOptions<AuthDbContext> options)
        : base(options)
    {
    }

    public DbSet<Car> Cars { get; set; }
    
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        foreach (var entity in modelBuilder.Model.GetEntityTypes())
        {
            // Replace table names
            if (entity.GetTableName() is not null)
            {
                entity.SetTableName(entity.GetTableName()!.ToSnakeCase());
            }

            // Replace column names            
            foreach (var property in entity.GetProperties())
            {
                if (property.GetColumnName() is not null)
                {
                    property.SetColumnName(property.GetColumnName().ToSnakeCase());
                }
            }

            // Replace key names
            foreach (var key in entity.GetKeys())
            {
                if (key.GetName() is not null)
                {
                    key.SetName(key.GetName()!.ToSnakeCase());
                }
            }

            // Replace foreign key names
            foreach (var key in entity.GetForeignKeys())
            {
                if (key.GetConstraintName() is not null)
                {
                    key.SetConstraintName(key.GetConstraintName()!.ToSnakeCase());
                }
            }

            // Replace indices names
            foreach (var index in entity.GetIndexes())
            {
                if (index.GetFilter() is not null)
                {
                    index.SetFilter(index.GetFilter()!.ToSnakeCase());
                }
            }
        }
    }
}

public static partial class StringExtensions
{
    public static string ToSnakeCase(this string input)
    {
        if (string.IsNullOrEmpty(input)) { return input; }

        var startUnderscores = Regex.Match(input, @"^_+");
        return startUnderscores + Regex.Replace(input, @"([a-z0-9])([A-Z])", "$1_$2").ToLowerInvariant();
    }
}
```

This method even renames keys and indices, something that we cannot do with data annotations! With this construct, we no longer need to manually rename our tables and columns. When we create our migrations, the tables and columns will automatically be converted into snake case for us! 

When using Postgres, this is my preferred way of configuring my entities to ensure my database has a consistent naming convention.

## Summary

In this post we discussed three different options to renaming database entities. We used data annotations and Fluent Api, then quickly realized that our solution did not scale as our entity count grow in size. 

We finally introduced a means of automating naming conventions. In particular, we illustrated how to convert all entities into snake case.

If your database has a different naming convention other than Pascal, consider using `OnModelCreating` with `GetEntityTypes` to automate the naming convention. This will make working with your data, outside of an ORM, that much more enjoyable.

Thanks for reading!