# 07 - Unit Testing

## Learning Goal

Write focused C# unit tests with xUnit, run them with `dotnet test`, and use test failures as fast feedback about public behavior.

By the end of this lesson, you should be able to:

- Explain what a unit test verifies and why it should be fast, isolated, repeatable, self-checking, and focused.
- Create a class library and xUnit test project with the .NET CLI.
- Reference the production project from the test project.
- Write xUnit tests with `[Fact]`, `[Theory]`, and `[InlineData]`.
- Use Arrange, Act, Assert to organize tests.
- Use `Assert.Equal`, `Assert.True`, `Assert.False`, and `Assert.Throws`.
- Name tests by method, scenario, and expected behavior.
- Avoid tests that depend on private implementation details, test order, manual inspection, or external systems.

## Why Unit Testing Matters

A unit test checks a small piece of behavior, usually a public method on a class, without requiring the whole application to run. Good unit tests make design feedback cheap. If a calculation changes, a validation rule breaks, or a refactor changes behavior by accident, the test suite can catch the problem before a user does.

Unit tests are most useful when they are:

- Fast: they run often enough that developers actually use them.
- Isolated: they do not require a database, network, file system, console input, system clock, or random value unless that dependency is abstracted.
- Repeatable: the same code and inputs produce the same result every run.
- Self-checking: the test decides pass or fail with assertions.
- Focused: one test explains one behavior.

.NET has two related testing pieces. A test framework defines how you write tests; common choices include xUnit, MSTest, NUnit, and TUnit. A test platform or runner discovers and runs those tests; .NET commonly uses VSTest or Microsoft.Testing.Platform. The platform details are version-sensitive, so this lesson stays with ordinary SDK-created xUnit projects and `dotnet test`.

## Create A Testable Project

Start with a solution, one class library for production code, and one xUnit project for tests.

```powershell
dotnet new sln
dotnet new classlib -o GradeBook
dotnet new xunit -o GradeBook.Tests
dotnet sln add GradeBook/GradeBook.csproj
dotnet sln add GradeBook.Tests/GradeBook.Tests.csproj
dotnet add GradeBook.Tests/GradeBook.Tests.csproj reference GradeBook/GradeBook.csproj
dotnet test
```

The project reference is important. The test project must reference the production project before test code can use production classes.

## A Small Class Under Test

Unit tests are easiest to learn with deterministic code. This `GradeCalculator` has no file access, no database, no network calls, no current time, no random values, and no console input. Given the same score, it always returns the same grade.

Put this class in the `GradeBook` project.

```csharp
using System;

namespace GradeBook;

public class GradeCalculator
{
    public string GetLetterGrade(int score)
    {
        if (score < 0 || score > 100)
        {
            throw new ArgumentOutOfRangeException(nameof(score), "Score must be between 0 and 100.");
        }

        if (score >= 90)
        {
            return "A";
        }

        if (score >= 80)
        {
            return "B";
        }

        if (score >= 70)
        {
            return "C";
        }

        if (score >= 60)
        {
            return "D";
        }

        return "F";
    }

    public bool IsPassing(int score)
    {
        return GetLetterGrade(score) != "F";
    }
}
```

The class has public behavior worth testing:

- Valid scores return the expected letter grade.
- Invalid scores throw `ArgumentOutOfRangeException`.
- Passing scores return `true` from `IsPassing`.
- Failing scores return `false` from `IsPassing`.

## Write xUnit Tests

xUnit uses attributes to mark test methods. `[Fact]` marks a test with one fixed case. `[Theory]` marks a parameterized test, and `[InlineData]` supplies input values and expected values.

Put this test class in the `GradeBook.Tests` project.

```csharp
using System;
using GradeBook;
using Xunit;

namespace GradeBook.Tests;

public class GradeCalculatorTests
{
    [Fact]
    public void GetLetterGrade_PerfectScore_ReturnsA()
    {
        // Arrange
        GradeCalculator calculator = new GradeCalculator();

        // Act
        string grade = calculator.GetLetterGrade(100);

        // Assert
        Assert.Equal("A", grade);
    }

    [Theory]
    [InlineData(95, "A")]
    [InlineData(82, "B")]
    [InlineData(73, "C")]
    [InlineData(64, "D")]
    [InlineData(41, "F")]
    public void GetLetterGrade_ValidScore_ReturnsExpectedGrade(int score, string expectedGrade)
    {
        GradeCalculator calculator = new GradeCalculator();

        string grade = calculator.GetLetterGrade(score);

        Assert.Equal(expectedGrade, grade);
    }

    [Fact]
    public void IsPassing_PassingScore_ReturnsTrue()
    {
        GradeCalculator calculator = new GradeCalculator();

        bool isPassing = calculator.IsPassing(60);

        Assert.True(isPassing);
    }

    [Fact]
    public void IsPassing_FailingScore_ReturnsFalse()
    {
        GradeCalculator calculator = new GradeCalculator();

        bool isPassing = calculator.IsPassing(59);

        Assert.False(isPassing);
    }

    [Theory]
    [InlineData(-1)]
    [InlineData(101)]
    public void GetLetterGrade_OutOfRangeScore_ThrowsArgumentOutOfRangeException(int score)
    {
        GradeCalculator calculator = new GradeCalculator();

        Assert.Throws<ArgumentOutOfRangeException>(() => calculator.GetLetterGrade(score));
    }
}
```

Run the tests:

```powershell
dotnet test
```

Expected outcome:

```text
The test run should build both projects, run the xUnit tests, and report that the tests passed.
```

The exact output includes SDK, runner, target framework, timing, and test count details that can vary by installed .NET version and template updates.

## Arrange, Act, Assert

Arrange, Act, Assert is a simple way to keep tests readable:

- Arrange: create the object and input values.
- Act: call the method being tested.
- Assert: verify the result.

In this test, the three parts are easy to see:

```csharp
[Fact]
public void GetLetterGrade_PerfectScore_ReturnsA()
{
    // Arrange
    GradeCalculator calculator = new GradeCalculator();

    // Act
    string grade = calculator.GetLetterGrade(100);

    // Assert
    Assert.Equal("A", grade);
}
```

Small tests do not always need comments for each section, but the structure should still be visible. A reader should be able to identify the setup, the behavior under test, and the expected result quickly.

## Test Names

A useful test name describes the method, scenario, and expected behavior:

```text
MethodName_Scenario_ExpectedBehavior
```

Examples:

- `GetLetterGrade_PerfectScore_ReturnsA`
- `GetLetterGrade_OutOfRangeScore_ThrowsArgumentOutOfRangeException`
- `IsPassing_FailingScore_ReturnsFalse`

This naming style makes failures easier to scan. When a test fails, the test name should already explain what behavior broke.

## Facts And Theories

Use `[Fact]` when one example tells the story clearly:

```csharp
[Fact]
public void IsPassing_FailingScore_ReturnsFalse()
{
    GradeCalculator calculator = new GradeCalculator();

    bool isPassing = calculator.IsPassing(59);

    Assert.False(isPassing);
}
```

Use `[Theory]` when the same behavior should be checked with multiple inputs:

```csharp
[Theory]
[InlineData(90, "A")]
[InlineData(89, "B")]
[InlineData(60, "D")]
[InlineData(59, "F")]
public void GetLetterGrade_BoundaryScore_ReturnsExpectedGrade(int score, string expectedGrade)
{
    GradeCalculator calculator = new GradeCalculator();

    string grade = calculator.GetLetterGrade(score);

    Assert.Equal(expectedGrade, grade);
}
```

The theory avoids copying the same test method four times. It also avoids putting loops or conditions inside the test body. The data table says what cases matter.

## Testing Exceptions

Use `Assert.Throws<TException>` when the expected behavior is an exception.

```csharp
[Fact]
public void GetLetterGrade_ScoreAboveOneHundred_ThrowsArgumentOutOfRangeException()
{
    GradeCalculator calculator = new GradeCalculator();

    Assert.Throws<ArgumentOutOfRangeException>(() => calculator.GetLetterGrade(101));
}
```

The lambda delays the method call until xUnit runs the assertion. If the method does not throw `ArgumentOutOfRangeException`, the test fails.

## What Not To Unit Test Directly

Prefer testing public behavior over private implementation details. If a class has a private helper method, test the public method that uses it. Private methods can change during refactoring even when the public behavior stays correct.

Avoid direct dependencies on external systems in unit tests:

- File system
- Database
- Network
- Current clock time
- Random values
- Console input or output

Those dependencies can make tests slow, flaky, or environment-specific. If a class needs one of them, wrap it behind an interface or pass in a deterministic value so the unit test can control the behavior.

## Common Mistakes

- Forgetting the project reference from the test project to the production project. If the test cannot find the class under test, check `dotnet add GradeBook.Tests/GradeBook.Tests.csproj reference GradeBook/GradeBook.csproj`.
- Testing private implementation details instead of public behavior.
- Checking too many behaviors in one test. A failure should point to one clear problem.
- Adding loops, `if` statements, or `switch` statements inside tests when a `[Theory]` with `[InlineData]` would be clearer.
- Depending on test order. Each test should arrange everything it needs.
- Printing values and manually inspecting output instead of using assertions.
- Treating coverage percentage as proof that behavior is correct. Coverage can show what code ran, not whether the assertions were meaningful.
- Using live file system, database, network, clock, random, or console dependencies in unit tests without abstraction.

## Practical Exercise

Build and test a `DiscountCalculator`.

Requirements:

1. Create a solution, a class library named `StorePricing`, and an xUnit test project named `StorePricing.Tests`.
2. Add both projects to the solution.
3. Add a project reference from `StorePricing.Tests` to `StorePricing`.
4. Implement a public `DiscountCalculator` class.
5. Add `CalculateFinalPrice(decimal price, decimal discountPercent)`.
6. Throw `ArgumentOutOfRangeException` when `price` is negative.
7. Throw `ArgumentOutOfRangeException` when `discountPercent` is less than `0` or greater than `100`.
8. Return the final price after applying the percentage discount.
9. Write at least one `[Fact]`.
10. Write at least one `[Theory]` with `[InlineData]`.
11. Write at least one `Assert.Throws<ArgumentOutOfRangeException>` test.
12. Run `dotnet test`.

Keep the tests deterministic. Test the public behavior of `CalculateFinalPrice`; do not test private helper methods.

## Worked Answer

Create the projects:

```powershell
dotnet new sln
dotnet new classlib -o StorePricing
dotnet new xunit -o StorePricing.Tests
dotnet sln add StorePricing/StorePricing.csproj
dotnet sln add StorePricing.Tests/StorePricing.Tests.csproj
dotnet add StorePricing.Tests/StorePricing.Tests.csproj reference StorePricing/StorePricing.csproj
```

Put this class in the `StorePricing` project:

```csharp
using System;

namespace StorePricing;

public class DiscountCalculator
{
    public decimal CalculateFinalPrice(decimal price, decimal discountPercent)
    {
        if (price < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(price), "Price cannot be negative.");
        }

        if (discountPercent < 0 || discountPercent > 100)
        {
            throw new ArgumentOutOfRangeException(
                nameof(discountPercent),
                "Discount percent must be between 0 and 100.");
        }

        decimal discountAmount = price * (discountPercent / 100m);
        return price - discountAmount;
    }
}
```

Put this test class in the `StorePricing.Tests` project:

```csharp
using System;
using StorePricing;
using Xunit;

namespace StorePricing.Tests;

public class DiscountCalculatorTests
{
    [Fact]
    public void CalculateFinalPrice_ZeroDiscount_ReturnsOriginalPrice()
    {
        DiscountCalculator calculator = new DiscountCalculator();

        decimal finalPrice = calculator.CalculateFinalPrice(80.00m, 0m);

        Assert.Equal(80.00m, finalPrice);
    }

    [Theory]
    [InlineData(100, 10, 90)]
    [InlineData(50, 20, 40)]
    [InlineData(19, 100, 0)]
    public void CalculateFinalPrice_ValidDiscount_ReturnsDiscountedPrice(
        decimal price,
        decimal discountPercent,
        decimal expectedFinalPrice)
    {
        DiscountCalculator calculator = new DiscountCalculator();

        decimal finalPrice = calculator.CalculateFinalPrice(price, discountPercent);

        Assert.Equal(expectedFinalPrice, finalPrice);
    }

    [Theory]
    [InlineData(-1, 10)]
    [InlineData(10, -1)]
    [InlineData(10, 101)]
    public void CalculateFinalPrice_InvalidInput_ThrowsArgumentOutOfRangeException(
        decimal price,
        decimal discountPercent)
    {
        DiscountCalculator calculator = new DiscountCalculator();

        Assert.Throws<ArgumentOutOfRangeException>(
            () => calculator.CalculateFinalPrice(price, discountPercent));
    }
}
```

Run the tests:

```powershell
dotnet test
```

Expected outcome:

```text
The solution should build successfully, discover the xUnit tests, run them, and report a passing test run.
```

Teaching notes:

- The `[Fact]` checks one simple behavior: a zero discount keeps the original price.
- The `[Theory]` checks several valid inputs without adding loops or conditional logic to the test.
- `Assert.Equal` compares the expected final price to the returned final price.
- `Assert.Throws<ArgumentOutOfRangeException>` verifies validation behavior for invalid prices and discount percentages.
- Each test creates its own `DiscountCalculator`, so no test depends on another test's state or order.
- The tests call only the public `CalculateFinalPrice` method. They do not care whether the implementation uses a local variable, a private helper, or a different formula internally as long as the public behavior stays correct.

## Sources

- Microsoft Learn: [Testing in .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- Microsoft Learn: [Unit testing C# in .NET using dotnet test and xUnit](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-xunit)
- Microsoft Learn: [Get started with C# and MSTest](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-mstest)
- Microsoft Learn: [Best practices for writing unit tests](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
- Microsoft Learn: [Testing with `dotnet test`](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test)
- Microsoft Learn: [Microsoft.Testing.Platform vs VSTest](https://learn.microsoft.com/en-us/dotnet/core/testing/test-platforms-overview)
