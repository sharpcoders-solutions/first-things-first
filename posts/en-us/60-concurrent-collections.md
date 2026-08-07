# 🧠 C# Fundamentals: Concurrent Collections (ConcurrentDictionary and Friends)

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Integration testing with an in-memory database  
- Static abstract interface members and the rest of C#'s advanced type features  

You've used `Dictionary<TKey, TValue>` and `List<T>` hundreds of times. But what happens when two threads try to write to them at the same time? The answer isn't pretty — and it's exactly the problem concurrent collections solve.

👉 **Let's learn about `ConcurrentDictionary` and .NET's other thread-safe collections**

---

# 💡 The problem: regular collections aren't thread-safe

```csharp
var counter = new Dictionary<string, int>();

// Two threads incrementing the same counter at the same time
Parallel.For(0, 1000, i =>
{
    if (counter.ContainsKey("total"))
        counter["total"]++;
    else
        counter["total"] = 1;
});

Console.WriteLine(counter["total"]); // ❌ unpredictable result, may even throw
```

👉 `Dictionary<TKey, TValue>` was **not** designed for multiple threads writing simultaneously. The result can be a number smaller than 1000 (lost increments), or even an `InvalidOperationException` ("Collection was modified") if two threads touch the internal structure at the same time

---

# 🔐 The naive solution: lock everything

```csharp
var counter = new Dictionary<string, int>();
var lockObj = new object();

Parallel.For(0, 1000, i =>
{
    lock (lockObj)
    {
        if (counter.ContainsKey("total"))
            counter["total"]++;
        else
            counter["total"] = 1;
    }
});
```

👉 This works, but serializes **all** access to the collection — even threads that just want to read end up waiting in line. In high-concurrency scenarios, this becomes a bottleneck

---

# 🎯 `ConcurrentDictionary<TKey, TValue>`: thread-safe by default

```csharp
var counter = new ConcurrentDictionary<string, int>();

Parallel.For(0, 1000, i =>
{
    counter.AddOrUpdate("total", 1, (key, currentValue) => currentValue + 1);
});

Console.WriteLine(counter["total"]); // ✅ always 1000
```

👉 `AddOrUpdate` is **atomic**: it either adds the initial value, or applies the update function — leaving no room for two threads to step on each other along the way. Internally, `ConcurrentDictionary` uses a combination of fine-grained locks (per data partition) instead of one single global lock, which allows for far more real parallelism than the `lock`-based solution

---

# 🔧 The most commonly used atomic methods

```csharp
var cache = new ConcurrentDictionary<int, string>();

// GetOrAdd: looks up, or creates and adds if missing — all atomically
var value = cache.GetOrAdd(1, id => FetchFromDatabase(id));

// TryAdd: adds only if the key doesn't already exist
bool added = cache.TryAdd(2, "new value");

// TryUpdate: updates only if the current value matches the expected one
bool updated = cache.TryUpdate(1, "new value", "old value");

// TryRemove: removes and returns the removed value
bool removed = cache.TryRemove(1, out var removedValue);
```

👉 Each of these methods solves a classic concurrency problem: "check, then act" (`ContainsKey` followed by `this[key] = value`) has a window where another thread can intervene between the two operations. The atomic methods eliminate that window

---

# 📦 Other concurrent collections in the `System.Collections.Concurrent` namespace

```csharp
// ConcurrentBag<T>: unordered collection, optimized for multiple producers/consumers
var bag = new ConcurrentBag<int>();
Parallel.For(0, 100, i => bag.Add(i));

// ConcurrentQueue<T>: thread-safe FIFO queue
var queue = new ConcurrentQueue<string>();
queue.Enqueue("task 1");
queue.TryDequeue(out var nextTask);

// ConcurrentStack<T>: thread-safe LIFO stack
var stack = new ConcurrentStack<int>();
stack.Push(1);
stack.TryPop(out var top);
```

👉 Each collection solves a different access pattern — `ConcurrentQueue<T>` is the natural structure for a work queue processed by multiple workers, something you'd pair with the `BackgroundService` from the Hangfire post

---

# ⚖️ When a `lock` still makes sense

```csharp
lock (lockObj)
{
    var sourceBalance = accounts[sourceId] - amount;
    var destinationBalance = accounts[destinationId] + amount;

    if (sourceBalance < 0)
        throw new InvalidOperationException("Insufficient balance");

    accounts[sourceId] = sourceBalance;
    accounts[destinationId] = destinationBalance;
}
```

👉 **Concurrent collections guarantee that each individual operation is atomic — not that a sequence of multiple related operations is atomic as a whole.** When you need multiple operations to happen as one indivisible unit (like transferring a balance between two accounts), you still need an explicit `lock` wrapping the entire sequence

---

# ⚠️ Common Mistakes

- Using a regular `Dictionary<TKey, TValue>` in code accessed by multiple threads, assuming "usually works" is the same as "is safe"  
- Combining `ContainsKey` followed by indexing (`dict[key]`) even on concurrent collections — use `TryGetValue` to avoid the window between the two calls  
- Assuming concurrent collections make any sequence of operations atomic, when only the individual operation is guaranteed  
- Using a `lock` over an entire collection when `ConcurrentDictionary` would solve the same problem with better performance  

---

# 📌 Conclusion

- `Dictionary<TKey, TValue>` and other regular collections aren't safe for multiple threads writing simultaneously  
- `ConcurrentDictionary` uses fine-grained internal locks, allowing more real parallelism than a manual `lock` around everything  
- Methods like `GetOrAdd`, `TryUpdate`, and `TryRemove` are atomic, eliminating race windows between "check" and "act"  
- `ConcurrentBag`, `ConcurrentQueue`, and `ConcurrentStack` cover other concurrent access patterns  
- Sequences of multiple related operations may still need an explicit `lock`, even with concurrent collections  

👉 Concurrent collections solve the problem of data structures shared across threads — but testing that kind of code demands its own discipline, which is where the next step in your software quality journey comes in

---

# 🔥 Next Step

Now that you know how to handle data shared across threads, the next level is:

👉 **C# Fundamentals: Mutation Testing**

Here you'll learn to test whether your own tests actually catch bugs, not just whether they pass.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
