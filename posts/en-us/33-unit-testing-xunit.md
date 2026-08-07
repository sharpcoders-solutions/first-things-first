# 🧠 C# Fundamentals: Unit Testing with xUnit

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- The five SOLID principles  
- Essential design patterns (Singleton, Factory, Strategy, Repository)  

You already know how to write well-designed code. But how do you make **sure** it keeps working once someone else (or you, six months later) touches it?

👉 **That's exactly what automated tests are for**

This is an important milestone: post number 30 in this track. From here on, you don't just write code — you write code that **proves** it works.

---

# 💡 Why test?

👉 **Unit test = code that automatically verifies that other code behaves as expected**

Without tests, every change is a leap of faith. With tests:

✅ You refactor without fear of breaking something hidden  
✅ Bugs get caught before reaching production  
✅ The test becomes **living documentation** of how the code should behave  

👉 Remember SOLID and design patterns? Decoupled code (thanks to interfaces and DIP) is exactly what makes tests easy to write — the previous two posts laid the groundwork for this one

---

# 🏗️ Setting up the test project

```bash
dotnet new xunit -o MyProject.Tests
cd MyProject.Tests
dotnet add reference ../MyProject/MyProject.csproj
```

👉 The `xunit` template already comes with everything set up: the testing framework, the runner, and the needed dependencies

---

# 🧪 Your first test

```csharp
public class CalculatorTests
{
    [Fact]
    public void Sum_TwoPositiveNumbers_ReturnsCorrectSum()
    {
        // Arrange
        var calculator = new Calculator();

        // Act
        int result = calculator.Sum(2, 3);

        // Assert
        Assert.Equal(5, result);
    }
}
```

## 🔹 The AAA pattern

- **Arrange** → sets up the data and objects needed  
- **Act** → runs the action being tested  
- **Assert** → checks whether the result is what was expected  

👉 Every well-written test follows this structure, even without the explicit comments

## 🔹 Naming convention

`MethodUnderTest_Scenario_ExpectedResult` — the test's name already tells the story of what's being verified, without needing to open the code

---

# 🎯 `[Fact]` vs `[Theory]`: testing multiple scenarios

```csharp
public class CalculatorTests
{
    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(-1, 1, 0)]
    [InlineData(0, 0, 0)]
    public void Sum_MultipleValues_ReturnsCorrectSum(int a, int b, int expected)
    {
        var calculator = new Calculator();

        int result = calculator.Sum(a, b);

        Assert.Equal(expected, result);
    }
}
```

👉 `[Fact]` tests **one** fixed scenario. `[Theory]` + `[InlineData]` runs the **same test** multiple times, with different inputs — avoiding duplicated code for each case

---

# ✅ The most commonly used assertions

```csharp
Assert.Equal(5, result);                 // values are equal
Assert.True(condition);                   // condition is true
Assert.Null(obj);                         // object is null
Assert.NotNull(obj);                      // object is not null

Assert.Throws<InvalidOperationException>(() =>
{
    account.Withdraw(1000); // should throw an exception
});
```

👉 `Assert.Throws` is essential for testing the error scenarios you learned about in the exceptions post — a good test suite covers both the happy path and the expected failures

---

# 🎭 Testing with dependencies: Mocking

Remember the `INotifier` and `OrderProcessor` example from the SOLID post (Dependency Inversion)? That exact decoupling is what makes testing possible:

```csharp
class FakeNotifier : INotifier
{
    public bool WasCalled { get; private set; }

    public void Send(string message)
    {
        WasCalled = true;
    }
}
```

```csharp
[Fact]
public void Process_ValidOrder_CallsNotifier()
{
    // Arrange
    var fakeNotifier = new FakeNotifier();
    var processor = new OrderProcessor(fakeNotifier);

    // Act
    processor.Process();

    // Assert
    Assert.True(fakeNotifier.WasCalled);
}
```

👉 `OrderProcessor` receives `INotifier` through its constructor (Dependency Inversion), so in the test you pass a **fake** version, without needing to send a real email or SMS

## 🔹 Mocking libraries (Moq)

Writing a fake class by hand works, but it gets repetitive for larger interfaces. The **Moq** library generates these "doubles" automatically:

```csharp
var mockNotifier = new Mock<INotifier>();
var processor = new OrderProcessor(mockNotifier.Object);

processor.Process();

mockNotifier.Verify(n => n.Send(It.IsAny<string>()), Times.Once);
```

👉 `Verify` confirms that the `Send` method was called exactly once — without needing to write a fake class by hand

---

# ⚠️ Common Mistakes

- Testing implementation details instead of behavior (this breaks the test on every refactor, even without a real bug)  
- Creating tests that depend on each other (a test should never require another one to run first)  
- Only testing the happy path, ignoring errors and edge cases  
- Skipping tests for code that's "too simple to break" — that's exactly the code that tends to break without warning  
- Naming tests generically (`Test1`, `TestSum`) instead of describing the scenario and expected result  

---

# 📌 Conclusion

- Unit tests automate verifying that your code behaves as expected  
- The AAA pattern (Arrange, Act, Assert) structures every test  
- `[Fact]` tests one fixed scenario; `[Theory]` + `[InlineData]` tests multiple scenarios with the same code  
- Code decoupled through interfaces (SOLID) is what makes mocks and tests possible  
- `Assert.Throws` guarantees that expected errors actually happen  

👉 With automated tests, your code stops relying on "it looks like it works" and starts **proving** that it does

---

# 🔥 Next Step

Now that you know how to validate your code automatically, the next level is:

👉 **C# Fundamentals: Building Your First API with ASP.NET Core**

Here you'll move beyond isolated code and build your first real web application, applying everything you've learned so far.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
