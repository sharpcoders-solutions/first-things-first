# 🧠 C# Fundamentals: Advanced Pattern Matching

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Generic Math and how to unify operations across any numeric type  
- `switch` expressions and basic pattern matching, since the earliest posts on control flow  

You've been using `is` and `switch` with simple patterns for many posts now. C# has been expanding pattern matching with every version, and today it covers cases far richer than "is this object of that type?". Time to see the full set.

👉 **Let's explore advanced pattern matching**

---

# 💡 A quick recap: where we came from

```csharp
// Basic pattern matching, which you already use
if (obj is Product product && product.Price > 0)
{
    Console.WriteLine(product.Name);
}
```

👉 This combines a type test with a variable capture — the starting point every more advanced pattern builds on

---

# 🔗 Relational patterns: direct comparisons in `switch`

```csharp
string ClassifyAge(int age) => age switch
{
    < 0 => "Invalid",
    < 13 => "Child",
    < 18 => "Teenager",
    < 65 => "Adult",
    _ => "Senior"
};
```

👉 Relational patterns (`<`, `>`, `<=`, `>=`) inside a `switch` eliminate chains of `if/else if` — each `case` is already a direct comparison, evaluated in order

---

# 🔀 Combining patterns with `and`, `or`, `not`

```csharp
string EvaluateGrade(int grade) => grade switch
{
    >= 90 and <= 100 => "Excellent",
    >= 70 and < 90 => "Good",
    >= 50 and < 70 => "Fair",
    not (>= 0 and <= 100) => "Invalid grade",
    _ => "Failing"
};
```

👉 `and`, `or`, and `not` combine patterns the same way logical operators combine boolean expressions — `>= 90 and <= 100` is far more direct than writing `grade >= 90 && grade <= 100` outside a `switch`

---

# 🧩 Property patterns: inspecting an object's fields

```csharp
public record Order(decimal Amount, string Status, bool VipCustomer);

string CalculateDiscount(Order order) => order switch
{
    { VipCustomer: true, Amount: > 1000 } => "20% discount",
    { VipCustomer: true } => "10% discount",
    { Amount: > 500 } => "5% discount",
    _ => "No discount"
};
```

👉 **Property pattern = testing values of specific object properties, directly inside the pattern, without manually accessing `order.VipCustomer` and `order.Amount`**

Remember `record`s (post 30)? Property patterns pair perfectly with them — records are often used exactly to model the data these patterns inspect

---

# 📋 List patterns: inspecting arrays and collections

```csharp
int[] numbers = { 1, 2, 3 };

string DescribeSequence(int[] values) => values switch
{
    [] => "Empty",
    [var only] => $"One element: {only}",
    [var first, var second] => $"Two elements: {first} and {second}",
    [var first, .., var last] => $"Starts with {first}, ends with {last}",
};

Console.WriteLine(DescribeSequence(numbers)); // Starts with 1, ends with 3
```

👉 **List pattern = testing the shape (count and position of elements) of an array or list directly in the `switch`**

The `..` (slice pattern) captures "the rest" of the sequence, letting you test just the start, just the end, or both, without writing manual index logic

---

# 🎯 Tuple patterns: multiple values at once

```csharp
(int X, int Y) point = (0, 5);

string DescribePosition(int x, int y) => (x, y) switch
{
    (0, 0) => "Origin",
    (0, _) => "On the Y axis",
    (_, 0) => "On the X axis",
    (var px, var py) when px == py => "On the diagonal",
    _ => "Any position"
};
```

👉 Remember tuples (post 51)? Combined with `switch`, they let you test multiple related values in a single pattern — including `when` clauses for conditions that don't fit a simple structural pattern

---

# 🌳 Nested patterns: combining everything

```csharp
public record Address(string City, string State);
public record Customer(string Name, Address Address, bool Active);

string ClassifyCustomer(Customer customer) => customer switch
{
    { Active: true, Address: { State: "NY" } } => "Active customer in New York",
    { Active: true, Address: { City: var city } } => $"Active customer in {city}",
    { Active: false } => "Inactive customer",
};
```

👉 Property patterns can be nested indefinitely — inspecting properties of properties, combined with relational, list, and tuple patterns, all inside a single expressive, readable `switch`

---

# ⚠️ Common Mistakes

- Stacking so many nested patterns that the `switch` becomes harder to read than the `if/else` chain it was meant to replace  
- Forgetting the `_` (default) case in a `switch expression`, causing a `MatchFailureException` at runtime for uncovered values  
- Using `and`/`or` without parentheses in complex combinations, creating ambiguity about evaluation order  
- Duplicating significant business logic inside a pattern, when it should live in a named, separately testable method  

---

# 📌 Conclusion

- Relational patterns (`<`, `>`) eliminate `if/else if` chains inside a `switch`  
- `and`, `or`, `not` combine patterns with the same clarity as logical operators  
- Property patterns inspect object fields directly, without manual access  
- List patterns test the shape of arrays and collections, including slices with `..`  
- Patterns can be freely nested, combining all the previous types  

👉 With advanced pattern matching mastered, the next step is seeing how C# guarantees, at compile time, that required properties are always initialized

---

# 🔥 Next Step

Now that you've mastered pattern matching in depth, the next level is:

👉 **C# Fundamentals: Required Members and Init-Only Properties**

Here you'll learn to enforce, at compile time, that certain properties are always filled in when an object is created.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
