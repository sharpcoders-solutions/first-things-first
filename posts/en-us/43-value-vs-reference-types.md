# 🧠 C# Fundamentals: Value Types vs Reference Types in Depth

⏱️ Reading time: 9 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- CQRS separating commands from queries  
- The Repository pattern and dependency injection, organizing the application into layers  

You've already written dozens of `class`es and a few `record`s. But there's a fundamental question that's rarely answered in depth: where exactly do these objects live in memory, and why does it matter for your code's performance?

👉 **Let's truly understand the difference between value types and reference types**

---

# 💡 The core difference: copying value vs copying reference

```csharp
struct Point
{
    public int X;
    public int Y;
}

class Rectangle
{
    public int Width;
    public int Height;
}

var p1 = new Point { X = 1, Y = 2 };
var p2 = p1; // copies the VALUES of X and Y
p2.X = 99;
Console.WriteLine(p1.X); // 1 — p1 didn't change

var r1 = new Rectangle { Width = 1, Height = 2 };
var r2 = r1; // copies the REFERENCE to the same object
r2.Width = 99;
Console.WriteLine(r1.Width); // 99 — r1 changed too
```

👉 **`struct` is a value type: assignment copies the data. `class` is a reference type: assignment copies the address, and both names point to the same object**

This behavior isn't a syntax detail — it's the fundamental difference that explains subtle bugs when someone assumes `p2 = p1` means "two independent copies" when, for classes, it actually means "two names for the same object."

---

# 🗺️ Where each one lives: stack vs heap

👉 Value types **generally** live on the stack (fast memory, automatically allocated and freed when leaving scope). Reference types always have their data on the heap, with the stack holding only the address

```csharp
void Method()
{
    int number = 42;             // the value 42 sits on the stack
    var person = new Person();   // the Person object sits on the heap; "person" (the reference) sits on the stack
}
```

👉 This is a useful simplification, but has one important exception: a `struct` that is a **field of a class**, or lives inside an array, sits on the heap along with the object that contains it — the value type only "borrows" the stack when it's a standalone local variable or parameter

---

# ⚖️ Equality: another behavior that changes

```csharp
struct Point { public int X, Y; }
class Rectangle { public int Width, Height; }

var p1 = new Point { X = 1, Y = 2 };
var p2 = new Point { X = 1, Y = 2 };
Console.WriteLine(p1.Equals(p2)); // true — structs compare by value, field by field

var r1 = new Rectangle { Width = 1, Height = 2 };
var r2 = new Rectangle { Width = 1, Height = 2 };
Console.WriteLine(r1.Equals(r2)); // false — classes compare by reference, by default
```

👉 Structs inherit from `ValueType`, which already implements `Equals` by comparing field by field. Classes inherit from `object`, whose default `Equals` checks whether they're the **same object** in memory — the same distinction you've already seen in practice with `record` (value-based equality) versus a regular `class` (reference-based equality), except now you know the reasoning behind the behavior

---

# 🎯 When to use `struct` instead of `class`

```csharp
public readonly struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
}
```

👉 **Practical rule: prefer `struct` for small, immutable types that represent a single conceptual value — coordinates, money, date ranges**

- **Use `struct`** when the type is small (few fields), immutable, and makes sense to be freely copied without side effects  
- **Use `class`** for the vast majority of cases: entities with identity, objects that change state over time, or anything large enough that copying the entire contents would be expensive  

The `readonly` on the struct above isn't decoration: it tells the compiler that no internal method modifies state, avoiding a real problem called "defensive copying," where the runtime silently copies the struct every time it calls a method on it, just as a precaution.

---

# 🔍 Primitive types and `string`: where they fit in

```csharp
int number = 10;       // struct (System.Int32) — value type
bool active = true;     // struct (System.Boolean) — value type
string name = "Vitor";  // class (System.String) — reference type, but immutable
```

👉 All numeric types (`int`, `double`, `decimal`, `bool`, `char`) are structs under the hood. `string`, on the other hand, is a class — but behaves in a way that confuses a lot of people, because it's **immutable**: every "modification" creates a brand-new string, so in practice it feels like a value type even though it's a reference

```csharp
string a = "Vitor";
string b = a;
b += " Santos"; // creates a NEW string; "a" is still "Vitor"
Console.WriteLine(a); // Vitor
```

---

# ⚠️ Common Mistakes

- Assuming copying a large `struct` is always cheaper than copying a reference — large structs (many fields) can be **more** expensive to copy than passing an 8-byte reference  
- Creating mutable `struct`s and storing them in collections, causing bugs where a modification seems not to "stick" (because each access can return a copy)  
- Comparing objects with `==` expecting value-based equality on a regular `class`, without realizing the default is reference comparison  
- Ignoring `readonly struct` and suffering from silent "defensive copying" in performance-sensitive code  

---

# 📌 Conclusion

- Value types (`struct`) copy data on assignment; reference types (`class`) copy the reference  
- Value types generally live on the stack; reference types always have their data on the heap  
- Structs compare by value by default; classes compare by reference by default  
- Prefer `struct` for small, immutable types; `class` for most other cases  
- `string` is a class, but its immutability makes it behave similarly to a value type  

👉 Understanding this distinction sets up the next performance problem: what happens when a value type needs to be treated as an `object`

---

# 🔥 Next Step

Now that you understand where each type lives in memory, the next level is:

👉 **C# Fundamentals: Boxing, Unboxing and Type Performance**

Here you'll see what happens — and the real cost — when a `struct` needs to be converted to `object`, and how to avoid it in performance-sensitive code.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
