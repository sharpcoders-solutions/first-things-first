# 🧠 C# Fundamentals: Extension Methods and Custom LINQ

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- LINQ introduction (post 19) — `Where`, `Select`, `OrderBy`  
- How to persist real data with EF Core  

Every time you wrote `orders.Where(o => o.Amount > 100)`, you used an extension method without stopping to think how that's possible — `Where` isn't a method on the `List<T>` class. Time to understand the mechanism behind it, and create your own.

👉 **Let's learn Extension Methods and Custom LINQ**

---

# 💡 What is an Extension Method?

👉 **Extension Method = a static method that looks like an instance method, "added" to a type you can't (or shouldn't) modify**

```csharp
public static class StringExtensions
{
    public static bool IsValidTaxId(this string value)
    {
        var digits = value.Where(char.IsDigit).ToArray();
        return digits.Length == 11;
    }
}
```

```csharp
string taxId = "123.456.789-00";
bool valid = taxId.IsValidTaxId(); // looks like a string method, but isn't
```

👉 The `this` before the first parameter is what turns an ordinary static method into an extension method — the compiler lets you call it as if it were part of the `string` class itself, even though you can't edit the BCL (Base Class Library)

---

# 🏗️ How LINQ is built entirely on this

```csharp
// A simplification of how LINQ's own Where works
public static class MyLinq
{
    public static IEnumerable<T> MyWhere<T>(this IEnumerable<T> source, Func<T, bool> predicate)
    {
        foreach (var item in source)
        {
            if (predicate(item))
                yield return item;
        }
    }
}
```

```csharp
var expensiveOrders = orders.MyWhere(o => o.Amount > 100);
```

👉 `Where`, `Select`, `OrderBy` — everything you've used since post 19 is just a collection of extension methods on `IEnumerable<T>`, defined in `System.Linq`. There's no separate language magic; it's the exact same mechanism you just created

---

# 🎯 Writing your own LINQ operator

```csharp
public static class OrderExtensions
{
    public static IEnumerable<Order> Overdue(this IEnumerable<Order> orders)
    {
        return orders.Where(o => o.Status == "Pending" && o.DueDate < DateTime.Now);
    }

    public static decimal TotalAmount(this IEnumerable<Order> orders)
    {
        return orders.Sum(o => o.Amount);
    }
}
```

```csharp
var total = orders.Overdue().TotalAmount();
```

👉 Chaining `.Overdue().TotalAmount()` works exactly like `.Where().Select()` — because it's the same technique. Extension methods on `IEnumerable<T>` that return `IEnumerable<T>` remain chainable, forming expressive queries specific to your domain

---

# 🔍 Extension methods vs inheritance

```csharp
// ❌ Not possible: string is sealed, you can't inherit from it
public class MyString : string { } 

// ✅ An extension method solves this without inheritance
public static string ToTitleCase(this string value) =>
    CultureInfo.CurrentCulture.TextInfo.ToTitleCase(value.ToLower());
```

👉 Remember the inheritance and polymorphism post (21)? Many .NET types are `sealed` precisely to avoid fragile hierarchies — extension methods give you flexibility without requiring inheritance, and without altering the original type

---

# ⚠️ Common Mistakes

- Creating extension methods for types you control, when a normal instance method would be more direct and discoverable  
- Overloading namespaces with overly generic extension methods, polluting autocomplete for basic types like `string` and `int`  
- Forgetting that extension methods can't access `private` members of the extended type — they only see the public API  
- Naming an extension method the same as an existing instance method, creating confusion about which one gets called  

---

# 📌 Conclusion

- Extension methods are static methods with `this` on the first parameter, called as if they were instance methods  
- All of LINQ (`Where`, `Select`, `OrderBy`) is built entirely with this technique  
- Your own extension methods on `IEnumerable<T>` remain chainable, just like native LINQ  
- They solve the problem of "adding behavior" without inheritance, especially useful on `sealed` types  

👉 With Extension Methods, you understand that LINQ was never language magic — it's an API you can extend with the same technique, building your own fluent vocabulary over any collection

---

# 🔥 Next Step

Now that you can extend types without inheritance, the next level is:

👉 **C# Fundamentals: Authentication and Authorization with JWT**

Here you'll learn to secure your API, making sure only authenticated and authorized users can access each endpoint.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
