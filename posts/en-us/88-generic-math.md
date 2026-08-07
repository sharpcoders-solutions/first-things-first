# 🧠 C# Fundamentals: Generic Math

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- The difference between `float`, `double`, and `decimal`, and when to use each  
- Static abstract interface members (post 58), including the `INumber<T>` example  

In the static abstract interface members post, you got a glimpse of the `INumber<T>` interface. Now it's time to explore Generic Math in depth — the feature that solves a problem that has existed since C#'s very first version of generics.

👉 **Let's master Generic Math**

---

# 💡 The historical problem: generics didn't support operators

```csharp
// Before C# 11, this was impossible to write
public T Add<T>(T a, T b)
{
    return a + b; // ❌ the compiler doesn't know if T supports "+"
}
```

👉 Remember the static abstract interface members post? Before C# 11, you had to write a separate overload of every generic math method for `int`, `double`, `decimal`, `float`... — duplication that had existed in the language forever, with no elegant solution

---

# 🔢 `INumber<T>`: the interface that unifies every numeric type

```csharp
using System.Numerics;

public T Add<T>(T a, T b) where T : INumber<T> => a + b;

int intSum = Add(3, 4);           // 7
double doubleSum = Add(3.5, 4.2); // 7.7
decimal decimalSum = Add(3.5m, 4.2m); // 7.7
```

👉 `INumber<T>` is implemented by **every** primitive numeric type in .NET (`int`, `long`, `float`, `double`, `decimal`, and even types like `BigInteger`) — a single generic method now works with any of them, with zero duplication

---

# 🎯 Real generic math operations

```csharp
public T Average<T>(IEnumerable<T> values) where T : INumber<T>
{
    T sum = T.Zero;
    int count = 0;

    foreach (var value in values)
    {
        sum += value;
        count++;
    }

    return sum / T.CreateChecked(count);
}

var averagePrices = Average(new decimal[] { 10.5m, 20.3m, 15.8m });
var averageGrades = Average(new double[] { 8.5, 9.0, 7.5 });
```

👉 `T.Zero` and `T.CreateChecked` also come from `INumber<T>` — the "generic" zero value and a safe way to convert another numeric type (like `int count`) into the `T` type being used, without boxing (remember post 44?) and without risky manual casts

---

# 🔍 Other interfaces in the `System.Numerics` namespace

```csharp
// IComparisonOperators: <, >, <=, >=
public T Max<T>(T a, T b) where T : IComparisonOperators<T, T, bool> => a > b ? a : b;

// IAdditionOperators: just the addition operation, without requiring all of INumber<T>
public T AddOnly<T>(T a, T b) where T : IAdditionOperators<T, T, T> => a + b;

// IMinMaxValue: the type's minimum and maximum values
public T MaxValueOfType<T>() where T : IMinMaxValue<T> => T.MaxValue;
```

👉 `INumber<T>` is the "full" interface, but .NET offers more granular interfaces — if your method only needs comparison, or only addition, you can restrict the constraint to exactly what's needed, following the same interface segregation principle you've already seen in SOLID

---

# 📊 A real use case: a generic statistics library

```csharp
public static class GenericStatistics
{
    public static T Sum<T>(this IEnumerable<T> values) where T : INumber<T>
    {
        T total = T.Zero;
        foreach (var value in values) total += value;
        return total;
    }

    public static T Max<T>(this IEnumerable<T> values) where T : INumber<T>, IMinMaxValue<T>
    {
        T max = T.MinValue;
        foreach (var value in values)
            if (value > max) max = value;
        return max;
    }
}

var totalSales = monthlySales.Sum();       // works with decimal
var highestTemperature = temperatures.Max(); // works with double
```

👉 Remember the extension methods and LINQ post? This is exactly the same pattern — except now, combined with Generic Math, your extension methods work with **any** numeric type, instead of needing one version for `int`, another for `decimal`, another for `double`

---

# ⚠️ Common Mistakes

- Using Generic Math for simple problems that a single concrete overload would already solve, adding unnecessary complexity  
- Forgetting that `T.CreateChecked` can throw on conversions that would lose data — use `T.CreateSaturating` or `T.CreateTruncating` when that behavior isn't desired  
- Mixing different numeric types in the same generic collection without conversion, expecting generic math to resolve type incompatibilities on its own  
- Not constraining to the most specific interface needed (using full `INumber<T>` when `IAdditionOperators<T,T,T>` would already solve it)  

---

# 📌 Conclusion

- `INumber<T>` unifies every .NET numeric type under a single generic abstraction  
- Generic Math eliminates the historical duplication of math methods per numeric type  
- `T.Zero`, `T.CreateChecked`, and `T.MinValue`/`MaxValue` come from `System.Numerics` interfaces  
- More granular interfaces (`IAdditionOperators`, `IComparisonOperators`) allow more precise constraints  

👉 With Generic Math mastered, you close out the block on numeric types — next up is a completely different feature: advanced pattern matching, which makes complex conditional decisions far more expressive

---

# 🔥 Next Step

Now that you write real generic math, the next level is:

👉 **C# Fundamentals: Advanced Pattern Matching**

Here you'll learn comparison patterns (`and`, `or`, `not`), list pattern matching, and other evolutions of the feature you've used since the earliest posts on `switch`.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
