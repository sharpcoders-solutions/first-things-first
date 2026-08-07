# 🧠 C# Fundamentals: Numeric Types and Precision (float, double, and decimal)

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- `DateOnly` and `TimeOnly` for representing date and time with precision  
- `decimal`, used without much explanation, in every money calculation since the earliest e-commerce posts  

You've probably heard (or even lived through) the story of a financial system broken by incorrect rounding. That almost always comes from choosing wrong between `float`, `double`, and `decimal` — three types that look interchangeable, but aren't.

👉 **Let's understand the real difference between them**

---

# 💡 The classic experiment

```csharp
double result = 0.1 + 0.2;
Console.WriteLine(result); // 0.30000000000000004
Console.WriteLine(result == 0.3); // false
```

👉 This isn't a C# bug — it's a consequence of how `float` and `double` represent numbers: in **binary floating point**, following the IEEE 754 standard. Some simple decimal numbers (like `0.1`) have no exact binary representation, the same way `1/3` has no exact decimal representation (`0.333...`)

---

# 🔬 `float` and `double`: approximate precision, fast

```csharp
float floatNumber = 3.14159265358979f;   // ~7 digits of precision, 4 bytes
double doubleNumber = 3.14159265358979;  // ~15-17 digits of precision, 8 bytes
```

👉 `float` and `double` are optimized for **performance and range**, not exact decimal precision. Both can represent astronomically large or tiny numbers, with an acceptable margin of error for most scientific and graphics calculations

```csharp
// Where float/double shine: physics, graphics, machine learning
float speed = CalculateSpeed(mass, force);
double distance = CalculateTrajectory(angle, initialSpeed);
```

---

# 💰 `decimal`: exact precision, slower

```csharp
decimal price = 19.99m; // the "m" suffix indicates a decimal literal
decimal total = price * 3;

Console.WriteLine(total); // 59.97 — exact, no surprises
```

👉 **`decimal` = a 128-bit type that represents numbers in **base 10**, not binary — specifically designed to eliminate the rounding errors `float`/`double` have with common decimal values**

The trade-off: `decimal` takes up more memory (16 bytes vs `double`'s 8) and is slower in arithmetic operations, because it isn't directly supported by the processor's floating-point unit — it's implemented in software

---

# ⚖️ Comparing all three side by side

```csharp
float f = 0.1f + 0.2f;
double d = 0.1 + 0.2;
decimal m = 0.1m + 0.2m;

Console.WriteLine(f); // 0.3 (but can drift in less visible decimal places)
Console.WriteLine(d); // 0.30000000000000004
Console.WriteLine(m); // 0.3 — exact
```

| | `float` | `double` | `decimal` |
|---|---|---|---|
| Size | 4 bytes | 8 bytes | 16 bytes |
| Precision | ~7 digits | ~15-17 digits | 28-29 digits, exact in base 10 |
| Speed | Fast | Fast | Slower |
| Ideal use | Graphics, games | Scientific calculations | Money, exact values |

---

# 🎯 The practical rule: money is always `decimal`

```csharp
public class OrderItem
{
    public decimal Price { get; set; }
    public int Quantity { get; set; }
    public decimal Subtotal => Price * Quantity;
}
```

👉 You've already used `decimal` for money in practically every post throughout this track that involved prices — now you understand why: any rounding error in a financial value, no matter how small, is unacceptable, and `decimal` is the only one of the three types designed to never introduce that kind of error in common decimal operations

---

# 🔍 `float`/`double` aren't "wrong" either — they're specialized

```csharp
// Physics and simulations: small imprecisions are acceptable and expected
double energy = 0.5 * mass * velocity * velocity;

// Screen coordinates in a game
float positionX = 152.375f;
```

👉 The mistake **isn't** using `float`/`double` — the mistake is using them for money. In domains where a tiny margin of error is irrelevant (screen positions, approximate physics calculations), `float`/`double` are the right choice for their performance and lower memory footprint

---

# ⚠️ Common Mistakes

- Using `double` to represent money, introducing rounding errors that accumulate across thousands of transactions  
- Comparing `float`/`double` values with `==` expecting exact equality, when small floating-point imprecisions make that unreliable  
- Using `decimal` in high-performance scientific calculations, paying an unnecessary speed cost where exact precision doesn't matter  
- Mixing `float` and `double` in the same expression without explicit conversions, causing silent precision loss  

---

# 📌 Conclusion

- `float`/`double` represent numbers in binary floating point — fast, but with inherent imprecision on decimal values  
- `decimal` represents numbers in base 10 — exact for common decimal values, but slower and larger  
- Money and any value requiring exact decimal precision should always use `decimal`  
- Scientific calculations, graphics, and physics should generally use `float`/`double`, for performance  

👉 Understanding why these three types exist sets up the next feature: how to write generic code that works with any of them, without duplicating logic

---

# 🔥 Next Step

Now that you choose the right numeric type for each scenario, the next level is:

👉 **C# Fundamentals: Generic Math**

Here you'll learn to write a single generic method that works with `int`, `double`, `decimal`, and any other numeric type, using the static abstract interface members you already know.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
