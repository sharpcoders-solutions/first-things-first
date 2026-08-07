# 🧠 C# Fundamentals: SOLID Principles (Introduction to Software Design)

⏱️ Reading time: 13 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Classes, inheritance, polymorphism, and interfaces  
- Generics, delegates, and the language's modern features  

You already know how to write C# that **works**. From here on, the question changes: how do you write C# that's still easy to maintain a year from now, with ten other people touching the same codebase?

👉 **That's exactly what SOLID solves — and that's why it's, without a doubt, one of the most important topics in this whole track**

Let's take this slow, example by example. This post deserves a careful read.

---

# 💡 What is SOLID?

👉 **SOLID = five object-oriented design principles, created to make software easier to understand, extend, and maintain**

The term was popularized by Robert C. Martin ("Uncle Bob"), and each letter stands for a principle:

- **S** — Single Responsibility Principle  
- **O** — Open/Closed Principle  
- **L** — Liskov Substitution Principle  
- **I** — Interface Segregation Principle  
- **D** — Dependency Inversion Principle  

👉 None of these principles is a rigid rule carved in stone — they're **heuristics**. The end goal is always the same: reduce coupling and increase cohesion.

---

# 🔤 S — Single Responsibility Principle

👉 **A class should have one, and only one, reason to change**

## ❌ Before

```csharp
class Order
{
    public List<string> Items { get; set; } = new();

    public decimal CalculateTotal()
    {
        // calculation logic
        return Items.Count * 100;
    }

    public void SaveToDatabase()
    {
        // data access logic
        Console.WriteLine("Saving order to the database...");
    }

    public void SendConfirmationEmail()
    {
        // email sending logic
        Console.WriteLine("Sending confirmation email...");
    }
}
```

👉 This class changes if the calculation rule changes, if the database changes, or if the email provider changes. **Three different reasons to change inside the same class**

## ✅ After

```csharp
class Order
{
    public List<string> Items { get; set; } = new();

    public decimal CalculateTotal() => Items.Count * 100;
}

class OrderRepository
{
    public void Save(Order order) => Console.WriteLine("Saving order to the database...");
}

class EmailNotifier
{
    public void SendConfirmation(Order order) => Console.WriteLine("Sending confirmation email...");
}
```

👉 Now each class has **a single reason to change**: `Order` changes if the business rule changes; `OrderRepository` changes if the persistence method changes; `EmailNotifier` changes if the notification method changes

**Warning sign:** if you describe what a class does using the word "and" ("calculates the total **and** saves it **and** sends an email"), it's probably violating SRP.

---

# 🔓 O — Open/Closed Principle

👉 **Classes should be open for extension, but closed for modification**

## ❌ Before

```csharp
class DiscountCalculator
{
    public decimal Calculate(string customerType, decimal amount)
    {
        if (customerType == "Regular")
            return amount * 0.95m;
        else if (customerType == "Premium")
            return amount * 0.90m;
        else if (customerType == "VIP")
            return amount * 0.80m;

        return amount;
    }
}
```

👉 Every new customer type requires **opening** this class and adding another `else if`. That's fragile — a typo in an existing `case` can break a rule that already worked

## ✅ After

```csharp
interface IDiscount
{
    decimal Apply(decimal amount);
}

class RegularDiscount : IDiscount
{
    public decimal Apply(decimal amount) => amount * 0.95m;
}

class PremiumDiscount : IDiscount
{
    public decimal Apply(decimal amount) => amount * 0.90m;
}

class VipDiscount : IDiscount
{
    public decimal Apply(decimal amount) => amount * 0.80m;
}

class DiscountCalculator
{
    public decimal Calculate(IDiscount discount, decimal amount) => discount.Apply(amount);
}
```

```csharp
var calculator = new DiscountCalculator();
Console.WriteLine(calculator.Calculate(new VipDiscount(), 1000)); // 800
```

👉 A new customer type becomes a **new class** implementing `IDiscount` — `DiscountCalculator` never needs to be touched again. This is `interface` and polymorphism (posts 24 and 22) working together with software design

---

# 🔄 L — Liskov Substitution Principle

👉 **An object of a child class should be able to replace an object of the parent class without breaking the expected behavior**

This is the subtlest of the five — and the classic example proves why.

## ❌ Before

```csharp
class Rectangle
{
    public virtual double Width { get; set; }
    public virtual double Height { get; set; }

    public double CalculateArea() => Width * Height;
}

class Square : Rectangle
{
    public override double Width
    {
        get => base.Width;
        set { base.Width = value; base.Height = value; } // unexpected side effect
    }
}
```

```csharp
void TestArea(Rectangle rectangle)
{
    rectangle.Width = 5;
    rectangle.Height = 10;

    Console.WriteLine(rectangle.CalculateArea()); // expected: 50
}

TestArea(new Square()); // actual result: 100 — broke the expectation!
```

👉 `Square` **is a** `Rectangle` in mathematical theory, but in practice the code breaks the contract: changing `Width` also unexpectedly changes `Height`. Whoever uses `Rectangle` can no longer trust its behavior

## ✅ After

```csharp
abstract class Shape
{
    public abstract double CalculateArea();
}

class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public override double CalculateArea() => Width * Height;
}

class Square : Shape
{
    public double Side { get; set; }

    public override double CalculateArea() => Side * Side;
}
```

👉 Instead of forcing an inheritance relationship that doesn't hold up, each shape implements its own contract through `Shape` (you already saw this pattern in the inheritance and polymorphism post)

**Practical rule:** if overriding a method in the child class requires "weakening" or changing the parent class's expected behavior, inheritance is probably the wrong model.

---

# 🧩 I — Interface Segregation Principle

👉 **No class should be forced to implement methods it doesn't use**

## ❌ Before

```csharp
interface IEmployee
{
    void Work();
    void CalculateVacation();
    void ReceiveBonus();
}

class Intern : IEmployee
{
    public void Work() => Console.WriteLine("Intern working");
    public void CalculateVacation() => throw new NotImplementedException(); // doesn't apply
    public void ReceiveBonus() => throw new NotImplementedException();     // doesn't apply
}
```

👉 `Intern` is forced to "implement" methods that don't make sense for it — the contract is too big for someone who only consumes part of it

## ✅ After

```csharp
interface IWorker
{
    void Work();
}

interface IVacationEligible
{
    void CalculateVacation();
}

interface IBonusEligible
{
    void ReceiveBonus();
}

class Intern : IWorker
{
    public void Work() => Console.WriteLine("Intern working");
}

class Manager : IWorker, IVacationEligible, IBonusEligible
{
    public void Work() => Console.WriteLine("Manager working");
    public void CalculateVacation() => Console.WriteLine("Calculating vacation");
    public void ReceiveBonus() => Console.WriteLine("Receiving bonus");
}
```

👉 Each class implements only the contracts that make sense for it — small, specific interfaces, exactly like you saw in the interfaces post

---

# 🔌 D — Dependency Inversion Principle

👉 **High-level modules shouldn't depend on low-level modules — both should depend on abstractions**

## ❌ Before

```csharp
class EmailService
{
    public void Send(string message) => Console.WriteLine($"Email: {message}");
}

class OrderProcessor
{
    private readonly EmailService _emailService = new(); // concrete dependency, created right here

    public void Process()
    {
        _emailService.Send("Order processed!");
    }
}
```

👉 `OrderProcessor` is "tied" to `EmailService`. If tomorrow you want to notify via SMS instead of email, you have to **modify** `OrderProcessor`

## ✅ After

```csharp
interface INotifier
{
    void Send(string message);
}

class EmailService : INotifier
{
    public void Send(string message) => Console.WriteLine($"Email: {message}");
}

class SmsService : INotifier
{
    public void Send(string message) => Console.WriteLine($"SMS: {message}");
}

class OrderProcessor
{
    private readonly INotifier _notifier;

    public OrderProcessor(INotifier notifier) // dependency injected from outside
    {
        _notifier = notifier;
    }

    public void Process()
    {
        _notifier.Send("Order processed!");
    }
}
```

```csharp
var processor = new OrderProcessor(new SmsService());
processor.Process(); // SMS: Order processed!
```

👉 `OrderProcessor` only depends on the **abstraction** `INotifier` — swapping `EmailService` for `SmsService` doesn't require touching a single line of the processor

This is exactly the mechanism behind **dependency injection**, one of the pillars of any modern ASP.NET Core application: instead of a class creating its own dependencies, they arrive ready-made from outside — usually via the constructor, just like we saw here.

---

# 📋 Quick summary

| Letter | Principle | In one sentence |
|---|---|---|
| **S** | Single Responsibility | One class, one reason to change |
| **O** | Open/Closed | Extend with new classes, don't edit existing ones |
| **L** | Liskov Substitution | The child class can't break the parent class's contract |
| **I** | Interface Segregation | Small, specific interfaces, not one giant one |
| **D** | Dependency Inversion | Depend on abstractions, not concrete implementations |

---

# ⚠️ Common Mistakes

- Applying SOLID dogmatically on small projects, creating too much abstraction for a simple problem  
- Confusing Dependency Inversion with "using a DI container" — the principle is about depending on abstractions; the container is just a tool that makes it easier  
- Thinking SRP means "a class should have a single method" — the principle is about **reason to change**, not amount of code  
- Ignoring SOLID entirely and only noticing the cost months later, when any small change breaks three other parts of the system  

---

# 📌 Conclusion

- **SRP**: every class should have a single reason to change  
- **OCP**: extend behavior with new classes, without modifying what already works  
- **LSP**: a child class should never break the expectations of whoever uses the parent class  
- **ISP**: prefer several small interfaces over one giant interface  
- **DIP**: depend on abstractions (interfaces), not concrete implementations  

👉 SOLID isn't about memorizing acronyms — it's about a single goal: code that's easy to extend without fear of breaking what already works. Once you truly understand this, you never write C# the same way again.

---

# 🔥 Next Step

Now that you've mastered the most important design principles of your career, the next level is:

👉 **C# Fundamentals: Essential Design Patterns (Singleton, Factory, Strategy, and Repository)**

Here you'll see how SOLID materializes into design patterns used every day in the industry.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
