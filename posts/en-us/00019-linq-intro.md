# 🧠 C# Fundamentals: Introduction to LINQ

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Methods, parameters, and return values  
- Collections: arrays, lists, and dictionaries  

You already know how to store groups of data. But filtering, sorting, and transforming those collections with manual loops gets repetitive fast.

👉 **That's where LINQ comes in**

---

# 💡 What is LINQ?

👉 **LINQ = Language Integrated Query**

It's a set of C# features that lets you **query collections** declaratively, similar to SQL — but right inside your code.

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

var evens = numbers.Where(n => n % 2 == 0);
```

👉 Instead of writing a `foreach` with an `if` inside, you describe **what** you want, not **how** to get it

---

# 🔎 `Where`: filtering data

```csharp
var greaterThanThree = numbers.Where(n => n > 3);
// result: 4, 5, 6
```

👉 `Where` takes a condition (lambda) and returns only the elements that pass it

---

# 🔄 `Select`: transforming data

```csharp
var doubled = numbers.Select(n => n * 2);
// result: 2, 4, 6, 8, 10, 12
```

👉 `Select` transforms each element into something else — like `map` in other languages

## 🔹 Combining `Where` and `Select`

```csharp
var result = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * 10);
// result: 20, 40, 60
```

👉 LINQ methods can be **chained**, forming a pipeline of transformations

---

# 📊 Sorting and searching

```csharp
var sorted = numbers.OrderBy(n => n);            // ascending
var descending = numbers.OrderByDescending(n => n);

int first = numbers.First();                      // throws if empty
int? firstOrNull = numbers.FirstOrDefault();       // returns 0 (or null) if empty

bool hasEven = numbers.Any(n => n % 2 == 0);
int evenCount = numbers.Count(n => n % 2 == 0);
int sum = numbers.Sum();
```

## 🔹 The most commonly used methods

- `OrderBy` / `OrderByDescending` → sorts  
- `First` / `FirstOrDefault` → first element (with or without an exception)  
- `Any` → does at least one element satisfy the condition?  
- `Count` → number of elements (with or without a filter)  
- `Sum`, `Average`, `Max`, `Min` → aggregations  

👉 Prefer `FirstOrDefault` when you're not sure a result exists

---

# ✍️ Method syntax vs query syntax

LINQ has two ways to write the same query:

## 🔹 Method syntax (more common day to day)

```csharp
var evens = numbers.Where(n => n % 2 == 0).OrderBy(n => n);
```

## 🔹 Query syntax (SQL-like)

```csharp
var evens = from n in numbers
            where n % 2 == 0
            orderby n
            select n;
```

👉 Both produce the same result — method syntax is more common in the industry and easier to chain

---

# ⏳ Deferred execution

An important detail: LINQ queries **don't run at the moment they're written**.

```csharp
var query = numbers.Where(n => n > 2); // hasn't run yet

numbers.Add(10);

foreach (var n in query) // runs now, already accounting for the 10
{
    Console.WriteLine(n);
}
```

👉 The query only runs when you **iterate** over the result (`foreach`, `ToList()`, etc.)

To force immediate execution:

```csharp
var fixedList = numbers.Where(n => n > 2).ToList();
```

---

# ⚠️ Common Mistakes

- Using `First()` without making sure at least one element exists  
- Thinking a LINQ query already ran the moment it was declared  
- Chaining too many methods in a single line, hurting readability  
- Calling `Count()` multiple times instead of storing the result in a variable  

---

# 📌 Conclusion

- LINQ lets you query collections declaratively  
- `Where`, `Select`, `OrderBy`, and aggregations cover most day-to-day cases  
- Method syntax is the most commonly used form in the industry  
- Execution is deferred until the collection is iterated  

👉 With LINQ, working with collections becomes more expressive and much more readable

---

# 🔥 Next Step

Now that you know how to query and transform collections, the next level is:

👉 **C# Fundamentals: DateOnly and TimeOnly**

Here you'll meet two types that solve a classic `DateTime` problem: representing just a date, or just a time, without carrying information that doesn't make sense for your domain.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
