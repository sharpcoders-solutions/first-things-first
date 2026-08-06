# 🧠 C# Fundamentals: Refactoring Legacy Code

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Clean Code, the principles of readable code  
- Testing (post 30) and Clean Architecture (post 33), applied from the start on new code  

Everything you've learned so far assumes projects starting from scratch. But most of a senior developer's career is spent on systems that already exist — without tests, without clear architecture, and that no one dares to touch. That's legacy code.

👉 **Let's learn Refactoring Legacy Code**

---

# 💡 Defining "legacy code"

👉 **Legacy code = code without automated tests, regardless of age**

```csharp
// This IS legacy code, even if written yesterday
public void ProcessOrder(Order o)
{
    // 200 lines with zero tests covering this behavior
}
```

👉 Michael Feathers (the reference author on this subject) defines it this way: if there are no tests, you don't know whether a change breaks something — and it's exactly that uncertainty that makes legacy code scary to touch

---

# 🏗️ The first step: building a safety net

```csharp
// Characterization test: documents the CURRENT behavior, even if it's odd
[Fact]
public void ProcessOrder_CurrentBehavior_DocumentsExistingState()
{
    var order = new Order { Amount = 100, Status = "X" }; // status "X" is weird, but that's what exists

    var result = _service.ProcessOrder(order);

    Assert.Equal("Y", result.Status); // documents what the code actually does today
}
```

👉 Remember post 30? Before refactoring, you write tests that capture the **current** behavior, even if it looks wrong — the goal here isn't to fix bugs yet, it's to have a safety net that warns you if you accidentally change something

---

# 🎯 Refactoring in small, safe steps

```csharp
// Step 1: extract a method, without changing behavior
public void ProcessOrder(Order order)
{
    ValidateOrder(order); // extracted, but still does exactly the same thing
    // rest of the original code, untouched
}

// Step 2: with the characterization test passing, now improve the logic
private void ValidateOrder(Order order)
{
    if (order.Amount <= 0)
        throw new ArgumentException("Invalid amount");
}
```

👉 Each step is small enough that you can run the characterization tests afterward and confirm nothing broke — legacy refactoring is about disciplined small steps, not a heroic one-shot rewrite

---

# 🔌 Seams: injection points in untestable code

```csharp
// ❌ Impossible to test in isolation
public class OrderService
{
    public void Process(Order order)
    {
        var client = new HttpClient(); // concrete dependency, right in the middle of the code
        client.PostAsJsonAsync("https://external-api.com/orders", order);
    }
}

// ✅ A "seam" created via dependency injection (remember the DI post?)
public class OrderService
{
    private readonly HttpClient _client;

    public OrderService(HttpClient client) // now you can inject a mock in tests
    {
        _client = client;
    }
}
```

👉 A "seam" is a point where you can alter behavior without editing the original code — usually introduced via dependency injection (the ASP.NET Core post), finally letting you test something that used to be impossible to isolate

---

# ⚖️ Refactor vs Rewrite

## 🔹 Incremental refactoring
- The system keeps working throughout the whole process  
- Risk distributed across small steps  
- Gains show up gradually  

## 🔹 Complete rewrite
- High risk — "big bang rewrites" frequently fail or take much longer than estimated  
- Only makes sense when the system is small enough, or truly unsalvageable  

👉 The near-universal industry recommendation is: prefer incremental refactoring. Complete rewrites have a poor track record of blowing past deadline and budget

---

# ⚠️ Common Mistakes

- Trying to refactor and add new functionality at the same time, mixing two types of change and making it hard to isolate the cause if something breaks  
- Refactoring without characterization tests first, removing the only available safety net  
- Making changes too large at once, making it difficult to pinpoint exactly what caused a regression  
- Ignoring legacy code until it becomes a crisis — continuous, small refactoring is cheaper than an emergency rewrite  

---

# 📌 Conclusion

- Legacy code is defined by the absence of tests, not by age  
- Characterization tests document current behavior before any change  
- Seams via dependency injection turn previously untestable code into testable code  
- Incremental refactoring, in small steps, is preferable to complete rewrites in most cases  

👉 With these techniques, legacy code stops being scary territory and becomes a system you can improve with confidence, one safe step at a time

---

# 🔥 Next Step

Now that you know how to work with legacy code, the next level is:

👉 **C# Fundamentals: Mentoring and Technical Leadership**

Here you'll learn to multiply your impact beyond the code itself, helping other developers grow.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
