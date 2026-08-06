# 🧠 C# Fundamentals: Async/Await in Practice

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Threads and concurrency, in theory  
- Delegates, events, and lambda expressions  

Back at the start of this track, you learned **why** asynchronous programming exists. Now it's time to apply it with real C# syntax.

👉 **Let's put `async` and `await` to work for real**

---

# 💡 The problem `async`/`await` solves

Imagine fetching data from an API:

```csharp
string data = FetchDataFromApi(); // this might take 2 seconds
Console.WriteLine("Processing...");
```

👉 While the call waits for a response, the **thread is blocked**, doing nothing else — in a web application, that means fewer users served at the same time

With `async`/`await`, the thread is **freed up** while it waits, and resumes executing the method once the response arrives.

---

# 🏗️ Basic syntax

```csharp
async Task<string> FetchDataAsync()
{
    await Task.Delay(2000); // simulates a slow operation (e.g., an API call)
    return "Data received";
}
```

```csharp
string result = await FetchDataAsync();
Console.WriteLine(result);
```

## 🔹 The three pieces of the syntax

- `async` → marks the method as asynchronous  
- `Task` / `Task<T>` → represents an operation that hasn't finished yet  
- `await` → pauses execution of **that method** until the task finishes, without blocking the thread  

👉 `Task` is like an asynchronous `void`; `Task<T>` is like a method that returns `T`, just asynchronously

---

# ⚠️ `async void`: the dangerous exception

```csharp
async void Save() // ❌ avoid this
{
    await SaveToDatabaseAsync();
}
```

👉 `async void` methods can't be awaited by whoever calls them, and exceptions thrown inside them can crash the application without warning

**Practical rule:** always use `async Task` (or `async Task<T>`). The one acceptable exception is event handlers (`async void OnClick(...)`), which already follow that signature by .NET convention

---

# 🧵 `async Main`: the asynchronous entry point

```csharp
async Task Main()
{
    string data = await FetchDataAsync();
    Console.WriteLine(data);
}
```

👉 Since modern versions of C#, `Main` itself can be asynchronous, letting you use `await` directly at the program's entry point

---

# 🔀 Running tasks in parallel: `Task.WhenAll`

If tasks don't depend on each other, awaiting them one at a time wastes time:

```csharp
// ❌ Sequential: adds up the time of each call
string data1 = await FetchDataAsync();
string data2 = await FetchDataAsync();

// ✅ Parallel: waits for all of them at the same time
Task<string> task1 = FetchDataAsync();
Task<string> task2 = FetchDataAsync();

string[] results = await Task.WhenAll(task1, task2);
```

👉 `Task.WhenAll` fires off all tasks at once and only continues once **all** of them finish — much faster when the calls are independent

---

# 🧨 The `.Result` and `.Wait()` trap

```csharp
string data = FetchDataAsync().Result; // ❌ can cause a deadlock
```

👉 Calling `.Result` or `.Wait()` from synchronous code to "force" an async task to finish can freeze the entire application, especially in UI apps or classic ASP.NET

**Practical rule:** use `await` from start to finish. If a method calls async code, it should be async too — this propagation is known as "async all the way"

---

# 🧰 Exception handling in async code

The good news: `try/catch` works normally with `await`.

```csharp
async Task ProcessAsync()
{
    try
    {
        await FetchDataAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
    }
}
```

👉 No special syntax needed — an exception thrown inside the `Task` is caught normally by the `catch` of whoever awaited it

---

# ⚠️ Common Mistakes

- Using `async void` outside of event handlers  
- Mixing `.Result`/`.Wait()` with async code, risking a deadlock  
- Awaiting independent tasks sequentially instead of using `Task.WhenAll`  
- Forgetting `await`, letting the method continue without waiting for the result (accidental "fire and forget")  
- Thinking `async` alone makes code faster — it makes code **non-blocking**, which is different from faster  

---

# 📌 Conclusion

- `async`/`await` frees up the thread while a slow operation is still running  
- `Task` and `Task<T>` represent asynchronous operations, with or without a return value  
- Avoid `async void`, except in event handlers  
- `Task.WhenAll` runs independent tasks in parallel  
- Mixing `.Result`/`.Wait()` with `await` is the most common recipe for deadlocks  

👉 With async/await in practice, you now know how to write C# code that truly scales in I/O-bound scenarios

---

# 🔥 Next Step

Now that you've applied asynchronous programming in practice, the next level is:

👉 **C# Fundamentals: Records and Pattern Matching (Modern C#)**

Here you'll get to know the language's most recent features, which make code even more concise and expressive.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
