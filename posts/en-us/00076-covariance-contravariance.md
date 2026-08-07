# 🧠 C# Fundamentals: Covariance and Contravariance

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Functional programming and function composition  
- Generics (post 27) and inheritance/polymorphism (post 24)  

You've already written code like this without stopping to think about why it works: `IEnumerable<string>` can be used where `IEnumerable<object>` is expected. That's not an accident — it's covariance, and it follows precise rules.

👉 **Let's learn Covariance and Contravariance**

---

# 💡 The problem that motivates this

```csharp
public class Animal { }
public class Dog : Animal { }

List<Dog> dogs = new();
List<Animal> animals = dogs; // ❌ won't compile!
```

👉 Even though `Dog` is an `Animal` (remember inheritance from post 24?), `List<Dog>` is **not** automatically a `List<Animal>` — because lists are mutable, and allowing this would break type safety

```csharp
animals.Add(new Cat()); // if this compiled, a List<Dog> would have a Cat in it!
```

---

# 🏗️ Covariance: when it's safe (read-only)

```csharp
IEnumerable<Dog> dogs = new List<Dog> { new Dog() };
IEnumerable<Animal> animals = dogs; // ✅ compiles!

foreach (Animal animal in animals)
{
    Console.WriteLine(animal.GetType().Name);
}
```

👉 `IEnumerable<T>` is covariant (declared as `IEnumerable<out T>`) because it's **read-only** — you can only read items from it, never add to it. No risk of putting a `Cat` where a `Dog` was expected

---

# 🎯 Contravariance: the reverse, for input

```csharp
public class AnimalComparer : IComparer<Animal>
{
    public int Compare(Animal x, Animal y) => 0; // simplified example
}

IComparer<Dog> comparer = new AnimalComparer(); // ✅ compiles!
```

👉 `IComparer<T>` is contravariant (`IComparer<in T>`) — a comparer that knows how to compare **any** `Animal` also knows how to compare `Dog` specifically. The direction is reversed compared to covariance

---

# 🔍 Where you've already used this, without noticing

```csharp
Func<Animal, string> describeAnimal = a => $"Animal: {a.GetType().Name}";
Func<Dog, string> describeDog = describeAnimal; // ✅ contravariance on the parameter

Action<Animal> processAnimal = a => Console.WriteLine(a);
Action<Dog> processDog = processAnimal; // ✅ same logic
```

👉 Remember the delegates post (25)? `Func<in T, out TResult>` combines both: contravariant on the input parameter, covariant on the return value — that's the general rule: **covariant output, contravariant input**

---

# ⚠️ Common Mistakes

- Trying to apply covariance to mutable collections (`List<T>`, `T[]`), expecting the same behavior as `IEnumerable<T>`  
- Confusing the direction — remember that "covariant" follows the same direction as inheritance (out), "contravariant" is the opposite (in)  
- Marking a generic as `out` or `in` without ensuring the interface actually respects those constraints, producing confusing compile errors  
- Ignoring that arrays in C# are covariant due to the language's historical design, but unsafely so (they can throw `ArrayTypeMismatchException` at runtime)  

---

# 📌 Conclusion

- Covariance (`out`) lets you substitute a more derived type where a more generic one is expected, in read-only contexts  
- Contravariance (`in`) allows the reverse, in input/parameter contexts  
- `IEnumerable<T>`, `Func<>`, and `IComparer<T>` already use these rules behind the scenes  
- The general rule: covariant output, contravariant input  

👉 With covariance and contravariance, you understand why certain generic conversions compile and others don't — it's not compiler inconsistency, it's type safety in action

---

# 🔥 Next Step

Now that you understand variance in generics, the next level is:

👉 **C# Fundamentals: Expression Trees**

Here you'll learn how C# represents code as data, the foundation of LINQ to SQL and ORM libraries.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
