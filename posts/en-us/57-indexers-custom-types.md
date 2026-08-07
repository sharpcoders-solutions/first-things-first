# 🧠 C# Fundamentals: Indexers on Custom Types

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to overload operators like `+`, `==`, and `<` on your types  
- `IComparable<T>` for enabling custom ordering  

You've used `list[0]` and `dictionary["key"]` since the first few weeks of this track. But can you get that same bracket syntax on a class of your own? You can — and the feature that does it is called an indexer.

👉 **Let's learn to build custom indexers**

---

# 💡 What is an indexer?

```csharp
public class Week
{
    private readonly string[] _days =
        { "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday" };

    public string this[int index] => _days[index];
}

var week = new Week();
Console.WriteLine(week[0]); // Sunday
Console.WriteLine(week[3]); // Wednesday
```

👉 **Indexer = a special member, declared with `this[...]`, that lets you access an object using bracket syntax, as if it were an array**

Under the hood, `week[0]` gets converted by the compiler into a method call — but to whoever reads the code, it looks like direct array access

---

# ✏️ Indexer with get and set

```csharp
public class Inventory
{
    private readonly Dictionary<string, int> _stock = new();

    public int this[string product]
    {
        get => _stock.TryGetValue(product, out var quantity) ? quantity : 0;
        set => _stock[product] = value;
    }
}

var inventory = new Inventory();
inventory["Laptop"] = 15;              // uses the "set"
int quantity = inventory["Laptop"];    // uses the "get" — 15

int outOfStock = inventory["Mouse"]; // 0 — no exception, returns default
```

👉 Just like a property, an indexer can have separate `get` and `set`. Here, the `get` handles missing keys by returning `0` instead of throwing — a design decision entirely under your control

---

# 🔢 Indexers with multiple parameters

```csharp
public class Matrix
{
    private readonly int[,] _data;

    public Matrix(int rows, int columns)
    {
        _data = new int[rows, columns];
    }

    public int this[int row, int column]
    {
        get => _data[row, column];
        set => _data[row, column] = value;
    }
}

var matrix = new Matrix(3, 3);
matrix[0, 0] = 1;
matrix[1, 1] = 5;
Console.WriteLine(matrix[1, 1]); // 5
```

👉 Unlike regular arrays, indexers accept **any number and type** of parameters — including combinations like `this[string key, DateTime date]`, something a native C# array could never allow

---

# 🎯 Overloading indexers

```csharp
public class Cache
{
    private readonly Dictionary<int, string> _byId = new();
    private readonly Dictionary<string, string> _byKey = new();

    public string this[int id] => _byId.TryGetValue(id, out var value) ? value : null;
    public string this[string key] => _byKey.TryGetValue(key, out var value) ? value : null;
}

var cache = new Cache();
var byId = cache[42];       // uses this[int]
var byKey = cache["active"]; // uses this[string]
```

👉 Just like regular methods, indexers can be overloaded — the compiler picks the right version based on the type of the argument passed between the brackets

---

# 🛡️ Validation inside the indexer

```csharp
public class Week
{
    private readonly string[] _days =
        { "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday" };

    public string this[int index]
    {
        get
        {
            if (index < 0 || index >= _days.Length)
                throw new ArgumentOutOfRangeException(nameof(index), "Index must be between 0 and 6");

            return _days[index];
        }
    }
}
```

👉 Unlike a plain array, an indexer is real code — you can validate, log, or apply any business rule before returning the value, exactly as you would in any other method

---

# ⚠️ Common Mistakes

- Creating indexers for types with no natural notion of "access by position or key," making the API confusing instead of intuitive  
- Not validating the received index, letting generic exceptions from the internal array leak out instead of a clear message  
- Forgetting that indexers can have independent `get` and `set` — a read-only indexer is perfectly valid  
- Duplicating logic between an indexer and equivalent methods (`GetById`, `SetById`) instead of centralizing it in one place  

---

# 📌 Conclusion

- Indexers enable `object[key]` syntax on custom types, via `this[...]`  
- They can have separate `get` and `set`, just like regular properties  
- They accept multiple parameters of any type, going beyond the limitations of native arrays  
- They can be overloaded, allowing multiple ways to access the same object  

👉 Indexers make your types more expressive and natural to use — the last advanced feature in this stretch of posts goes further, letting an **entire interface** require operators and static members from the types that implement it

---

# 🔥 Next Step

Now that you know how to build custom indexers, the next level is:

👉 **C# Fundamentals: Static Abstract Interface Members**

Here you'll learn one of C#'s most recent features, which lets interfaces require operators and static members from the classes that implement them.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
