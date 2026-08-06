# 🧠 C# Fundamentals: Collections (Arrays, Lists, and Dictionaries)

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Variables, types, and conversions  
- Conditionals and loops  
- Methods, parameters, and return values  

But in practice, data rarely shows up alone — it comes in groups: a list of users, a shopping cart, a dictionary of settings.

👉 **Time to learn how to store and manipulate groups of data**

---

# 💡 Arrays: the most basic collection

👉 **Array = a fixed-size collection where all elements share the same type**

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
Console.WriteLine(numbers[0]); // 1
```

## 🔹 Characteristics

- Size is set at creation and **never changes afterward**  
- Fast access by index (`numbers[2]`)  
- Indexes start at `0`  

```csharp
string[] names = new string[3];
names[0] = "Maria";
names[1] = "João";
names[2] = "Valentina";
```

👉 Arrays are great when you already know exactly how many elements you'll need

---

# 📦 `List<T>`: when the size changes

In most real cases, you don't know how many items you'll have. That's where `List<T>` comes in:

```csharp
List<string> names = new List<string>();
names.Add("Maria");
names.Add("João");
names.Remove("Maria");

Console.WriteLine(names.Count); // 1
```

## 🔹 Common operations

- `Add(item)` → adds an element  
- `Remove(item)` → removes by the value's reference  
- `RemoveAt(index)` → removes by position  
- `Contains(item)` → checks if it exists  
- `Count` → number of elements  

👉 `List<T>` is the most commonly used collection day to day — it grows and shrinks automatically

## 🔹 Iterating over a list

```csharp
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

---

# 🗂️ `Dictionary<K, V>`: key-value pairs

When you need to look up values by a key instead of a numeric index:

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>();
ages["Maria"] = 25;
ages["João"] = 30;

Console.WriteLine(ages["Maria"]); // 25
```

## 🔹 Common operations

- `dictionary[key] = value` → adds or updates  
- `ContainsKey(key)` → checks if the key exists  
- `Remove(key)` → removes by key  
- `TryGetValue(key, out value)` → safe lookup, no exception thrown  

```csharp
if (ages.TryGetValue("Valentina", out int age))
{
    Console.WriteLine(age);
}
else
{
    Console.WriteLine("Not found");
}
```

👉 Use `TryGetValue` whenever the key isn't guaranteed to exist — it avoids `KeyNotFoundException`

## 🔹 Iterating over a dictionary

```csharp
foreach (KeyValuePair<string, int> pair in ages)
{
    Console.WriteLine($"{pair.Key}: {pair.Value}");
}
```

---

# 🔀 Array vs List vs Dictionary: which one to pick?

## 🔹 Use `Array` when:
- The size is fixed and known  
- Maximum performance is the priority  

## 🔹 Use `List<T>` when:
- The size can change  
- You need order and access by index  

## 🔹 Use `Dictionary<K, V>` when:
- You look up values by a key, not by position  
- Keys need to be unique  

👉 When in doubt, `List<T>` is usually the safest starting point

---

# ⚠️ Common Mistakes

- Trying to add items to an array (arrays don't grow — use `List<T>`)  
- Accessing a missing key in a `Dictionary` without using `TryGetValue`  
- Using `List<T>` when a `Dictionary` would solve the lookup more efficiently  
- Forgetting that indexes start at `0`, causing `IndexOutOfRangeException`  

---

# 📌 Conclusion

- `Array` stores fixed-size collections  
- `List<T>` grows and shrinks dynamically  
- `Dictionary<K, V>` looks up values by key, not by position  
- Choosing the right collection avoids complicated, inefficient code  

👉 With these three structures, you can already model most day-to-day problems

---

# 🔥 Next Step

Now that you know how to store groups of data, the next level is:

👉 **C# Fundamentals: Introduction to LINQ**

Here you'll learn to query, filter, and transform collections in a much more expressive way.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
