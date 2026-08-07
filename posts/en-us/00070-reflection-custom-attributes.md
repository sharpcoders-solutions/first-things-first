# 🧠 C# Fundamentals: Reflection and Custom Attributes

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Unsafe code, stepping outside C#'s managed layer  
- `[HttpGet]`, `[Authorize]`, `[Fact]` — attributes you've been using since the earliest ASP.NET Core and xUnit posts  

You've used dozens of attributes throughout this track, but never asked: how does ASP.NET Core know that `[HttpGet]` marks an endpoint? The answer is Reflection.

👉 **Let's learn Reflection and Custom Attributes**

---

# 💡 What is Reflection?

👉 **Reflection = C#'s ability to inspect types, methods, and properties at runtime, not just at compile time**

```csharp
Type type = typeof(Order);

Console.WriteLine(type.Name); // "Order"

foreach (var property in type.GetProperties())
{
    Console.WriteLine($"{property.Name}: {property.PropertyType}");
}
```

👉 Normally, you access `order.Id` directly in code. With Reflection, you discover that the `Id` property exists, what type it is, and you can even read/write the value — all without knowing the class ahead of time

---

# 🏗️ Creating a custom attribute

```csharp
[AttributeUsage(AttributeTargets.Property)]
public class RequiredForExportAttribute : Attribute
{
    public string Reason { get; }

    public RequiredForExportAttribute(string reason)
    {
        Reason = reason;
    }
}
```

```csharp
public class Order
{
    [RequiredForExport("Tax system requires this field")]
    public string CustomerTaxId { get; set; } = default!;

    public string Notes { get; set; } = default!;
}
```

👉 A custom attribute is nothing more than a class that inherits from `Attribute` — the same mechanism behind `[Fact]` (post 33) and `[Authorize]` (post 37)

---

# 🎯 Reading the attribute at runtime

```csharp
public static void ValidateRequiredFields<T>(T obj)
{
    var properties = typeof(T).GetProperties();

    foreach (var property in properties)
    {
        var attribute = property.GetCustomAttribute<RequiredForExportAttribute>();

        if (attribute != null)
        {
            var value = property.GetValue(obj);

            if (value is null || string.IsNullOrWhiteSpace(value.ToString()))
            {
                throw new InvalidOperationException(
                    $"{property.Name} is required: {attribute.Reason}");
            }
        }
    }
}
```

👉 `GetCustomAttribute` looks up the attribute on the property, and `GetValue`/`SetValue` read and write values dynamically — this is exactly how validation frameworks (FluentValidation) and serialization (System.Text.Json) work under the hood

---

# 🔍 How ASP.NET Core actually uses this

```csharp
[HttpGet("{id}")]
public IActionResult GetOrder(int id) => Ok(_service.Find(id));
```

👉 When the application starts, ASP.NET Core uses Reflection to scan every controller, find methods marked with `[HttpGet]`, `[HttpPost]`, etc., and automatically build the routing table — the same mechanism you just learned, running at scale inside the framework

---

# ⚠️ Common Mistakes

- Using Reflection in high-performance loops without caching — `GetProperties()` and `GetCustomAttribute()` are relatively slow compared to direct access  
- Ignoring `BindingFlags` when looking up private members, producing unexpected empty results  
- Using Reflection to bypass encapsulation (accessing `private` fields from outside), breaking the OOP principles from post 23  
- Not considering Source Generators (next post) as a faster alternative when the same task is known at compile time  

---

# 📌 Conclusion

- Reflection inspects and manipulates types at runtime  
- Custom attributes are classes that inherit from `Attribute`, read via Reflection  
- Frameworks like ASP.NET Core use this exact mechanism to discover routes, validations, and configurations  
- Reflection's performance cost needs to be considered in critical code  

👉 With Reflection, you understand how the "magic" behavior of the attributes you've used since the start of this track actually works under the hood

---

# 🔥 Next Step

Now that you understand Reflection, the next level is:

👉 **C# Fundamentals: Source Generators**

Here you'll learn a modern, faster alternative to Reflection, generating code at compile time.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
