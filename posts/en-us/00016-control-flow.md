# 🧠 C# Fundamentals: Control Flow (if, else, switch, and loops)

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to create and run your first program  
- Variables, types, and how to convert data  

But so far, your code only does one thing: it runs line by line, without deciding anything.

👉 **Time to control the flow of the program**

---

# 💡 Conditionals: `if`, `else if`, and `else`

The most basic decision structure in C#:

```csharp
int age = 20;

if (age >= 18)
{
    Console.WriteLine("Adult");
}
else if (age >= 12)
{
    Console.WriteLine("Teenager");
}
else
{
    Console.WriteLine("Child");
}
```

👉 The block that runs is always the **first** one whose condition is true

## 🔹 Ternary operator

For simple decisions, there's a leaner version:

```csharp
string status = age >= 18 ? "Adult" : "Minor";
```

👉 Useful for short assignments — avoid it for complex decisions, it hurts readability

---

# 🔀 `switch`: when there are multiple options

When you have many conditions on the same value, `switch` reads better than a chain of `if/else if`:

```csharp
switch (dayOfWeek)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    default:
        Console.WriteLine("Another day");
        break;
}
```

👉 Unlike other languages, C# does **not** allow silent fallthrough — every `case` needs a `break` (or `return`)

## 🔹 Switch expression (modern C#)

A more concise form, ideal when the goal is just to return a value:

```csharp
string dayName = dayOfWeek switch
{
    1 => "Monday",
    2 => "Tuesday",
    _ => "Another day"
};
```

👉 `_` works like `default` — it captures any unhandled value

---

# 🔁 Loops: repeating code

## 🔹 `for`

Ideal when you know **how many times** to repeat:

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);
}
```

## 🔹 `while`

Repeats **while** the condition is true — useful when you don't know the exact number of repetitions:

```csharp
int attempts = 0;
while (attempts < 3)
{
    Console.WriteLine("Trying...");
    attempts++;
}
```

## 🔹 `do while`

Same as `while`, but guarantees the block runs **at least once**:

```csharp
int number;
do
{
    number = new Random().Next(1, 10);
} while (number != 7);
```

## 🔹 `foreach`

Built for iterating over collections, without worrying about indexes:

```csharp
string[] names = { "Maria", "João", "Valentina" };

foreach (string name in names)
{
    Console.WriteLine(name);
}
```

👉 Use `foreach` whenever you don't need to modify the collection or control the index manually

---

# ⏹️ Controlling the loop: `break` and `continue`

- `break` → ends the loop immediately  
- `continue` → skips to the next iteration, without ending the loop  

```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 5) break;       // stops at 5
    if (i % 2 == 0) continue; // skips even numbers
    Console.WriteLine(i);
}
```

---

# ⚠️ Common Mistakes

- Forgetting `break` in each `case` of a traditional `switch`  
- Creating infinite loops by forgetting to update the condition  
- Using `for` when a `foreach` would make the code clearer  
- Overusing the ternary operator on complex conditions, hurting readability  

---

# 📌 Conclusion

- `if/else` decides based on boolean conditions  
- `switch` (traditional or expression) organizes multiple options  
- `for`, `while`, `do while`, and `foreach` cover the different repetition scenarios  
- `break` and `continue` give fine-grained control over the loop's flow  

👉 With conditionals and loops, your program can finally make decisions and repeat tasks

---

# 🔥 Next Step

Now that your program knows how to decide and repeat, the next level is:

👉 **C# Fundamentals: Methods, Parameters, and Return Values**

Here you'll learn to organize your code into reusable blocks.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
