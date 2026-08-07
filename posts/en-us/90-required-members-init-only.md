# 🧠 C# Fundamentals: Required Members and Init-Only Properties

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Advanced pattern matching with property, list, and tuple patterns  
- `record`s and immutable properties, since the modern C# post  

You already use `record` and know it favors immutability. But what guarantees an object is created **correctly**, with all essential properties filled in, without relying on a giant constructor or manual team discipline? That's what `required` and `init` solve together.

👉 **Let's understand `required` members and `init`-only properties**

---

# 💡 The problem: optional properties that are actually required

```csharp
public class Product
{
    public string Name { get; set; } = default!;
    public decimal Price { get; set; }
}

var product = new Product(); // ✅ compiles, but Name is empty and Price is zero
```

👉 Using an object initializer (`new Product { Name = "...", Price = ... }`) is convenient, but nothing **forces** the creator to fill in `Name` — the compiler silently accepts `new Product()`, even though a nameless product makes no sense in your domain

---

# ✅ `required`: enforcing fields at compile time

```csharp
public class Product
{
    public required string Name { get; set; }
    public required decimal Price { get; set; }
    public string? Description { get; set; } // still optional
}

var product1 = new Product(); // ❌ compile error: Name and Price are required
var product2 = new Product { Name = "Laptop", Price = 3500m }; // ✅ ok
```

👉 **`required` = a property the compiler requires to be filled in at initialization, via an object initializer**

Unlike runtime validation (remember FluentValidation, from the API security post?), `required` catches the error **before the code even runs** — the whole class of "forgot to fill in a required field" bugs disappears entirely at development time

---

# 🔒 `init`: properties that can only be set at creation

```csharp
public class Product
{
    public required string Name { get; init; }
    public required decimal Price { get; init; }
}

var product = new Product { Name = "Laptop", Price = 3500m };
product.Name = "Another name"; // ❌ compile error: Name can only be set during initialization
```

👉 **`init` = like `set`, but can only be called during the object's creation (object initializer or constructor) — after that, the property becomes read-only**

Remember the difference between `record` and a regular mutable `class`? `init` is exactly the mechanism that gives `record`s (and any `class` you want) that post-construction immutability behavior

---

# 🎯 `required` + `init`: the most common combination

```csharp
public class ApiConfiguration
{
    public required string BaseUrl { get; init; }
    public required int TimeoutSeconds { get; init; }
    public int MaxRetries { get; init; } = 3; // optional, with a default value
}

var config = new ApiConfiguration
{
    BaseUrl = "https://api.example.com",
    TimeoutSeconds = 30
    // MaxRetries uses the default (3)
};
```

👉 This combination is the recommended pattern for DTOs, configuration options (remember the Options Pattern, post 76?), and any class where some properties are essential and others are genuinely optional — the compiler guarantees the required fields, `init` guarantees no one changes those values later

---

# 🏗️ `required` with constructors

```csharp
public class Product
{
    public required string Name { get; init; }

    [SetsRequiredMembers]
    public Product(string name)
    {
        Name = name;
    }
}

var product1 = new Product("Laptop");           // ✅ constructor covers the required
var product2 = new Product { Name = "Laptop" };  // ✅ object initializer also works
```

👉 If you'd rather expose a traditional constructor instead of forcing an object initializer, the `[SetsRequiredMembers]` attribute tells the compiler that specific constructor already guarantees all `required` members have been filled in

---

# ⚖️ `required`/`init` vs a `record`'s positional constructor

```csharp
// record: enforcement via position in the primary constructor
public record ProductRecord(string Name, decimal Price);

// class with required/init: enforcement via name, more flexible for many optional properties
public class ProductClass
{
    public required string Name { get; init; }
    public required decimal Price { get; init; }
    public string? Description { get; init; }
    public string? Category { get; init; }
}
```

👉 **Practical rule: use a `record`'s positional constructor when all (or almost all) properties are required and the order is natural. Use `required`/`init` on a `class` when you have a mix of many required and optional properties** — naming each one explicitly at creation reads more clearly than a constructor with ten positional parameters

---

# ⚠️ Common Mistakes

- Using `= default!` to "trick" the compiler instead of `required`, losing the real guarantee that a field gets filled in  
- Forgetting `init` on properties that should be immutable after creation, allowing accidental mutation later in the code  
- Applying `required` to every property on a class, even genuinely optional ones, forcing the creator to fill in unnecessary fields  
- Confusing `init` with `readonly` — `readonly` can only be set in the constructor; `init` can be set in any initializer, including object initializers  

---

# 📌 Conclusion

- `required` enforces, at compile time, that a property is filled in when the object is created  
- `init` lets you set a property only during creation, making it read-only afterward  
- The `required` + `init` combination is the recommended pattern for DTOs and configuration classes  
- `[SetsRequiredMembers]` lets you use traditional constructors even with `required` properties  

👉 With objects that self-validate at compile time, the next step is looking at another feature that's evolved a lot: regular expressions, and how to compile them for maximum performance

---

# 🔥 Next Step

Now that you guarantee correctly initialized objects at compile time, the next level is:

👉 **C# Fundamentals: Regex and GeneratedRegex**

Here you'll learn to use regular expressions in C#, and how the `GeneratedRegex` source generator eliminates the runtime compilation cost.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
