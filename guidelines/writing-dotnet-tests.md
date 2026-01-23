# Writing .NET Tests

We follow all guidelines published by Microsoft. These guidelines are based on the [Unit testing best practices for .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)

## Tools

In MORYX we use the following tools and packages to write tests:

- NUnit as Test-framework
- Moq as mocking - framework

## Naming Convention 

Use the pattern: **`MethodName_Scenario_ExpectedBehavior`**

```csharp
// Bad
public void Test_Single() { }

// Good
public void Add_SingleNumber_ReturnsSameNumber() { }
public void Withdraw_ValidAmount_DecreasesBalance() { }
```

### Naming Flexibility

The `MethodName_Scenario_ExpectedBehavior` pattern is a **recommendation**, not a strict requirement. The real goal is clarity - anyone should understand what a test does from its name.

**Simpler Alternatives**

**Option 1: Just describe the behavior**

```cs
public void ReturnsZeroForEmptyString() { }
public void ThrowsOnNegativeNumbers() { }
```

**Option 2: Scenario and result**

```cs
public void EmptyString_ReturnsZero() { }
public void NegativeNumber_Throws() { }
```

**Option 3: Natural language style**

```cs
public void Should_return_zero_when_string_is_empty() { }
```

#### What Actually Matters

| Important                         | Less Important                      |
| --------------------------------- | ----------------------------------- |
| Test name explains the intent     | Exact naming format                 |
| Failure message is understandable | Including method name in test name  |
| Consistent style within project   | Following any specific pattern      |

## Arrange-Act-Assert Pattern

Microsoft recommends structuring tests with clear AAA sections:

```csharp
[Fact]
public void Add_EmptyString_ReturnsZero()
{
    // Arrange
    var calculator = new StringCalculator();

    // Act
    var actual = calculator.Add("");

    // Assert
    Assert.Equal(0, actual);
}
```

- Arrange your objects, create, and configure them as necessary
- Act on an object
- Assert that something is as expected

**Note:** Comments like `// Arrange`, `// Act`, `// Assert` are optional but help readability.

**Benefits**

- Prevents accidental mixing of phases
- Consistency across tests
- Tests serve as documentation
- Easier debugging and failure diagnosis
- Speeds up debugging
- Guides new team members

## Fill Test Description

Since method names often become very long and hard to read, it is good to provide a value for the `Description` property of the `Test` or `TestCase` attribute.

## Sample

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
