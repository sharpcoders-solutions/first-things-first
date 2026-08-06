# 🧠 C# Fundamentals: Setting Up Your Development Environment

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How computers work  
- Git, GitHub, and a team's workflow  
- What IL and the CLR are, and how .NET runs your code  

All that theory only becomes practice once you have one thing installed on your machine:

👉 **The .NET SDK**

---

# 💡 What is the SDK?

👉 **SDK = Software Development Kit**

It's the full package you install to **develop** .NET applications. It includes:

- The compiler (`Roslyn`)  
- The CLI (`dotnet`)  
- The Runtime  
- Libraries and project templates  

👉 If you're going to write C# code, the SDK is the minimum you need

---

# ⚙️ What is the Runtime?

👉 **Runtime = what executes .NET applications that are already built**

It contains only what's needed to **run** a program, including:

- The CLR (Common Language Runtime)  
- The base libraries needed for execution  

👉 The Runtime doesn't compile code — it only runs what's already been compiled

---

# 🔀 SDK vs Runtime: what's the difference?

This is a common source of confusion for beginners.

## 🔹 SDK
- Used by **whoever develops**  
- Includes the Runtime  
- Allows building, testing, and publishing  
- Larger in size  

## 🔹 Runtime
- Used by **whoever only runs** the application  
- Doesn't compile code  
- Ideal for production servers  
- Smaller in size  

👉 In short: **every SDK includes a Runtime, but not every Runtime includes an SDK**

---

# 🏗️ Installing the .NET SDK

The process is simple and cross-platform:

1. Go to [dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)  
2. Download the **SDK** installer (not the Runtime) for your operating system  
3. Run the installer  
4. Restart your terminal  

## 🔹 Confirming the installation

After installing, open your terminal and run:

```bash
dotnet --version
```

👉 This should show the installed SDK version

To see everything installed on your machine (SDKs and Runtimes):

```bash
dotnet --list-sdks
dotnet --list-runtimes
```

---

# 🧱 Multiple SDK versions

It's common to have more than one .NET version installed at the same time — for example, a legacy project on .NET 6 and a new one on .NET 8.

👉 This isn't a problem: `dotnet` knows how to pick the right version

When you need to pin a specific version for a project, there's the `global.json` file:

```json
{
  "sdk": {
    "version": "8.0.100"
  }
}
```

👉 It guarantees the project always uses the defined version, even with several installed

---

# ⚠️ Common Mistakes

- Installing only the Runtime, thinking you can develop with it  
- Confusing the SDK version with the project's version (`TargetFramework`)  
- Not restarting the terminal after installing, and assuming "it didn't work"  
- Ignoring multiple installed versions and not knowing which one is being used  

---

# 📌 Conclusion

- The **SDK** is for developing: building, testing, publishing  
- The **Runtime** is for executing: it only runs what's already built  
- `dotnet --version` confirms your installation  
- `global.json` pins the SDK version used by a project  

👉 With the SDK installed, your environment is ready to write real code

---

# 🔥 Next Step

Now that your environment is set up, the next level is:

👉 **C# Fundamentals: Your First Program (Hello World and Project Structure)**

Here you'll create, run, and understand your first C# project in practice.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
