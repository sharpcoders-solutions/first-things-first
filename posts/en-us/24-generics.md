# 🧠 C# Fundamentals: Generics

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Collections like `List<T>` and `Dictionary<K, V>`  
- Interfaces, inheritance, and exception handling  

You've been using `List<T>` since the collections post, but maybe you never stopped to ask: **why does that `T` exist?**

👉 **Today you'll understand the mechanism behind it: Generics**

---

# 💡 The problem Generics solve

Imagine writing a `Box` class without generics:

```csharp
class IntBox
{
    public int Item { get; set; }
}

class StringBox
{
    public string Item { get; set; }
}
```

👉 Same code, repeated for every type. And what if you need a box for `Person`, `Product`, `Order`...? The duplication explodes.

## 🔹 The bad alternative: using `object`

```csharp
class GenericBox
{
    public object Item { get; set; }
}

GenericBox box = new GenericBox();
box.Item = "text";
int value = (int)box.Item; // 💥 InvalidCastException at runtime
```

👉 Using `object` solves the duplication, but loses type safety — the error only shows up once the program is already running

---

# 🧱 The solution: generic classes

```csharp
class Box<T>
{
    public T Item { get; set; }
}
```

```csharp
Box<int> intBox = new Box<int>();
intBox.Item = 10;

Box<string> textBox = new Box<string>();
textBox.Item = "Hello";

// intBox.Item = "text"; // ❌ compile error — not allowed
```

👉 `T` is a **type placeholder**: you only define the real type when using the class, and the compiler guarantees safety

This is exactly how `List<T>` and `Dictionary<K, V>` work under the hood — now you know the "why" behind the `<T>` you've already been using.

---

# ⚙️ Generic methods

Generics aren't exclusive to classes — methods can be generic too:

```csharp
T GetFirst<T>(List<T> list)
{
    return list[0];
}

int firstNumber = GetFirst(new List<int> { 1, 2, 3 });
string firstName = GetFirst(new List<string> { "Maria", "João" });
```

👉 The compiler infers the `T` type automatically based on the argument passed — you rarely need to write `GetFirst<int>(...)` explicitly

---

# 🧩 Multiple type parameters

A generic class (or method) can have more than one placeholder:

```csharp
class Pair<T1, T2>
{
    public T1 First { get; set; }
    public T2 Second { get; set; }
}

var pair = new Pair<string, int> { First = "age", Second = 30 };
```

👉 This is exactly how `Dictionary<TKey, TValue>` is implemented internally

---

# 🔒 Type constraints (`where`)

Sometimes you need to guarantee that `T` has certain capabilities. That's where **constraints** come in:

```csharp
class Repository<T> where T : class
{
    private List<T> items = new List<T>();

    public void Add(T item) => items.Add(item);
}
```

## 🔹 The most common constraints

- `where T : class` → `T` must be a reference type  
- `where T : struct` → `T` must be a value type  
- `where T : new()` → `T` must have a parameterless constructor  
- `where T : IComparable<T>` → `T` must implement that interface  
- `where T : Animal` → `T` must be (or inherit from) `Animal`  

```csharp
T CreateNew<T>() where T : new()
{
    return new T(); // only possible because of the constraint
}
```

👉 Without the right constraint, the compiler won't allow operations that depend on a specific capability of the type

---

# 🕳️ `default(T)`: the "empty" value of any type

```csharp
T GetDefaultValue<T>()
{
    return default(T); // 0 for int, null for string, false for bool...
}
```

👉 `default(T)` returns the appropriate default value regardless of the type used — essential when you don't know whether `T` is a value or reference type

---

# 🏗️ Real-world example: a generic repository

```csharp
class Repository<T> where T : class
{
    private readonly List<T> _items = new List<T>();

    public void Add(T item) => _items.Add(item);
    public void Remove(T item) => _items.Remove(item);
    public IEnumerable<T> ListAll() => _items;
}

class Product
{
    public string Name { get; set; }
}

class Customer
{
    public string Name { get; set; }
}
```

```csharp
var productRepository = new Repository<Product>();
productRepository.Add(new Product { Name = "Laptop" });

var customerRepository = new Repository<Customer>();
customerRepository.Add(new Customer { Name = "João" });
```

👉 A single `Repository<T>` class works for `Product`, `Customer`, or any other entity — without duplicating code and without losing type safety

---

# ⚠️ Common Mistakes

- Using `object` instead of generics, losing type safety  
- Forgetting needed constraints and trying operations `T` doesn't guarantee support for  
- Creating generics where a simple interface would already solve the problem  
- Confusing `Box<T>` (the definition) with `Box<int>` (the closed, ready-to-use type)  

---

# 📌 Conclusion

- Generics eliminate code duplication without giving up type safety  
- `T` is replaced by the real type only at the point of use  
- Constraints (`where`) guarantee `T` has the capabilities your code needs  
- `List<T>` and `Dictionary<K, V>` are generics you've been using since early in this track  

👉 With generics, you write truly reusable code — a core piece of any .NET library or framework

---

# 🔥 Next Step

Now that you know how to write generic, reusable code, the next level is:

👉 **C# Fundamentals: Delegates, Events, and Lambda Expressions**

Here you'll learn to treat methods as values — the foundation of callbacks, events, and functional programming in C#.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
