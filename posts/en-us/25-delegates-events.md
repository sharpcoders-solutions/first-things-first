# 🧠 C# Fundamentals: Delegates, Events, and Lambda Expressions

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Generics: reusable, type-safe code  
- Interfaces and contracts between classes  

So far, methods have always been called directly. But what if you needed to **pass a method as a parameter**, or notify other parts of the system when something happens?

👉 **That's exactly what delegates, events, and lambdas are for**

---

# 💡 What is a delegate?

👉 **Delegate = a type that represents a reference to a method**

```csharp
delegate int Operation(int a, int b);

int Sum(int a, int b) => a + b;
int Multiply(int a, int b) => a * b;

Operation op = Sum;
Console.WriteLine(op(2, 3)); // 5

op = Multiply;
Console.WriteLine(op(2, 3)); // 6
```

👉 Just like an `int` variable holds a number, an `Operation` variable holds **a method** — and you can change which method it points to at runtime

---

# 📦 Built-in delegates: `Action`, `Func`, and `Predicate`

You rarely need to create your own delegates — .NET already provides the most common ones:

## 🔹 `Action<T>` — a method with no return value

```csharp
Action<string> print = message => Console.WriteLine(message);
print("Hello!");
```

## 🔹 `Func<T, TResult>` — a method with a return value

```csharp
Func<int, int, int> sum = (a, b) => a + b;
int result = sum(2, 3); // 5
```

## 🔹 `Predicate<T>` — a method that returns `bool`

```csharp
Predicate<int> isEven = number => number % 2 == 0;
bool result2 = isEven(4); // true
```

👉 `Func` and `Predicate` already showed up in disguise in the LINQ post — `Where(n => n % 2 == 0)` takes exactly a `Func<T, bool>`

---

# ⚡ Lambda expressions: the syntax behind `=>`

```csharp
Func<int, int> doubleIt = number => number * 2;

// equivalent to:
int DoubleIt(int number)
{
    return number * 2;
}
```

👉 A lambda is just a leaner way to write a method without needing to name it — heavily used when the method only exists to be passed as an argument

## 🔹 Methods as parameters (callbacks)

```csharp
void ProcessList(List<int> numbers, Action<int> action)
{
    foreach (int number in numbers)
    {
        action(number);
    }
}

ProcessList(new List<int> { 1, 2, 3 }, n => Console.WriteLine(n * 10));
```

👉 `ProcessList` doesn't know (and doesn't need to know) what `action` does — it just runs whatever was passed in

---

# 🔔 Events: communication between objects

👉 **Event = a special delegate that notifies other parts of the system when something happens**

```csharp
class Order
{
    public event Action<string> StatusChanged;

    private string status;

    public string Status
    {
        get => status;
        set
        {
            status = value;
            StatusChanged?.Invoke(status); // notifies whoever is "listening"
        }
    }
}
```

```csharp
Order order = new Order();
order.StatusChanged += message => Console.WriteLine($"New status: {message}");
order.StatusChanged += message => Console.WriteLine($"Sending notification: {message}");

order.Status = "Shipped";
// New status: Shipped
// Sending notification: Shipped
```

👉 With `+=`, you **subscribe** a method to be called when the event fires. With `-=`, you **unsubscribe**

## 🔹 Why not just use a public `Action` directly?

```csharp
public Action<string> StatusChanged; // ❌ any outside code can call this directly

order.StatusChanged("Hacked!"); // this shouldn't be allowed from outside
```

👉 The `event` keyword prevents outside code from **firing** the event directly — only the class itself can invoke it. This is encapsulation applied to communication between objects

---

# ✅ `?.Invoke`: avoiding `NullReferenceException`

```csharp
StatusChanged?.Invoke(status);
```

👉 If no one subscribed to the event, it's `null` — the `?.` avoids throwing an exception when trying to call an event with no listeners

---

# ⚠️ Common Mistakes

- Forgetting the `?.` before `Invoke`, causing a `NullReferenceException` when there are no subscribers  
- Failing to unsubscribe (`-=`) from events, causing memory leaks in long-lived objects  
- Using a public `Action` instead of `event`, losing the encapsulation  
- Writing large, complex lambdas when a named method would be more readable  

---

# 📌 Conclusion

- Delegates let you treat methods as values stored in variables  
- `Action`, `Func`, and `Predicate` cover the vast majority of day-to-day cases  
- Lambdas are the leaner way to write anonymous methods  
- `event` adds a layer of encapsulation over delegates, essential for notifying changes between objects  

👉 With delegates, lambdas, and events, your code gains a new level of flexibility — the foundation of callbacks, LINQ, and event-driven programming in C#

---

# 🔥 Next Step

Now that you know how to treat methods as values, the next level is:

👉 **C# Fundamentals: Async/Await in Practice**

Here you'll apply, with real C# syntax, the asynchronous programming concepts you already saw in theory.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
