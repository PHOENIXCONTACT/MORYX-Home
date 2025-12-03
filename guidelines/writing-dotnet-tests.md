# Writing .NET Tests

We follow all guidelines published by Microsoft. These guidelines are based on the [Unit testing best practices for .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)

## Tools

In MORYX we use the following tools and packages to write tests:

- NUnit as Test-framework

## Fill Test Description

Since method names often become very long and hard to read, it is mandatory to provide a value for the `Description` property of the `Test` or `TestCase` attribute.

## Arrange your tests

As described in the _best practices_: The "Arrange, Act, Assert" pattern is a common approach for writing unit tests. As the name implies, the pattern consists of three main tasks: 

- Arrange your objects, create, and configure them as necessary
- Act on an object
- Assert that something is as expected

**Benefits**

- Prevents accidental mixing of phases
- Consistency across tests
- Tests serve as documentation
- Easier debugging and failure diagnosis
- Speeds up debugging

As not explictly written down in the _best practices_, in MORYX we write down the comments `// Arrange // Act // Assert`.

- Immediate visual clarity
- Helps differentiate phases in complex tests
- Guides new team members
- Improves maintainability over time
- Supports team-wide understanding and keeps tests self-explanatory.

````cs
[Test(Description = "Adding empty string returns zero.")]
public void Add_EmptyString_ReturnsZero()
{
    // Arrange
    var stringCalculator = new StringCalculator();

    // Act
    var actual = stringCalculator.Add("");

    // Assert
    Assert.That(actual, Is.EqualTo(0));
}
````

Always include `// Arrange`, `// Act`, `// Assert` comments in unit tests, especially for anything beyond trivial one-liners. This is a lightweight convention that pays off in readability, maintainability, and debugging efficiency.
