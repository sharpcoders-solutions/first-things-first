# 🧠 C# Fundamentals: Functional Programming in C#

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Channels for asynchronous coordination  
- Records and immutability (post 30), LINQ (post 19), delegates and lambdas (post 28)  

C# is an object-oriented language at its core, but throughout this track you've already used several functional concepts without formally naming them. Time to connect the dots.

👉 **Let's learn Functional Programming in C#**

---

# 💡 The pillars of functional programming

## 🔹 Immutability

```csharp
// We've already used this since post 30
public record Order(int Id, decimal Amount, string Status);

var order = new Order(1, 100m, "New");
var updatedOrder = order with { Status = "Confirmed" }; // creates a new object
```

👉 Instead of modifying the original object, you create a new version — the same `with` you've used since the records post

## 🔹 Pure functions

```csharp
// ✅ Pure: same input, always the same output, no side effects
public static decimal CalculateDiscount(decimal amount, decimal percentage)
    => amount * (1 - percentage);

// ❌ Impure: depends on external state and has a side effect
private decimal _accumulatedTotal;
public decimal CalculateImpureDiscount(decimal amount)
{
    _accumulatedTotal += amount; // side effect
    return amount * 0.9m;
}
```

👉 Pure functions are easier to test (remember post 33?) — no mocks, no setup, just input and output

---

# 🏗️ Function composition

```csharp
Func<decimal, decimal> applyDiscount = amount => amount * 0.9m;
Func<decimal, decimal> applyShipping = amount => amount + 15m;
Func<decimal, decimal> round = amount => Math.Round(amount, 2);

Func<decimal, decimal> calculateTotal = amount =>
    round(applyShipping(applyDiscount(amount)));

var total = calculateTotal(100m); // 100 → 90 → 105 → 105.00
```

👉 Remember the delegates post (25)? `Func<T, TResult>` is already the foundation of this — composition is simply chaining small pure functions to form a larger behavior

---

# 🎯 LINQ is functional programming

```csharp
var expensiveOrders = orders
    .Where(o => o.Amount > 100)        // filters, without modifying the original list
    .Select(o => o with { Status = "Priority" }) // transforms, without mutation
    .OrderByDescending(o => o.Amount)
    .ToList();
```

👉 You've used LINQ since post 19 — and every method (`Where`, `Select`, `OrderBy`) is a pure function that takes a sequence and returns another, without modifying the original. That's functional programming in action, even without that label at the time

---

# 🚫 Avoiding exceptions as control flow: Option/Result

```csharp
public record Result<T>(bool Success, T? Value, string? Error);

public Result<Order> FindOrder(int id)
{
    var order = _repository.Find(id);

    return order is not null
        ? new Result<Order>(true, order, null)
        : new Result<Order>(false, null, "Order not found");
}
```

```csharp
var result = FindOrder(123);

if (result.Success)
    Console.WriteLine(result.Value);
else
    Console.WriteLine(result.Error);
```

👉 Instead of throwing exceptions for expected control flow (remember post 26, about when to use exceptions?), the `Result` pattern makes the error path explicit in the return type — the caller is forced to handle both cases

---

# ⚠️ Common Mistakes

- Trying to turn C# into a 100% functional language, ignoring that OOP is still the language's core paradigm  
- Overusing function composition to the point where code becomes hard to debug  
- Using `Result<T>` for every error, even genuinely exceptional situations that should throw an exception  
- Forgetting that immutability has an allocation cost — use it judiciously in high-performance loops  

---

# 📌 Conclusion

- Functional programming isn't a separate language, it's a set of principles C# already embraces  
- Immutability, pure functions, and composition have already appeared throughout this track since records and LINQ  
- The `Result` pattern makes expected errors explicit, instead of using exceptions as control flow  
- C# is multi-paradigm — the goal is using the best concept for each problem, not picking a "side"  

👉 With functional programming, you see that much of modern C# is already functional by nature, even while writing classes all day

---

# 🔥 Next Step

Now that you connect C#'s functional concepts, the next level is:

👉 **C# Fundamentals: Covariance and Contravariance**

Here you'll learn how generics can be flexible with type hierarchies, and when that's safe.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
