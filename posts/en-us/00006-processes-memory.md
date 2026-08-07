# 🧠 Processes and Memory: How Programs Actually Run

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you’ve learned:

- Hardware (who executes)  
- Software (the instructions)  
- Operating System (who coordinates everything)  

Now let’s go deeper:

👉 **How does a program actually run inside a computer?**

To understand that, we need two fundamental concepts:

👉 **Processes and Memory**

---

# 💡 What Is a Process?

A **process** is simply:

👉 **a program in execution**

When you open an application (browser, editor, etc.):

- It leaves storage (disk)  
- Moves into memory  
- Starts executing  

👉 At that moment, it becomes a process  

💡 Example:
> Opening Chrome = creating a process  

The operating system manages multiple processes at the same time.

---

# ⚙️ How Processes Work

Your computer can run multiple programs at once because the operating system:

- Splits CPU time  
- Switches rapidly between processes  

👉 This creates the illusion that everything is running simultaneously  

This control is called:

👉 **scheduling** 

---

# 🔹 What Is Memory (RAM)

**RAM (Random Access Memory)** is where programs stay while they are running.

👉 Think of it as a temporary workspace  

- More RAM → more programs running  
- When a program closes → memory is released  

💡 Important:
> RAM is fast, but temporary  
> (it loses data when the computer turns off)

---

# 🔗 The Relationship Between Process and Memory

Here’s the key idea:

👉 Every process needs memory to run  

The operating system:

- Allocates memory space for each process  
- Ensures one process does not interfere with another  

👉 This prevents errors and crashes  

This responsibility is part of:

👉 **memory management**   

---

# 🧠 The Role of the Operating System

The OS is responsible for everything happening here:

## It does:

- Creates and terminates processes  
- Allocates and frees memory  
- Controls CPU access  
- Prevents conflicts  

👉 It manages system resources to keep everything stable   

---

# 🔥 Important Concept: Isolation

Each process runs in isolation:

- Has its own memory space  
- Does not directly access other processes  

👉 This improves security and stability  

---

# 👨‍💻 What Happens When You Run a Program

Let’s break it down step by step:

1. You open a program  
2. The OS creates a process  
3. It loads the program into memory  
4. The CPU executes instructions  
5. The result appears on the screen  

👉 All of this happens in milliseconds  

---

# 🏗️ Real-World Example

When you open a browser:

- A process is created  
- Memory is allocated  
- The CPU executes instructions  
- The interface is displayed  

If you open multiple tabs:

👉 Each tab may run as a separate process  

---

# 🚀 Why This Matters

Understanding processes and memory helps you:

✅ Diagnose performance issues  
✅ Understand RAM usage  
✅ Write more efficient code  
✅ Learn advanced topics faster  

---

# 📌 Conclusion

- Process = a running program  
- Memory = where it runs  
- Operating System = who manages everything  

👉 This is the core of software execution

---

# 🔥 Next Step

Now that you understand execution, the next natural topic is:

👉 **Threads and Concurrency: How multiple tasks run at the same time**

This is where things start getting more advanced.

## ✍️ Author’s Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
