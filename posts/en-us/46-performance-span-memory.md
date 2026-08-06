# 🧠 C# Fundamentals: Performance in C# (Span, Memory, and Benchmarking)

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- gRPC for efficient communication between services  
- Caching, resilience, and messaging to scale your application  

You've already optimized the architecture. Now let's go one level deeper: how to write C# code that uses less memory and less CPU time, at the lowest level of the language.

👉 **Let's talk about real performance, backed by measurement, not guesswork**

---

# 💡 The golden rule: measure before optimizing

👉 **"Premature optimization is the root of all evil" — optimizing without measuring is just guessing**

Before any performance change, you need data. That's where **BenchmarkDotNet** comes in:

```bash
dotnet add package BenchmarkDotNet
```

```csharp
[MemoryDiagnoser]
public class ConcatenationBenchmark
{
    [Benchmark]
    public string WithStringBuilder()
    {
        var sb = new StringBuilder();
        for (int i = 0; i < 1000; i++)
            sb.Append(i);
        return sb.ToString();
    }

    [Benchmark]
    public string WithConcatenation()
    {
        string result = "";
        for (int i = 0; i < 1000; i++)
            result += i;
        return result;
    }
}
```

```csharp
BenchmarkRunner.Run<ConcatenationBenchmark>();
```

👉 The result shows execution time **and** memory allocated per method — often the difference in memory allocation matters more than the difference in raw speed, because less allocation means less work for the Garbage Collector (remember it, from the .NET architecture post?)

---

# 🧵 The problem with unnecessary allocations

```csharp
string text = "John,25,New York";
string[] parts = text.Split(','); // allocates a new array + three new strings
```

👉 Every time you slice a string with `Split` or `Substring`, the CLR allocates **new memory** — in code that runs millions of times (a parser, log processing), this adds up fast and pressures the Garbage Collector

---

# 🔪 `Span<T>`: slicing without allocating

```csharp
ReadOnlySpan<char> text = "John,25,New York".AsSpan();

int firstComma = text.IndexOf(',');
ReadOnlySpan<char> name = text.Slice(0, firstComma);

Console.WriteLine(name.ToString()); // John
```

👉 **`Span<T>` = a "window" into an existing chunk of memory, without copying data**

`Slice` doesn't create a new string — it points to the same memory, just delimiting a smaller portion. This is different from `Substring`, which always allocates a new copy

## 🔹 Where `Span<T>` shines

- Text parsing (files, protocols, logs)  
- Manipulating arrays without creating intermediate copies  
- High-performance code, like serialization libraries  

👉 For everyday C# (controllers, business services), this optimization rarely matters — the real gain shows up in code running in tight loops, with high data volume

---

# 🧠 `Memory<T>`: `Span<T>` for asynchronous scenarios

```csharp
public async Task ProcessAsync(Memory<byte> data)
{
    await Task.Delay(100); // Span<T> can't cross an await; Memory<T> can
    Process(data.Span);
}
```

👉 `Span<T>` can only live on the stack (it's a `ref struct`), which means it **cannot** be used inside `async` methods (remember the async/await post? An async method's state can live on the heap between awaits). `Memory<T>` solves this, working as a version of `Span<T>` that can cross asynchronous operations

---

# 📊 `struct` vs `class`: the impact on allocation

```csharp
public struct PointStruct // value type — lives on the stack when possible
{
    public int X;
    public int Y;
}

public class PointClass // reference type — always allocated on the heap
{
    public int X;
    public int Y;
}
```

👉 Remember the difference between value and reference types from the variables post? This has a direct performance impact: small `struct`s, used at high volume (e.g., millions of coordinates), avoid Garbage Collector pressure by not needing heap allocation

**Practical rule:** use `struct` for small, immutable data used in large quantities. For most business entities, `class` remains the right choice.

---

# ⚙️ `StringBuilder`: avoiding repeated concatenation

```csharp
// ❌ Every += creates a new string, discarding the previous one
string result = "";
foreach (var item in items)
    result += item + ", ";

// ✅ StringBuilder reuses the same internal buffer
var sb = new StringBuilder();
foreach (var item in items)
    sb.Append(item).Append(", ");
string result = sb.ToString();
```

👉 Strings in C# are **immutable** (you saw this back in the variables post) — every concatenation with `+=` creates a brand new string. In loops with many iterations, this turns into `O(n²)` of wasted work

---

# ⚠️ Common Mistakes

- Optimizing code without measuring first with a benchmark, based purely on intuition  
- Using `Span<T>` in simple, everyday code where the extra complexity brings no noticeable gain  
- Concatenating strings with `+=` inside large loops instead of `StringBuilder`  
- Turning every business entity into a `struct` "for performance," without understanding it changes the value-vs-reference copy semantics  

---

# 📌 Conclusion

- Measure with BenchmarkDotNet before optimizing — data beats intuition  
- `Span<T>` avoids unnecessary allocations when slicing existing memory  
- `Memory<T>` is the `Span<T>` version compatible with `async` code  
- `struct` reduces Garbage Collector pressure for small, numerous data  
- `StringBuilder` avoids the cost of repeated concatenation on immutable strings  

👉 Performance in C# isn't about obscure tricks — it's about understanding how the language allocates memory, and measuring before acting

---

# 🔥 Next Step

Now that you know how to optimize at the lowest level, the next level is:

👉 **C# Fundamentals: Advanced API Security (OWASP Top 10 in Practice)**

Here you'll learn to protect your application against the most common, most exploited real-world vulnerabilities.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
