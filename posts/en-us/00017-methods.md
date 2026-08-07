# 🧠 C# Fundamentals: Methods, Parameters, and Return Values

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Variables, types, and conversions  
- Conditionals and loops  

Your code can already make decisions and repeat tasks. But as it grows, repeating the same blocks in multiple places becomes a problem.

👉 **Time to organize your code into reusable blocks: methods**

---

# 💡 What is a method?

👉 **Method = a named block of code that performs a specific task**

```csharp
void Greet()
{
    Console.WriteLine("Hello!");
}

Greet(); // calling the method
```

A method has four main parts:

- **Return type** (`void`, `int`, `string`...)  
- **Name** (`Greet`)  
- **Parameters** (in this case, none)  
- **Body** (the code between `{ }`)  

👉 Without methods, every program would become a single giant, hard-to-maintain block

---

# 📥 Parameters: passing data in

```csharp
void Greet(string name)
{
    Console.WriteLine($"Hello, {name}!");
}

Greet("Vitor");
```

## 🔹 Optional parameters

```csharp
void Greet(string name = "visitor")
{
    Console.WriteLine($"Hello, {name}!");
}

Greet();          // "Hello, visitor!"
Greet("Vitor");   // "Hello, Vitor!"
```

## 🔹 Named arguments

```csharp
void CreateUser(string name, int age, bool active) { }

CreateUser(name: "Maria", active: true, age: 25);
```

👉 Named arguments make the call clearer, especially with many parameters

---

# 📤 Returning values

```csharp
int Sum(int a, int b)
{
    return a + b;
}

int result = Sum(2, 3); // 5
```

👉 The declared return type (`int`) must match what `return` gives back

## 🔹 `void`: when there's no return

```csharp
void LogMessage(string message)
{
    Console.WriteLine($"[LOG] {message}");
}
```

👉 `void` methods perform an action but don't hand back a value to the caller

---

# 🔀 `ref`, `out`, and `params`

C# has special ways to handle parameters:

## 🔹 `ref` — pass by reference

```csharp
void Double(ref int number)
{
    number *= 2;
}

int value = 5;
Double(ref value); // value is now 10
```

👉 The original variable is changed inside the method — use it carefully, it can hurt readability

## 🔹 `out` — return additional values

```csharp
bool TryDivide(int a, int b, out int result)
{
    if (b == 0)
    {
        result = 0;
        return false;
    }
    result = a / b;
    return true;
}

if (TryDivide(10, 2, out int r))
{
    Console.WriteLine(r); // 5
}
```

👉 A common pattern in `TryX` methods, like `int.TryParse`

## 🔹 `params` — a variable number of arguments

```csharp
int Sum(params int[] numbers)
{
    int total = 0;
    foreach (int n in numbers) total += n;
    return total;
}

Sum(1, 2, 3, 4); // 10
```

---

# ✍️ Method overloading

You can have methods with the same name, as long as the parameters differ:

```csharp
int Sum(int a, int b) => a + b;
double Sum(double a, double b) => a + b;
```

👉 The compiler picks the right version based on the types passed in the call

---

# ⚡ Expression-bodied methods

For simple methods, there's a leaner syntax:

```csharp
int Sum(int a, int b) => a + b;
```

👉 Equivalent to writing `{ return a + b; }`, but more direct for one-line logic

---

# ⚠️ Common Mistakes

- Overusing `ref`/`out` when a normal return would already solve it  
- Creating giant methods that do several things at once  
- Forgetting that optional parameters must come after required ones  
- Confusing method overloading with methods that do entirely different things  

---

# 📌 Conclusion

- Methods organize code into reusable, testable blocks  
- Optional and named parameters make calls more flexible and clear  
- `ref`, `out`, and `params` cover special data-passing cases  
- Overloading allows variations of the same method for different types  

👉 With well-defined methods, your code becomes more organized, readable, and easier to test

---

# 🔥 Next Step

Now that you know how to organize logic into methods, the next level is:

👉 **C# Fundamentals: Collections (Arrays, Lists, and Dictionaries)**

Here you'll learn to store and manipulate groups of data.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
