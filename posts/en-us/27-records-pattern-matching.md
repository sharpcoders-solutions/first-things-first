# 🧠 C# Fundamentals: Records and Pattern Matching (Modern C#)

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Classes, objects, and the pillars of OOP  
- Async/await in practice  

Now let's look at two features that changed how modern C# is written: less boilerplate, more expressiveness.

👉 **Records and Pattern Matching**

---

# 💡 What is a record?

👉 **Record = a type designed to represent data, with value-based equality**

```csharp
record Product(string Name, decimal Price);
```

That single line already generates automatically:

- `Name` and `Price` properties  
- A constructor that takes both values  
- A readable `ToString()`  
- Equality by **value**, not by reference  

```csharp
var p1 = new Product("Laptop", 3500);
var p2 = new Product("Laptop", 3500);

Console.WriteLine(p1 == p2); // true — even though they're different objects in memory
```

---

# 🔀 Record vs Class: the difference that matters

```csharp
class ProductClass
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}

var c1 = new ProductClass { Name = "Laptop", Price = 3500 };
var c2 = new ProductClass { Name = "Laptop", Price = 3500 };

Console.WriteLine(c1 == c2); // false — compares reference, not value
```

👉 With `class`, `==` checks whether they're **the same object** in memory. With `record`, `==` checks whether the **values** are equal

**Practical rule:** use `record` when the object represents **data** (a DTO, an API response, an immutable value). Use `class` when the object represents an **entity with identity and behavior**, like you saw in the OOP posts.

---

# 🧊 Immutability and `with`

By default, a record's properties are `init` — they can only be set at creation:

```csharp
record Product(string Name, decimal Price);

var original = new Product("Laptop", 3500);
// original.Price = 4000; // ❌ compile error
```

To "change" a record, you create a **copy** with different values:

```csharp
var discounted = original with { Price = 2999 };

Console.WriteLine(original.Price);    // 3500 — original untouched
Console.WriteLine(discounted.Price);  // 2999 — new instance
```

👉 `with` creates a new instance by copying the existing values, changing only what you specify — this is called **non-destructive mutation**

---

# 🧱 `record class` vs `record struct`

```csharp
record class Product(string Name, decimal Price);   // reference type (default)
record struct Point(int X, int Y);                   // value type
```

👉 By default, `record` is a reference type. Since C# 10, `record struct` lets you get the same benefits (value equality, `with`) on a value type, which is lighter for cases like coordinates or small data structures

---

# 📦 Deconstruction

```csharp
var product = new Product("Laptop", 3500);
var (name, price) = product;

Console.WriteLine(name);  // Laptop
Console.WriteLine(price); // 3500
```

👉 Records automatically generate a deconstruction method, letting you "unpack" the values straight into variables

---

# 🔍 Pattern Matching: beyond the basic `switch`

You already saw `switch expressions` in the control flow post. Pattern matching goes further, letting you check **type, properties, and values** all at once.

## 🔹 Type pattern

```csharp
object value = 42;

if (value is int number)
{
    Console.WriteLine($"It's an integer: {number}");
}
```

## 🔹 Property pattern

```csharp
Product product = new Product("Laptop", 3500);

string category = product switch
{
    { Price: > 3000 } => "Premium",
    { Price: > 1000 } => "Mid-range",
    _ => "Budget"
};
```

👉 The `switch` is directly evaluating the object's `Price` **property**, without needing explicit `product.Price` in each `case`

## 🔹 Relational and logical patterns

```csharp
string ClassifyAge(int age) => age switch
{
    < 12 => "Child",
    >= 12 and < 18 => "Teenager",
    >= 18 => "Adult"
};
```

👉 `and`, `or`, and `not` combine conditions directly inside the pattern, without needing nested `if`s

---

# 🏗️ Real-world example: combining records and pattern matching

```csharp
record Order(string Customer, decimal Total, string Status);

string Describe(Order order) => order switch
{
    { Status: "Cancelled" } => $"{order.Customer}'s order was cancelled",
    { Total: > 1000, Status: "Pending" } => $"{order.Customer}'s large order is awaiting payment",
    { Status: "Delivered" } => $"{order.Customer}'s order has already been delivered",
    _ => $"{order.Customer}'s order is in progress"
};

var order = new Order("Maria", 1500, "Pending");
Console.WriteLine(Describe(order));
// Maria's large order is awaiting payment
```

👉 Records keep the data model lean, and pattern matching makes the decision logic declarative and easy to read

---

# ⚠️ Common Mistakes

- Using `record` for entities that need identity and mutable behavior (that's `class`'s job)  
- Forgetting that `with` creates a **copy**, it doesn't change the original  
- Stacking nested `if`s when a `switch` with a property pattern would solve it more cleanly  
- Thinking `record` fully replaces `class` — they solve different problems  

---

# 📌 Conclusion

- `record` is ideal for representing data, with value-based equality and immutability by default  
- `with` lets you create modified copies without changing the original object  
- `record struct` brings record benefits to value types  
- Pattern matching (type, property, relational, logical) makes complex decisions more declarative  

👉 With records and pattern matching, your C# gets more modern, concise, and expressive

---

# 🔥 Next Step

Now that you know the language's modern features, the next level is:

👉 **C# Fundamentals: SOLID Principles (Introduction to Software Design)**

Here you'll start moving beyond language syntax and into the world of professional software architecture and design.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
