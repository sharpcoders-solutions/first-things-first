# 🧠 C# Fundamentals: Enums and Bitwise Flags

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to write fast C# and actually measure performance  
- `struct` vs `class` and the impact each has on allocation  

You've probably already used `enum` to represent a status or a category. But there's a more advanced — and much more powerful — use that most developers never explore: combining multiple options into a single value, using bits.

👉 **Let's dig into enums, and what bitwise flags actually are**

---

# 💡 The basics: `enum` as a named set of values

```csharp
public enum OrderStatus
{
    Pending,    // 0
    Paid,       // 1
    Shipped,    // 2
    Delivered,  // 3
    Cancelled   // 4
}

var status = OrderStatus.Paid;
```

👉 **`enum` = a value type (actually an `int` in disguise) that gives names to a fixed set of options**

Under the hood, each member is just an integer — `Pending` is `0`, `Paid` is `1`, and so on. This already beats using loose strings (`"paid"`, `"Paid"`, `"PAID"` generating typo bugs), because the compiler validates the possible values.

---

# 🔢 Controlling the values explicitly

```csharp
public enum OrderStatus
{
    Pending = 1,
    Paid = 2,
    Shipped = 3,
    Delivered = 4,
    Cancelled = 99
}
```

👉 You can assign explicit values — useful when the enum is persisted in a database or serialized in an API, and you need to guarantee the numbers never change, even if the member order changes in the code

---

# 🚩 The problem: when one value isn't enough

```csharp
public enum Permission
{
    Read,
    Write,
    Delete,
    Admin
}

// How do you represent "Read AND Write" at the same time?
Permission permissions = ???
```

👉 A regular `enum` represents **a single option at a time**. When you need to combine several options simultaneously (a user who can read and write, but not delete), the traditional enum doesn't solve it on its own

---

# 🎛️ `[Flags]`: enums that combine with bitwise operators

```csharp
[Flags]
public enum Permission
{
    None    = 0,      // 0000
    Read    = 1 << 0, // 0001
    Write   = 1 << 1, // 0010
    Delete  = 1 << 2, // 0100
    Admin   = 1 << 3  // 1000
}
```

👉 Notice the `1 << 0`, `1 << 1`, `1 << 2`: each value occupies a **different bit**. That's no coincidence — it's what allows values to be combined without overlapping

```csharp
// Combining with OR (|)
Permission permissions = Permission.Read | Permission.Write;
// 0001 | 0010 = 0011 (Read AND Write, at the same time)

// Checking with HasFlag or AND (&)
bool canRead = permissions.HasFlag(Permission.Read);     // true
bool canDelete = permissions.HasFlag(Permission.Delete);  // false

// Removing a flag with AND + NOT (&~)
permissions = permissions & ~Permission.Write; // removes only "Write"
```

👉 The `[Flags]` attribute doesn't change the bitwise behavior itself (that already works on any enum based on powers of two) — it changes how the enum is **displayed**: `Console.WriteLine(permissions)` shows `"Read, Write"` instead of a raw number

---

# ⚡ Why powers of two?

```csharp
1 << 0 = 0001 = 1
1 << 1 = 0010 = 2
1 << 2 = 0100 = 4
1 << 3 = 1000 = 8
```

👉 Each bit is an independent option. Combining `1` and `2` with OR gives `3` (`0011`), which is an **unambiguous** combination — given the number `3`, you can always decompose it back into `Read + Write`, because no other pair of flags adds up to the same value

If you used `1, 2, 3, 4` (sequential, not powers of two), `Read | Write` (`1 | 2 = 3`) would be indistinguishable from the `Delete` value (`3`) on its own — the bitwise combination only works because each flag is an isolated bit

---

# 🔍 Checking multiple flags at once

```csharp
Permission required = Permission.Read | Permission.Write;

bool hasAll = (permissions & required) == required;
```

👉 This checks whether `permissions` contains **all** the flags in `required`, not just one — useful in authorization rules where an action requires multiple permissions simultaneously

---

# ⚠️ Common Mistakes

- Using `[Flags]` on an enum without power-of-two values, silently breaking the bitwise combination  
- Forgetting the `None = 0` member on a `[Flags]` enum, leaving the "empty" state without an explicit representation  
- Using a regular `enum` (without `[Flags]`) and trying to combine values with `|`, producing a number that matches no named member  
- Comparing `[Flags]` enums with `==` expecting to check a single flag, when `HasFlag` (or `&`) is what actually answers that question  

---

# 📌 Conclusion

- `enum` is a named `int` — great for representing a single option among several  
- `[Flags]` lets you combine multiple values into a single enum, using bits  
- `[Flags]` values must be powers of two (`1, 2, 4, 8...`) for the combination to work unambiguously  
- `HasFlag` (or `&`) checks whether a specific flag is present; `|` combines, `&~` removes  

👉 Well-modeled enums eliminate an entire class of "magic string" bugs — next up is another language feature that solves a similar problem: returning multiple values from a method without creating a class just for that

---

# 🔥 Next Step

Now that you've mastered enums and flags, the next level is:

👉 **C# Fundamentals: Tuples and ValueTuple**

Here you'll learn to return and group multiple related values without needing to create a class or a `record` for every case.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
