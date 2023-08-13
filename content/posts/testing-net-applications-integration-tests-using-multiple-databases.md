---
title: "Testing .NET Applications: Integration Tests Using Multiple Databases"
date: 2023-08-13T00:00:00-07:00
draft: false
---

To continue our previous discussion on testing .NET applications, in this article, we will delve into integration testing. In particular, how to perform integration tests for applications that utilize one or more databases.

## Database Structure

Let's assume we have an application that communicates with two databases. One database will store application logic, while the second database will store user information.

For example, we can have `ApplicationDbContext` for the application. This database will be primarily used for application logic and inherits from `DbContext`:

```c# {linenos=true,linenostart=1}
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    { }
}
```

Our second database, `AuthenticationDbContext`, will store user information, and inherit from `IdentityDbContext`:

```c#
public class AuthenticationDbContext : IdentityDbContext
{
    public AuthenticationDbContext(
        DbContextOptions<AuthenticationDbContext> options)
        : base(options)
    {
    }
}
```

Our objective is then to perform tests against these two databases.

## Setting Up Our WebApplicationFactory

Before we create our integration tests, we will need a test server where we can execute our test cases. This can be accomplished with `WebApplicationFactory`. This class is inheritable and is the main component to .NET integration tests. `WebApplicationFactory` works by spinning up a test server with capabilities, such as mocking our services. An example of such a test server is given below:

```c#
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder) =>
        builder.ConfigureTestServices();
}
```

In `ConfigureWebHost` we have the option of overriding services using `builder.ConfigureTestServices`. It is this method where we will instantiate our databases.

## Mocking Our Databases

To mock our databases, we will be using `Testcontainers.PostgreSql`. We can add this package with the following command:

```
dotnet add package Testcontainers.PostgreSql
```

This package will allow us to mock a Postgres database. Additionally, Testcontainers also supports SQL Server under their package `Testcontainers.MsSql`, or any docker database, via their package `Testcontainers`. 

`Testcontainers` works by creating a docker database, and executing our test cases within this docker database. Therefore, it is essential to have docker installed on the system you plan to execute your tests.

Furthermore, we will need a process for resetting our data after each test execution. Our test cases should be independent of one another, and the data from one test, should not, in any way, affect a subsequent test case.

A tool that will provide us with such capabilities is `Respawn`. This package can be added with the following command:

```
dotnet add package Respawn
```

With these tools installed, we are ready to start mocking our databases.

We will use NUnit as our test framework, however, any other framework will work just as well.

### Testcontainer Base Class

To begin, we will create our test database using the following base class:

```c#
public abstract class TestcontainersTestDatabase : ITestDatabase
{
    private readonly PostgreSqlContainer _container;
    private DbConnection _connection = null!;
    private string _connectionString = null!;
    private Respawner _respawner = null!;

    public TestcontainersTestDatabase()
    {
        _container = new PostgreSqlBuilder()
            .WithAutoRemove(true)
            .Build();
    }

    public async Task InitialiseAsync()
    {
        await _container.StartAsync();

        _connectionString = _container.GetConnectionString();

        _connection = new NpgsqlConnection(_connectionString);

        MigrateDatabase(_connection);

        await _connection.OpenAsync();

        _respawner = await Respawner.CreateAsync(_connection,
            new RespawnerOptions
            {
                DbAdapter = DbAdapter.Postgres,
                TablesToIgnore = new Respawn.Graph.Table[]
                { "__EFMigrationsHistory" },
            });
    }

    public abstract void MigrateDatabase(DbConnection connection);

    public DbConnection GetConnection() => _connection;

    public async Task ResetAsync()
    => await _respawner.ResetAsync(_connection);

    public async Task DisposeAsync()
    {
        await _connection.DisposeAsync();
        await _container.DisposeAsync();
    }
}

public interface ITestDatabase
{
    Task InitialiseAsync();

    DbConnection GetConnection();

    Task ResetAsync();

    Task DisposeAsync();
}
```

This class will be responsible for creating our Postgres database, and providing methods for resetting the state of our data. We also added a method to dispose of the container once we would like to destroy our database and the data contained within.

### Applying Migrations To Databases

If you are using an ORM via a code-first approach, we have also provided an abstract migrate method. The migrate method can be used like the following:

```c#
public class TestcontainersAuthTestDatabase : TestcontainersTestDatabase
{
    public override void MigrateDatabase(DbConnection connection)
    {
        var options = new DbContextOptionsBuilder<AuthenticationDbContext>()
            .UseNpgsql(connection)
            .Options;

        var context = new AuthenticationDbContext(options);

        context.Database.Migrate();
    }
}

public class TestcontainersAppTestDatabase : TestcontainersTestDatabase
{
    public override void MigrateDatabase(DbConnection connection)
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseNpgsql(connection)
            .Options;

        var context = new ApplicationDbContext(options);

        context.Database.Migrate();
    }
}
```

Once `InitialiseAsync` is executed, these two classes will be responsible for applying the respective migrations to the database. These methods will ensure our databases have the correct tables and schemas before we begin executing our tests.

## Instantiating Our Databases

Next, we will need to create a method to instantiate our databases. We can accomplish this with the following static class: 

```c#
public static class TestDatabaseCreation
{
    public static async Task<Dictionary<Type, ITestDatabase>> CreateAsync()
    {
        var appDatabase = new TestcontainersAppTestDatabase();
        var authDatabase = new TestcontainersAuthTestDatabase();

        await appDatabase.InitialiseAsync();
        await authDatabase.InitialiseAsync();

        return new Dictionary<Type, ITestDatabase>()
        {
            { typeof(ApplicationDbContext), appDatabase },
            { typeof(AuthDbContext), authDatabase },
        };
    }
}
```

The `CreateAsync` will be responsible for initializing our database, and returning each databases in a `Dictionary`, for further manipulation. This method will come in handy, as we continue setting up our `WebApplicationFactory`.

### Setting Up Our Databases

Our database is finally ready to be stubbed. Since the creation of a database is a relatively time-consuming process, we would like to perform this action, at most, once. For this reason, we create our database in `OneTimeSetup`:

```c#
[SetUpFixture]
public partial class Testing
{
    private static Dictionary<Type, ITestDatabase> _databases;
    private static CustomWebApplicationFactory _factory = null!;

    [OneTimeSetUp]
    public async Task RunBeforeAnyTests()
    {
        _databases = await TestDatabaseCreation.CreateAsync();
        _factory = new CustomWebApplicationFactory(_databases);
    }

    public static WebApplicationFactory<Program> WebApplicationFactory => _factory;

    [OneTimeTearDown]
    public async Task RunAfterAnyTests()
    {
        foreach (var database in _databases.Values)
        {
            await database.DisposeAsync();
        }
        await _factory.DisposeAsync();
    }
}
```

Notice how we are also disposing of our database on `OneTimeTearDown`. This will dispose of the database and containers prior to shutting down our test server.

### Registering / Mocking Our Databases

In the `OneTimeSetup`, you may have noticed that we are also starting our application test server. We have made an extra improvement to `CustomWebApplicationFactory` by passing in the databases to the constructor. This will allow us to register our databases with our test server, overriding the real database used in our application:

```c#
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    private readonly Dictionary<Type, ITestDatabase> _databases;

    public CustomWebApplicationFactory(Dictionary<Type, ITestDatabase> databases)
    {
        _databases = databases;
    }

    protected override void ConfigureWebHost(IWebHostBuilder builder) =>
        builder.ConfigureTestServices(services =>
        {
            foreach (var database in _databases)
            {
                if (database.Key == typeof(ApplicationDbContext))
                {
                    _ = services
                    .RemoveAll<DbContextOptions<ApplicationDbContext>>()
                    .AddDbContext<ApplicationDbContext>((sp, options) =>
                    {
                        _ = options.UseNpgsql(database.Value.GetConnection());
                    });
                }
                if (database.Key == typeof(AuthenticationDbContext))
                {
                    _ = services
                    .RemoveAll<DbContextOptions<AuthenticationDbContext>>()
                    .AddDbContext<AuthenticationDbContext>((sp, options) =>
                    {
                        _ = options.UseNpgsql(database.Value.GetConnection());
                    });
                }
            }
        });
}
```

### First Integration Test!!!

With all that in place, our test database should be fully functional. The only remaining piece is resetting our database after every test execution. We can perform this action in `Setup` using `ResetState`:

```c#
using static Testing;

[TestFixture]
public abstract class BaseTestFixture
{
    [SetUp]
    public async Task TestSetUp() => await ResetState();
}
```

Now, to perform an action against our test database, we can call any of our controllers that performs operations against our database:

```c#
using static Testing;

public class MyControllerTests : BaseTestFixture
{
    private HttpClient _client = null!;

    [SetUp]
    public void SetUp() =>
        _client = WebApplicationFactory.CreateClient();

    [Test]
    public async Task Test_MyController_Endpoint()
    {
        // Arrange

        // Act
        var response = await _client.GetAsync("/mycontroller/createentity");

        // Assert
        response.EnsureSuccessStatusCode();
        response.Content.Should().NotBeNull()
        // Add more assertions as needed
    }

    [TearDown]
    public void TearDown() =>
        _client.Dispose();
}
```

The database operations will be performed against our test database, not our actual database.

## Summary

In this article, we learned how to set up a test database for our integration tests. This test database was created using docker and utilized features, such as resetting the data after each test case, and even disposing of itself as soon as our tests were completed.

Thanks for reading!