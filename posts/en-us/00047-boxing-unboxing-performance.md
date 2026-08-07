# 🧠 C# Fundamentals: Boxing, Unboxing and Type Performance

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How value types and reference types behave differently  
- Where each one lives in memory: stack vs heap  

You know `int` is a value type. But have you ever stopped to think about what happens when you store an `int` in an `ArrayList`, or pass it as `object` to a method? There's a hidden cost in that conversion, and it shows up exactly where performance tends to matter.

👉 **Let's understand boxing, unboxing, and how to avoid their cost**

---

# 💡 What is boxing?

```csharp
int number = 42;
object box = number; // BOXING: the value 42 is copied onto the heap
```

👉 **Boxing = wrapping a value type inside an object on the heap, so it can be treated as `object`**

When this happens, the runtime allocates a new object on the heap, copies the `int`'s value into it, and the `box` variable holds a reference to that "package." The original `int` on the stack still exists, untouched — but now there's an **entirely separate copy** on the heap.

---

# 📦 Unboxing: the way back

```csharp
object box = 42;
int number = (int)box; // UNBOXING: copies the value back to the stack
```

👉 **Unboxing = extracting the value type back out of the object, with a runtime type check**

If the type inside the box doesn't exactly match the expected type, unboxing throws `InvalidCastException`:

```csharp
object box = 42;
long number = (long)box; // ❌ InvalidCastException — the box holds an int, not a long
```

---

# 💸 Why this is expensive

```csharp
// Scenario with implicit boxing
var list = new System.Collections.ArrayList();
for (int i = 0; i < 1_000_000; i++)
{
    list.Add(i); // boxing on EVERY iteration
}
```

👉 `ArrayList` stores `object`, so every `int` added gets boxed — a million heap allocations, one per number, plus the extra work the garbage collector has to do to free all of it later

```csharp
// Scenario without boxing
var list = new List<int>();
for (int i = 0; i < 1_000_000; i++)
{
    list.Add(i); // no boxing — List<int> stores int directly
}
```

👉 `List<T>` is generic: when `T` is `int`, it stores actual `int`s, without converting to `object`. This is one of the reasons generic collections (`List<T>`, `Dictionary<TKey, TValue>`) completely replaced the non-generic collections (`ArrayList`, `Hashtable`) from .NET's early days

---

# 🕵️ Where boxing hides

```csharp
// String interpolation with value types — no boxing in modern C#,
// but string.Format can still cause boxing depending on the overload used
Console.WriteLine("Value: {0}", 42); // may cause boxing depending on the signature used

// Passing a struct into an object parameter
void Log(object value) { }
Log(42); // boxing — 42 becomes an object

// Storing value types in old non-generic collections
Hashtable table = new Hashtable();
table["key"] = 42; // boxing
```

👉 The common pattern: **whenever a value type needs to be treated as `object` (or as an interface it implements), boxing happens**. Even implementing an interface can trigger boxing:

```csharp
interface IIdentifiable { int Id { get; } }
struct Item : IIdentifiable { public int Id { get; set; } }

void Process(IIdentifiable item) { } // receives it through the interface

var item = new Item { Id = 1 };
Process(item); // boxing — struct converted to the interface
```

---

# ✅ How to avoid unnecessary boxing

- **Prefer generic collections** (`List<T>`, `Dictionary<TKey, TValue>`) over non-generic ones (`ArrayList`, `Hashtable`)  
- **Avoid `object` parameters for value types** when a generic (`T`) solves the same problem without conversion  
- **Watch out for LINQ over non-generic `IEnumerable`** — iterating old collections with LINQ can silently introduce boxing  
- **In genuinely performance-sensitive code** (tight loops, hot paths), measure with `BenchmarkDotNet` before assuming boxing is the problem — premature optimization has its own cost  

```csharp
// Generic avoids boxing entirely
void Process<T>(T value) where T : IIdentifiable { }
```

👉 Using a generic method with an interface constraint, instead of receiving the interface directly, is how you get polymorphism without paying the boxing cost — the compiler generates a specialized version of the method for each value type used

---

# ⚠️ Common Mistakes

- Using `ArrayList` or `Hashtable` in new code, without realizing the implicit boxing on every insertion  
- Assuming every `object value` in a method signature is "just a type detail," ignoring the allocation cost  
- Optimizing boxing in code that runs once per request, where the cost is irrelevant next to a network or database call  
- Unboxing to the wrong type and getting surprised by an `InvalidCastException` in production  

---

# 📌 Conclusion

- Boxing wraps a value type inside an object on the heap; unboxing does the reverse  
- Every boxing operation is a heap allocation — expensive when it happens in loops or high-frequency code  
- Generic collections (`List<T>`) avoid boxing that non-generic collections (`ArrayList`) always pay  
- Implementing an interface on a struct and passing it through the interface also triggers boxing  
- Measure before optimizing: boxing only really matters on genuine hot paths  

👉 With value types, reference types, and now boxing/unboxing mastered, you have the full foundation to understand why C# behaves the way it does at runtime

---

# 🔥 Next Step

Now that you understand the real cost of conversions between value and reference types, the next level is:

👉 **C# Fundamentals: gRPC — Efficient Communication Between Services**

Here you'll learn a faster, strongly-typed alternative to REST for communication between services.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
