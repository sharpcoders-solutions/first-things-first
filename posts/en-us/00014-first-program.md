# 🧠 C# Fundamentals: Your First Program (Hello World and Project Structure)

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- What IL and the CLR are, and how .NET runs your code  
- The difference between SDK and Runtime  
- How to install the SDK and confirm the installation  

Time to move from theory to practice:

👉 **Let's create, run, and understand your first C# project**

---

# 💡 Creating a project

With the SDK installed, you can create a project directly from the terminal:

```bash
dotnet new console -o MyFirstApp
cd MyFirstApp
```

👉 The `dotnet new` command uses a **template** to generate the entire initial project structure for you

---

# 🏗️ Understanding the generated structure

After running the command, you'll see a few files and folders:

## 🔹 `MyFirstApp.csproj`
- The project's configuration file  
- Defines the **TargetFramework** (e.g., `net8.0`)  
- Lists dependencies (NuGet packages)  

## 🔹 `Program.cs`
- Where your code actually lives  
- The application's entry point  

## 🔹 `bin/` and `obj/` folders
- Generated automatically during the build  
- `obj/` holds intermediate compilation files  
- `bin/` holds the final result (the executable and compiled IL)  

👉 You should never commit `bin/` and `obj/` to Git — they're regenerated on every build

---

# 📄 What's inside `Program.cs`?

In modern versions of C#, Hello World is surprisingly simple:

```csharp
Console.WriteLine("Hello, World!");
```

👉 This is called **top-level statements** — modern C# removes the "ceremony" of manually writing `class Program` and `static void Main`

## 🔹 Under the hood

The compiler still generates that traditional structure for you, just automatically:

```csharp
namespace MyFirstApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

👉 Understanding this "full" version helps a lot once you start reading older or more explicit code

---

# ⚙️ Running the program

To run your project:

```bash
dotnet run
```

What happens behind the scenes:

1. `dotnet` compiles your source code into **IL**  
2. The CLR converts that IL into machine code via **JIT**  
3. The program runs and prints the result to the terminal  

👉 You already saw this flow in the .NET architecture post — now you're seeing it in practice

---

# 🔧 Build vs Run vs Publish

Three commands that look similar but serve different purposes:

## 🔹 `dotnet build`
- Compiles the project  
- Generates files in `bin/`  
- Doesn't run anything  

## 🔹 `dotnet run`
- Compiles (if needed) **and** runs  
- Ideal during development  

## 🔹 `dotnet publish`
- Generates a version ready for **deployment**  
- Can include the runtime (self-contained) or not  

👉 In day-to-day development, `dotnet run` is your best friend

---

# ⚠️ Common Mistakes

- Editing files inside `bin/` or `obj/`, thinking it changes the app's behavior  
- Committing `bin/` and `obj/` to Git (always use a `.gitignore`)  
- Confusing `dotnet build` with `dotnet run` and not understanding why "nothing happened"  
- Thinking top-level statements are a different language, when it's just a leaner way to write the same thing  

---

# 📌 Conclusion

- `dotnet new console` creates the initial project structure  
- `Program.cs` is where your code lives, with **top-level statements** simplifying Hello World  
- `dotnet run` compiles and runs the project  
- `bin/` and `obj/` are generated automatically and shouldn't be versioned  

👉 Now you know how to create, run, and understand a C# project from scratch

---

# 🔥 Next Step

Now that you've run your first program, the next level is:

👉 **C# Fundamentals: Variables, Types, and Basic Syntax**

Here you'll learn to actually store and manipulate data in your code.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
