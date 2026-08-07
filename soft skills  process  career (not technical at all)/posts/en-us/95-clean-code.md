# 🧠 C# Fundamentals: Clean Code

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Cryptography and data protection  
- 94 posts of specific techniques, each solving a particular problem  

Time to take a step back. Every technique in this track — SOLID (28), Clean Architecture (33), testing (30) — serves a bigger goal: code that humans can read, understand, and maintain. That's Clean Code.

👉 **Let's consolidate the principles of Clean Code**

---

# 💡 Code is read far more than it's written

```csharp
// ❌ Written fast, read with difficulty
public decimal Calc(decimal v, int t)
{
    if (t == 1) return v * 0.9m;
    if (t == 2) return v * 0.8m;
    return v;
}
```

```csharp
// ✅ Written with the same effort, read instantly
public decimal CalculateDiscountedAmount(decimal originalAmount, CustomerType customerType)
{
    return customerType switch
    {
        CustomerType.Premium => originalAmount * 0.9m,
        CustomerType.VIP => originalAmount * 0.8m,
        _ => originalAmount
    };
}
```

👉 Remember the pattern matching from post 27? Descriptive names and a clear structure don't take more time to write — they just require thinking a bit more about who's going to read this later

---

# 🏗️ Names that tell the truth

```csharp
// ❌ Generic names hide intent
var d = DateTime.Now;
var list = GetData();
var flag = true;

// ✅ Specific names reveal intent
var dueDate = DateTime.Now.AddDays(30);
var pendingOrders = GetPendingOrders();
var userIsAuthenticated = true;
```

👉 A good name eliminates the need for a comment explaining what the variable represents — the code itself already documents the intent

---

# 🎯 Small functions, doing one thing

```csharp
// ❌ One function doing several things
public void ProcessOrder(Order order)
{
    // validate
    if (order.Amount <= 0) throw new Exception("Invalid amount");
    
    // calculate discount
    var discount = order.Customer.Type == CustomerType.VIP ? 0.1m : 0m;
    order.Amount -= order.Amount * discount;
    
    // save
    _context.Orders.Add(order);
    _context.SaveChanges();
    
    // notify
    _emailService.Send(order.Customer.Email, "Order confirmed");
}
```

```csharp
// ✅ Each function with a single responsibility (remember SRP, post 28?)
public async Task ProcessOrder(Order order)
{
    ValidateOrder(order);
    ApplyDiscount(order);
    await SaveOrder(order);
    await NotifyCustomer(order);
}
```

👉 This is the same idea as the Single Responsibility Principle from post 28 — but applied at the method level, not just the class level. Every function should do one thing, and the function's name should say exactly what

---

# 🚫 Comments that compensate for bad code

```csharp
// ❌ Comment compensating for a bad name
// checks if the order can be cancelled
if (o.St == 2 && (DateTime.Now - o.CreatedDt).Days < 7)

// ✅ Code that explains itself, no comment needed
if (order.CanBeCancelled())
```

```csharp
public bool CanBeCancelled() =>
    Status == OrderStatus.Confirmed && (DateTime.Now - CreatedAt).Days < 7;
```

👉 Comments aren't inherently bad, but a comment explaining "what" the code does is usually a sign the code should be rewritten to explain itself — save comments for explaining "why," not "what"

---

# 🔍 Where Clean Code connects with everything you've already learned

```
Clear names + small functions → easier tests to write (post 30)
Single responsibility → classes easier to test in isolation (post 28, SOLID)
Self-explanatory code → less need for separate documentation (post 49)
Consistent structure → faster code reviews (post 11, Git Workflow)
```

👉 Clean Code isn't an isolated technique — it's the thread that runs through and makes every other technique in this track easier to apply consistently

---

# ⚠️ Common Mistakes

- Chasing "clean code" as an absolute dogma, ignoring context and pragmatism — readable code is the goal, not rigid rules without exception  
- Confusing short code with clean code — sometimes more lines, well-named, are clearer than one dense, "clever" line  
- Refactoring working code purely for aesthetics, with no tests covering the behavior (remember post 30?) to guarantee nothing broke  
- Applying patterns that are too complex for simple problems, making code harder to understand, not easier  

---

# 📌 Conclusion

- Code is read far more than it's written — optimize for whoever reads it later  
- Descriptive names and small functions eliminate the need for many comments  
- Clean Code is the thread connecting SOLID, testing, architecture, and documentation  
- The ultimate goal is always readability and maintainability, not rules for their own sake  

👉 With Clean Code, you understand that every technique in this track points to the same place: code any developer can understand, six months later, without needing to ask the original author

---

# 🔥 Next Step

Now that you've consolidated clean code principles, the next level is:

👉 **C# Fundamentals: Refactoring Legacy Code**

Here you'll learn to apply all of this to old systems, without tests, that already exist and can't simply be rewritten from scratch.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
