# 🧠 C# Fundamentals: Mutation Testing

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Testcontainers for testing against real dependencies  
- Code coverage as a quality metric for tests  

Your test suite has 95% coverage. Does that mean your tests are good? Not necessarily — coverage only measures whether a line **ran**, not whether the test actually verified the right behavior. Mutation Testing answers that question.

👉 **Let's learn Mutation Testing**

---

# 💡 The empty coverage problem

```csharp
public class DiscountCalculator
{
    public decimal Calculate(decimal amount)
    {
        if (amount > 100)
            return amount * 0.9m;

        return amount;
    }
}
```

```csharp
[Fact]
public void Calculate_ShouldRunWithoutError()
{
    var result = _calculator.Calculate(150);
    Assert.NotNull(result); // ❌ doesn't verify the actual value
}
```

👉 This test gives 100% coverage on the whole class, but wouldn't catch anything if someone swapped `0.9m` for `0.5m` by mistake — `Assert.NotNull` always passes

---

# 🧬 What is Mutation Testing?

👉 **Mutation Testing = deliberately altering small pieces of code (mutants) and checking whether the tests detect the change**

Examples of automatic mutations:

```csharp
// Original
if (amount > 100)

// Mutant 1: swaps the operator
if (amount >= 100)

// Mutant 2: swaps the constant
if (amount > 0)

// Mutant 3: negates the condition
if (amount < 100)
```

If the tests still pass with any of these mutations, the mutant **survived** — a sign that piece of code isn't actually being tested.

---

# 🏗️ Using Stryker.NET

```bash
dotnet tool install -g dotnet-stryker
```

```bash
cd MyProject.Tests
dotnet stryker
```

```
Mutation testing report:
  Total mutants: 42
  Killed: 35 (83%)
  Survived: 7 (17%)

  Mutation Score: 83%
```

👉 Unlike line coverage, Mutation Score measures the **quality** of assertions — how many of the simulated bugs the tests actually caught

---

# 🎯 Fixing a weak test

```csharp
// ❌ Before: mutant survives
[Fact]
public void Calculate_ShouldRunWithoutError()
{
    var result = _calculator.Calculate(150);
    Assert.NotNull(result);
}

// ✅ After: mutant dies
[Fact]
public void Calculate_WithAmountAbove100_ShouldApply10PercentDiscount()
{
    var result = _calculator.Calculate(150);
    Assert.Equal(135m, result);
}

[Fact]
public void Calculate_AtThe100Boundary_ShouldNotApplyDiscount()
{
    var result = _calculator.Calculate(100);
    Assert.Equal(100m, result);
}
```

👉 The second test, testing exactly the `100` boundary, is what kills the mutant that swapped `>` for `>=` — remember the edge case tests we've discussed since post 30? Mutation testing forces you to write them

---

# ⚠️ Common Mistakes

- Running mutation testing on the entire codebase at once, making the process too slow to run frequently  
- Blindly chasing a 100% Mutation Score, when some mutants are equivalent and genuinely don't matter  
- Using mutation testing as a substitute for coverage, when they answer different, complementary questions  
- Running it on every CI commit, when the computational cost is high — better to run it periodically or on critical code  

---

# 📌 Conclusion

- Code coverage measures execution, not test quality  
- Mutation Testing deliberately alters code and checks whether tests detect the change  
- Surviving mutants indicate weak tests, without assertions that actually validate behavior  
- Stryker.NET automates this process for C# projects  

👉 With Mutation Testing, you stop asking "did my tests run?" and start asking "do my tests actually catch bugs?"

---

# 🔥 Next Step

Now that you can measure the real quality of your tests, the next level is:

👉 **C# Fundamentals: Roslyn Analyzers**

Here you'll learn to create your own static analysis rules, applied automatically during compilation.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
