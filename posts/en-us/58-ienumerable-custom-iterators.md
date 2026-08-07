# 🧠 C# Fundamentals: IEnumerable and Custom Iterators

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How an API Gateway centralizes routing between services  
- Extension methods, and how LINQ is built under the hood  

Every `foreach` you've ever written depends on an interface you've probably never implemented directly: `IEnumerable<T>`. Let's open up that black box and see how to build your own custom sequences.

👉 **Let's dig into `IEnumerable<T>`, and the power of `yield return`**

---

# 💡 What `foreach` actually does

```csharp
foreach (var number in numbers)
{
    Console.WriteLine(number);
}

// The code above is syntactic sugar for this:
var enumerator = numbers.GetEnumerator();
while (enumerator.MoveNext())
{
    var number = enumerator.Current;
    Console.WriteLine(number);
}
```

👉 **`foreach` = a shorthand for calling `GetEnumerator()`, then repeatedly calling `MoveNext()`/`Current` until it's done**

Any type that implements `IEnumerable<T>` (that is, has a `GetEnumerator()` that returns an `IEnumerator<T>`) can be used in a `foreach` — that's the contract `List<T>`, `Dictionary<TKey, TValue>`, and all of LINQ (from the extension methods post) know how to consume

---

# 🏗️ Implementing `IEnumerable<T>` by hand

```csharp
public class Range3 : IEnumerable<int>
{
    private readonly int _start, _end;

    public Range3(int start, int end)
    {
        _start = start;
        _end = end;
    }

    public IEnumerator<int> GetEnumerator()
    {
        for (int i = _start; i <= _end; i++)
            yield return i;
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

foreach (var number in new Range3(1, 5))
    Console.WriteLine(number); // 1, 2, 3, 4, 5
```

👉 Notice that manually implementing an `IEnumerator<T>` (with `MoveNext`, `Current`, `Reset`) would be far more verbose — `yield return` makes the compiler generate that state machine for you automatically

---

# ✨ `yield return`: generating values on demand

```csharp
public IEnumerable<int> EvenNumbers(int limit)
{
    for (int i = 0; i <= limit; i += 2)
    {
        Console.WriteLine($"Generating {i}");
        yield return i;
    }
}

foreach (var even in EvenNumbers(6))
{
    Console.WriteLine($"Received {even}");
    if (even == 4) break;
}

// Output:
// Generating 0
// Received 0
// Generating 2
// Received 2
// Generating 4
// Received 4
```

👉 **`yield return` = pauses the method's execution, hands back a value, and resumes exactly where it left off on the next call**

Notice that "Generating 6" is **never** printed — `break` stops execution before the method continues. This is **lazy evaluation**: values are generated one at a time, on demand, not all at once into a pre-computed list

---

# 💾 Why this matters: memory and performance

```csharp
// ❌ Materializes ALL numbers in memory at once
public List<int> EvenNumbers(int limit)
{
    var list = new List<int>();
    for (int i = 0; i <= limit; i += 2)
        list.Add(i);
    return list;
}

// ✅ Generates one number at a time, never storing the whole sequence
public IEnumerable<int> EvenNumbers(int limit)
{
    for (int i = 0; i <= limit; i += 2)
        yield return i;
}
```

👉 For large (or even infinite) sequences, `yield return` avoids allocating an entire list in memory — each value is consumed and discarded before the next is generated, exactly like you saw in the streams post processing large files

---

# ♾️ Infinite sequences: only possible with lazy evaluation

```csharp
public IEnumerable<int> Fibonacci()
{
    int previous = 0, current = 1;

    while (true)
    {
        yield return previous;
        (previous, current) = (current, previous + current);
    }
}

var firstTen = Fibonacci().Take(10);
```

👉 This would be **impossible** with a method returning `List<int>` — the list would never finish being filled. With `yield return`, LINQ's `Take(10)` simply stops asking for new values after the tenth, and the method never generates the eleventh

---

# ⚠️ Common Mistakes

- Materializing a sequence with `.ToList()` too early, losing the lazy evaluation benefit `yield return` provides  
- Forgetting that a method with `yield return` only executes when you start **iterating** over it, not when you call it  
- Doing side effects (writing to a database, logging) inside a `yield return` iterator without realizing they only run as enumeration progresses  
- Trying to manually implement `IEnumerator<T>` when `yield return` would solve the same problem with far less code  

---

# 📌 Conclusion

- `foreach` is syntactic sugar for `GetEnumerator()` + `MoveNext()`/`Current`  
- `yield return` generates a state machine automatically, without manually implementing `IEnumerator<T>`  
- Iterators are lazy: values are generated on demand, one at a time  
- Lazy evaluation makes large (or infinite) sequences possible without exhausting memory  

👉 Custom iterators open the door to another feature that also redefines how a type behaves: custom operators, which make `+`, `==`, and other symbols work on your own types

---

# 🔥 Next Step

Now that you've mastered custom iterators, the next level is:

👉 **C# Fundamentals: Operator Overloading**

Here you'll learn to overload operators like `+`, `==`, and `<` so they make sense on your own types.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
