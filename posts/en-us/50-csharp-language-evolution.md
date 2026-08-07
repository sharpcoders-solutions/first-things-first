# 🧠 C# Fundamentals: The Evolution of the C# Language

⏱️ Reading time: 9 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Anonymous types, and why `dynamic` should be avoided in most cases  
- The entire foundation of the language, from `Console.WriteLine` to `Span<T>`, passing through enums, tuples, and anonymous types  

You've used dozens of features throughout this track — `record`, pattern matching, `async`/`await`, nullable reference types — without stopping to think where each one came from. C# wasn't born this way. These features were added, version by version, over more than twenty years.

👉 **Let's take a tour through C#'s evolution, and understand why the language ended up where it is**

---

# 💡 Why this matters in practice

👉 **Knowing where a feature came from helps you understand its purpose — and recognize legacy code that hasn't yet adopted the modern way to solve the same problem**

In technical interviews and code review, it's common to run into code written in the style of an old C# version, even in projects already running on a recent .NET version. Understanding this timeline helps you modernize legacy code with confidence.

---

# 🏗️ C# 1.0 to 2.0: the fundamentals and generics

```csharp
// C# 1.0 (2002): no generics — ArrayList stores "object", requiring boxing
ArrayList list = new ArrayList();
list.Add(42); // boxing

// C# 2.0 (2005): generics — the milestone that changed everything
List<int> genericList = new List<int>();
genericList.Add(42); // no boxing
```

👉 Remember the boxing and unboxing post? The pain that motivated generics in C# 2.0 is exactly that: before them, every collection stored `object`, forcing boxing on every value. C# 2.0 also brought `nullable<T>` (`int?`), iterators with `yield return`, and anonymous methods — the foundation for everything that came after

---

# 🔧 C# 3.0: LINQ changes how you write code

```csharp
// Before LINQ: explicit loops to filter and transform
var expensive = new List<Product>();
foreach (var product in products)
{
    if (product.Price > 100)
        expensive.Add(product);
}

// C# 3.0 (2007): LINQ, lambda expressions, anonymous types, var
var expensive = products.Where(p => p.Price > 100).ToList();
```

👉 C# 3.0 is probably the version that changed the **style** of writing C# the most. LINQ, lambdas, `var`, expression trees, and anonymous types (from the previous post) all arrived together — and most of the "modern" C# code you've seen throughout this track uses features from this version

---

# ⚡ C# 4.0 to 5.0: dynamic and async/await

```csharp
// C# 4.0 (2010): dynamic, optional and named parameters
void Log(string message, string level = "Info") { }
Log(message: "Critical error", level: "Error");

// C# 5.0 (2012): async/await — the biggest leap in asynchronous programming
public async Task<Product> GetProductAsync(int id)
{
    return await _repository.GetByIdAsync(id);
}
```

👉 `async`/`await` (C# 5.0) deserves special mention: before it, asynchronous code in .NET meant nested callbacks, hard to read and hard to debug. That single syntax change is the reason all the async code you wrote throughout this track reads just like ordinary synchronous code

---

# 🎯 C# 6.0 to 7.0: quality of life and early pattern matching

```csharp
// C# 6.0 (2015): string interpolation, expression-bodied members
public string FullName => $"{FirstName} {LastName}";

// C# 7.0 (2017): tuples (from the previous post!), basic pattern matching, inline out
if (obj is Product product && product.Price > 0)
{
    Console.WriteLine(product.Name);
}
```

👉 The tuple you learned about in the previous post was born exactly in C# 7.0 — alongside `is` pattern matching, which evolved significantly in later versions into the robust pattern matching you already use today

---

# 🛡️ C# 8.0: nullable reference types and more pattern matching

```csharp
#nullable enable

public string? Nickname { get; set; } // can be null, and the compiler knows it
public string Name { get; set; } = ""; // cannot be null

// switch expressions
string category = price switch
{
    < 50 => "Cheap",
    < 200 => "Mid-range",
    _ => "Expensive"
};
```

👉 Nullable reference types (`string?` vs `string`) was one of the most impactful changes for reducing `NullReferenceException` in production — the compiler started warning you when you try to use a value that could be `null` without checking first

---

# 🚀 C# 9.0 to 12.0: `record`, static abstract members, and the language you use today

```csharp
// C# 9.0 (2020): record — the foundation of everything you've used in DTOs
public record Product(int Id, string Name, decimal Price);

// C# 10-11: required members, generic math, raw string literals
public class Order
{
    public required string Customer { get; init; }
}

// C# 12.0 (2023): primary constructors on regular classes
public class ProductService(IProductRepository repository)
{
    public Product GetById(int id) => repository.GetById(id);
}
```

👉 Each recent version adds fewer "revolutionary features" and more syntax **refinements** — `record`, `required`, primary constructors — all reducing the amount of repetitive boilerplate needed to express the same intent

---

# 📅 How C# evolves today: annual cycle, open on GitHub

👉 Since 2020, a new major C# version has shipped **every year**, alongside the corresponding .NET version, and the entire design process happens publicly in the `dotnet/csharplang` GitHub repository — any developer can follow (or even propose) discussions about the language's future

---

# ⚠️ Common Mistakes

- Writing pre-C# 3.0 style code (manual loops instead of LINQ) in a project already running on a recent .NET version  
- Ignoring nullable reference types in new projects, losing the compiler warnings that prevent `NullReferenceException`  
- Thinking "the newest language version" and "the newest .NET version" are the same thing — they're related but distinct concepts  
- Adopting very new features (preview features) in production code without evaluating their stability  

---

# 📌 Conclusion

- Generics (C# 2.0) eliminated the forced boxing of old collections  
- LINQ and lambdas (C# 3.0) changed the predominant style of writing C#  
- `async`/`await` (C# 5.0) made asynchronous code read like synchronous code  
- Nullable reference types (C# 8.0) and `record` (C# 9.0) directly reduced bugs and boilerplate  
- C# evolves on an annual cycle, with open design on GitHub  

👉 The entire language you've mastered throughout this track is the result of more than twenty years of incremental evolution — and that evolution continues with every new version

---

# 🔥 Next Step

Now that you understand the language's journey, the next level is:

👉 **C# Fundamentals: Feature Flags and Dynamic Configuration**

Here you'll learn to ship new features safely, without needing a new deployment for every change.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
