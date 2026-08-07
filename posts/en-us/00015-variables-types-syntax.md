# 🧠 C# Fundamentals: Variables, Types, and Basic Syntax

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- What IL and the CLR are, and how .NET runs your code  
- The difference between SDK and Runtime  
- How to create and run your first program  

Now it's time to learn what every program needs to do:

👉 **Store and manipulate data**

---

# 💡 What is a variable?

👉 **Variable = a named space in memory that holds a value**

```csharp
int age = 30;
```

Here you have three parts:

- `int` → the **type** of the data  
- `age` → the variable's **name**  
- `30` → the assigned **value**  

👉 C# is a **strongly typed** language: every variable has a defined type, and the compiler checks it before the program even runs

---

# 🧱 The types you'll use most

## 🔹 Numbers
- `int` → whole numbers (`10`, `-5`)  
- `double` → decimal numbers (`3.14`)  
- `decimal` → high precision, ideal for financial values  

## 🔹 Text and characters
- `string` → text (`"Hello, world"`)  
- `char` → a single character (`'A'`)  

## 🔹 Logic
- `bool` → `true` or `false`  

👉 Picking the right type avoids subtle bugs, especially with decimal numbers and money

---

# 🔀 Value types vs reference types

This is a fundamental distinction in C#:

## 🔹 Value types
- `int`, `double`, `bool`, `struct`  
- Store the data directly  
- Copying the variable copies the value  

## 🔹 Reference types
- `string`, `class`, arrays, objects  
- Store an **address** that points to the data  
- Copying the variable copies the reference, not the data  

👉 Understanding this avoids surprises when you change an object and the change "shows up" somewhere else in the code

---

# ⚙️ Type inference with `var`

You can let the compiler figure out the type:

```csharp
var name = "Vitor";     // becomes string
var total = 99.90;      // becomes double
```

👉 `var` doesn't make the variable dynamic — the type is set at compile time and never changes afterward

## 🔹 When to use `var`

- When the type is already obvious from the value  
- When the variable's name already makes clear what it represents  

👉 When in doubt, be explicit — clarity beats "saving keystrokes"

---

# 🔒 Constants and immutable values

Not every value should be allowed to change:

```csharp
const double Pi = 3.14159;
readonly string AppName = "MyApp";
```

## 🔹 `const`
- A fixed value, set at compile time  

## 🔹 `readonly`
- Can only be set in the constructor  
- Useful for values that vary per instance but shouldn't change afterward  

---

# 🔄 Converting between types

Sometimes you need to turn one type into another:

```csharp
double price = 10;              // implicit conversion (int → double)
int quantity = (int)9.9;        // explicit conversion (cast) → 9
string text = "42";
int number = int.Parse(text);   // string to int conversion
```

## 🔹 Implicit vs explicit

- **Implicit**: safe, no data loss (`int` → `double`)  
- **Explicit (cast)**: can lose data, requires `(type)` in front  

👉 Conversions from `string` can always fail — use `int.TryParse` when the value isn't guaranteed

---

# ⚠️ Common Mistakes

- Using `double` for money instead of `decimal`  
- Thinking `var` makes C# a "typeless" language  
- Confusing value copies with reference copies on objects  
- Using `Parse` without validating input, crashing the app with an exception  

---

# 📌 Conclusion

- Every variable in C# has a defined **type**  
- Value types copy the data; reference types copy the address  
- `var` infers the type but doesn't remove strong typing  
- `const` and `readonly` protect values that shouldn't change  

👉 Mastering variables and types is the foundation for writing any logic in C#

---

# 🔥 Next Step

Now that you know how to store and manipulate data, the next level is:

👉 **C# Fundamentals: Control Flow (if, else, switch, and loops)**

Here you'll learn to control your program's flow with decisions and repetition.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
