---
title: "Creating Our First Unit Test in .NET"
date: 2023-08-06T00:00:00-07:00
draft: false
---

There is much to be said about testing in .NET.

A test can be categorized into one of three categories: 
1. Unit tests: These tests focus on testing individual units or components of the software in isolation,
2. Integration tests: These tests ensure that different units of the software work together as expected, 
3. End-to-end tests: These tests validate the entire system, including all its components, from end to end.

As a developer, most of our time will be spent creating unit tests. They form the foundation of our tests, are meant to execute quickly, and provide a quick feedback loop when a breaking change is introduced into the system. 

Integration and end-to-end tests are equally as important, however, they are more suspectable to breaking as our components and system evolves. Moreover, creating these types of test takes considerable more effort, especially as we start testing and adding more components to our system.

In this article, we will focus on creating unit tests, following current industry standards.

## Setting up a NUnit project

To showcase our first unit test, we will need a testing framework. For .NET, the primary choices are NUnit, XUnit, and MSTest. All of these have similar functionality, since I have mainly worked with NUnit, we will go ahead and use this testing framework.

Now, it is possible to have your tests within the same executable as your source code. However, this will grow your codebase unnecessarily. To keep things separate, it is best to create a separate testing project alongside your source code.

A typical structure looks like so:

```
TestingApp
|________src
    |________NUnitProject

|________test
    |________NUnitProject.Tests
```

For every project, we have a corresponding `Tests` project. We can further divide the tests project into `Project1.UnitTests`, `Project1.IntegrationTests`, etc. to clearly divide our tests. However, for simplicity, we will keep the current structure.

To create the above folder structure, we can execute the following commands:
```powershell
mkdir TestingApp
cd .\TestingApp

dotnet new sln -n TestingApp

mkdir src
cd ./src
dotnet new classlib -n NUnitProject

cd ..
dotnet sln add .\src\NUnitProject\NUnitProject.csproj

mkdir tests
cd ./tests
dotnet new nunit -n NUnitProject.Tests

cd ..
dotnet sln add .\tests\NUnitProject.Tests\NUnitProject.Tests.csproj

cd .\tests\NUnitProject.Tests\
dotnet add reference ..\..\src\NUnitProject\NUnitProject.csproj
```

This will create 2 projects, add the projects to the solution, and add a reference from source code to the test project.

## Creating Our First Unit Test

There are two important attributes in an NUnit test project: `TestFixture` and `Test`.

A `TestFixture` instructs NUnit that a class contains tests, while `Test`, which resides with a `TestFixture`, indicates that a method is a test method.

To begin our testing journey, we need a class to test. As this is our first test, we will keep the example simple and use a `Calculator` class with a single `Add` method.

```c#
namespace NUnitProject;

public class Calculator
{
    public int Add(int leftNumber, int rightNumber)
    {
        return -1;
    }
}
```

This simple method will add two numbers. In the spirit of making our tests fail first, we will return a dummy value. This will ensure that our tests do not trivially pass for all cases. 

Now we move onto the exciting part, the test class. In `NUnitProject.Tests`, let's begin by creating our first test using `TestFixture` and `Test`:

```c#
using NUnitProject;

namespace NUnitProject.Tests;

[TestFixture]
public class CalculatorTests
{
    private Calculator _calculator;

    [SetUp]
    public void Setup()
    {
        _calculator = new Calculator();
    }

    [Test]
    public void Add_InputIs0And0_Return0()
    {
        // Act
        var leftNumber = 0;
        var rightNumber = 0;

        // Arrange
        var addition = _calculator.Add(leftNumber, rightNumber);

        // Assert
        Assert.AreEqual(addition, 0);
    }
}
```

In the example above, we have created a test class called `CalculatorTests`. The class has a single test method called `Add_InputIs0And0_Return0`. The method is following the naming pattern of `MethodName_TestInputValues_ExpectedResult`, which is the naming pattern recommended by [Microsoft](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#naming-your-tests). Using this naming pattern, when our tests fail, we have a clear understanding of which tests are failing and how they are failing.

Executing our tests using `dotnet test`, will yield the following output:

```bash
A total of 1 test files matched the specified pattern.
  Failed Add_InputIs0And0_Return0 [45 ms]
  ...
  Failed!  - Failed:     1, Passed:     0, Skipped:     0, Total:     1, Duration: 45 ms - NUnitProject.Tests.dll (net7.0)
```

This is great! This proves that we do not have a trivial test case.

To make our test pass, we can implement `Add` correctly.

```c#
namespace NUnitProject;

public class Calculator
{
    public int Add(int leftNumber, int rightNumber)
    {
        return leftNumber + rightNumber;
    }
}
```

Now if we execute our tests again, we will see:

```bash
A total of 1 test files matched the specified pattern.

Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: 23 ms - NUnitProject.Tests.dll (net7.0)
```

This is even better! Now, we have a test case that will protect us if we ever decide to refactor `Add`.

## Summary

In this article, we covered how to create a simple test case using NUnit. We showed the red, green pattern by verifying our test cases do not trivially pass. We also recommended naming pattern to consider when creating test cases. 

For more best practices, please read Microsoft's [unit testing best practices](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices) article.

Thanks for reading!