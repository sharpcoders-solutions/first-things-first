# 🧠 C# Fundamentals: Essential Design Patterns (Singleton, Factory, Strategy, and Repository)

⏱️ Reading time: 9 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- The five SOLID principles  
- How interfaces and polymorphism decouple code  

SOLID gives you the **principles**. Design patterns are the **ready-made solutions** the industry has already tested and approved for problems that repeat across virtually every system.

👉 **Let's look at four patterns you'll find in practically every professional C# project**

---

# 💡 What is a Design Pattern?

👉 **Design pattern = a reusable solution to a recurring software design problem**

It's not ready-made code to copy and paste — it's a **model**, a proven way to solve a category of problem, originally cataloged by the "Gang of Four" (GoF) book in the '90s.

---

# 🔒 Singleton

👉 **Guarantees a class has only one instance for the entire life of the program**

```csharp
class AppConfig
{
    private static readonly AppConfig _instance = new();

    public static AppConfig Instance => _instance;

    public string CurrentEnvironment { get; set; } = "Production";

    private AppConfig() { } // private constructor — no one creates it from outside
}
```

```csharp
AppConfig.Instance.CurrentEnvironment = "Staging";
Console.WriteLine(AppConfig.Instance.CurrentEnvironment); // Staging
```

## 🔹 How it works

- `private` constructor → prevents `new AppConfig()` from outside the class  
- `static readonly` field → guarantees only one instance exists, created once  
- Public static property → single access point  

## ⚠️ Use with caution

👉 Singleton is useful for global configuration, caching, or logging — but it's easy to abuse and turn it into **disguised global state**, making automated testing harder (you can't easily swap the instance for a mock)

**Practical rule:** in modern ASP.NET Core applications, prefer registering the dependency as a singleton in the dependency injection container instead of hand-coding the pattern — you get the same behavior without losing testability.

---

# 🏭 Factory

👉 **Centralizes object creation logic, hiding the details of "how" and "which" to instantiate**

Remember the `IDiscount` example from the SOLID post? Let's use a factory to decide which implementation to create:

```csharp
interface IDiscount
{
    decimal Apply(decimal amount);
}

class RegularDiscount : IDiscount
{
    public decimal Apply(decimal amount) => amount * 0.95m;
}

class VipDiscount : IDiscount
{
    public decimal Apply(decimal amount) => amount * 0.80m;
}

class DiscountFactory
{
    public static IDiscount Create(string customerType) => customerType switch
    {
        "VIP" => new VipDiscount(),
        _ => new RegularDiscount()
    };
}
```

```csharp
IDiscount discount = DiscountFactory.Create("VIP");
Console.WriteLine(discount.Apply(1000)); // 800
```

👉 Whoever calls `DiscountFactory.Create` doesn't need to know which classes exist or how they're built — it just asks for a result and gets back something that fulfills the `IDiscount` contract

This reinforces the **Open/Closed Principle**: new discount types become new classes plus one line in the factory, without touching whoever consumes `IDiscount`.

---

# 🎯 Strategy

👉 **Encapsulates interchangeable algorithms behind a common interface, letting you swap behavior at runtime**

Here's a reveal: you've **already seen** the Strategy pattern in the SOLID post, just without the name.

```csharp
interface IDiscount
{
    decimal Apply(decimal amount);
}

class DiscountCalculator
{
    private readonly IDiscount _strategy;

    public DiscountCalculator(IDiscount strategy)
    {
        _strategy = strategy;
    }

    public decimal Calculate(decimal amount) => _strategy.Apply(amount);
}
```

```csharp
var calculator = new DiscountCalculator(new VipDiscount());
Console.WriteLine(calculator.Calculate(1000)); // 800
```

👉 `DiscountCalculator` doesn't know which algorithm it's using — it just runs the **strategy** that was injected. Swapping behavior is just a matter of swapping which `IDiscount` implementation you pass in

**The difference between Factory and Strategy:** Factory solves **who to create**; Strategy solves **which behavior to use**. They often work together — like in the example above, where the factory decides which strategy to instantiate.

---

# 🗄️ Repository

👉 **Abstracts data access behind an interface, isolating the rest of the application from the details of how data is stored**

```csharp
interface IRepository<T>
{
    void Add(T item);
    T GetById(int id);
    IEnumerable<T> ListAll();
}
```

```csharp
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}

class InMemoryRepository<T> : IRepository<T> where T : class
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);
    public T GetById(int id) => _items.FirstOrDefault(); // simplified
    public IEnumerable<T> ListAll() => _items;
}
```

```csharp
IRepository<Product> repository = new InMemoryRepository<Product>();
repository.Add(new Product { Id = 1, Name = "Laptop" });

foreach (var product in repository.ListAll())
{
    Console.WriteLine(product.Name);
}
```

👉 If tomorrow you swap the in-memory list for Entity Framework, Dapper, or an external API, whoever consumes `IRepository<T>` **doesn't change a single line** — pure Dependency Inversion Principle applied to data access

This pattern uses exactly what you learned in [Generics](24-generics.md) (`IRepository<T>`) and in [DIP](28-solid-principles.md) (depending on the interface, not the concrete implementation).

---

# 🔗 How the four connect

| Pattern | Problem it solves | Related SOLID principle |
|---|---|---|
| **Singleton** | Guaranteeing a single shared instance | — (use with caution) |
| **Factory** | Centralizing and hiding creation logic | Open/Closed |
| **Strategy** | Swapping algorithms at runtime | Open/Closed, Dependency Inversion |
| **Repository** | Isolating the rest of the system from persistence details | Dependency Inversion |

👉 Notice that none of these patterns is new magic — they're all SOLID applied to a concrete, recurring problem

---

# ⚠️ Common Mistakes

- Using Singleton for everything, creating hidden global state and code that's hard to test  
- Creating a Factory for a single type, when a direct `new` would already solve it without complication  
- Confusing Strategy with Factory — Strategy swaps behavior, Factory decides which object to create  
- Implementing Repository just to check a "best practices" box, when the application doesn't actually need to swap the data source  

---

# 📌 Conclusion

- **Singleton** guarantees a single instance — powerful, but easy to abuse  
- **Factory** centralizes and hides object creation logic  
- **Strategy** lets you swap algorithms at runtime through a common interface  
- **Repository** isolates data access, applying Dependency Inversion in practice  

👉 Design patterns aren't rules — they're shared vocabulary. When a teammate says "this is a Strategy," you both now know exactly what they mean.

---

# 🔥 Next Step

Now that you know the patterns used most often in day-to-day work, the next level is:

👉 **C# Fundamentals: Unit Testing with xUnit**

Here you'll learn to guarantee, in an automated way, that all this well-designed code keeps working as the system grows.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
