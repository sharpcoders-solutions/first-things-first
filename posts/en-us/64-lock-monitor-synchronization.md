# 🧠 C# Fundamentals: Lock, Monitor, and Synchronization

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to package and publish your own code as a NuGet package  
- Concurrent collections for data shared across threads (post 60)  

Concurrent collections solve one specific problem: data structures. But what do you do when the "critical section" that needs protecting isn't a collection, but an arbitrary sequence of operations in your own code? That's where `lock` and `Monitor` come in.

👉 **Let's dig into thread synchronization, with `lock` and `Monitor`**

---

# 💡 The problem: race conditions in regular code

```csharp
public class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        var currentBalance = Balance;  // read
        Thread.Sleep(1);               // simulates processing
        Balance = currentBalance + amount; // write
    }
}

var account = new BankAccount();
Parallel.For(0, 1000, i => account.Deposit(10));

Console.WriteLine(account.Balance); // ❌ rarely 10000
```

👉 **Race condition = when the final result depends on the exact order threads execute in, unpredictably**

Two threads can read the same `currentBalance`, both add `10`, and both write the same result — an entire deposit gets lost, with no visible error or exception

---

# 🔐 `lock`: the simplest way to protect a critical section

```csharp
public class BankAccount
{
    private readonly object _lockObj = new object();
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        lock (_lockObj)
        {
            var currentBalance = Balance;
            Thread.Sleep(1);
            Balance = currentBalance + amount;
        }
    }
}
```

👉 **`lock` = guarantees only one thread at a time executes the protected block; any other thread trying to enter waits until the first one leaves**

The object passed to `lock` (`_lockObj`) doesn't matter for its content — it only matters as a shared "padlock." By convention, use a private, dedicated `object` just for this purpose, never `this` or a public object that external code could also lock on

---

# ⚙️ What `lock` really is: syntactic sugar for `Monitor`

```csharp
// This:
lock (_lockObj)
{
    // critical section
}

// Is syntactic sugar for this:
Monitor.Enter(_lockObj);
try
{
    // critical section
}
finally
{
    Monitor.Exit(_lockObj);
}
```

👉 `Monitor.Enter`/`Monitor.Exit` is the low-level mechanism behind `lock`. The automatically generated `try`/`finally` is crucial: even if an exception happens inside the critical section, `Monitor.Exit` is always called, releasing the lock for other threads

---

# ⏱️ `Monitor.TryEnter`: avoiding waiting forever

```csharp
if (Monitor.TryEnter(_lockObj, TimeSpan.FromSeconds(2)))
{
    try
    {
        // critical section
    }
    finally
    {
        Monitor.Exit(_lockObj);
    }
}
else
{
    Console.WriteLine("Couldn't acquire the lock in time — moving on");
}
```

👉 Unlike `lock`, which waits indefinitely, `Monitor.TryEnter` with a timeout lets you give up after a while — useful to avoid a thread hanging forever waiting for a resource that never gets released

---

# 🔔 `Monitor.Wait` and `Monitor.Pulse`: coordinating threads with each other

```csharp
public class BoundedQueue<T>
{
    private readonly Queue<T> _items = new();
    private readonly int _capacity;
    private readonly object _lockObj = new();

    public BoundedQueue(int capacity) => _capacity = capacity;

    public void Add(T item)
    {
        lock (_lockObj)
        {
            while (_items.Count >= _capacity)
                Monitor.Wait(_lockObj); // releases the lock and waits to be notified

            _items.Enqueue(item);
            Monitor.PulseAll(_lockObj); // wakes up whoever was waiting
        }
    }

    public T Remove()
    {
        lock (_lockObj)
        {
            while (_items.Count == 0)
                Monitor.Wait(_lockObj);

            var item = _items.Dequeue();
            Monitor.PulseAll(_lockObj);
            return item;
        }
    }
}
```

👉 `Monitor.Wait` temporarily releases the lock and puts the thread to sleep until someone calls `Monitor.Pulse`/`PulseAll` — a classic pattern for manually implementing producer-consumer queues. In practice, you rarely need this: `Channel<T>` (which you'll see later in this track) solves the same problem with a much simpler API

---

# ⚠️ Deadlocks: when two locks wait on each other

```csharp
// Thread 1
lock (lockA)
{
    lock (lockB) { /* ... */ }
}

// Thread 2
lock (lockB)
{
    lock (lockA) { /* ... */ } // ❌ deadlock if both threads get here at the same time
}
```

👉 **Golden rule: always acquire multiple locks in the same order, everywhere in the code.** If Thread 1 always grabs `lockA` before `lockB`, and Thread 2 does the reverse, there's a window where each one holds a lock and waits for the other — forever

---

# ⚠️ Common Mistakes

- Using `lock (this)` or locking on a public object, letting external code lock the same object and cause unexpected deadlocks  
- Holding a lock during a slow operation (network call, I/O), unnecessarily blocking other threads  
- Forgetting that `lock` only protects code that **also** uses `lock` on the same object — there's no automatic protection against direct access without a lock elsewhere  
- Acquiring multiple locks in different orders in different parts of the code, creating deadlock risk  

---

# 📌 Conclusion

- `lock` is syntactic sugar for `Monitor.Enter`/`Monitor.Exit` with an automatic `try`/`finally`  
- Only one thread at a time executes the code inside a `lock` on the same object  
- `Monitor.TryEnter` with a timeout avoids waiting indefinitely for a lock  
- Deadlocks happen when multiple locks are acquired in inconsistent orders across threads  

👉 `lock` works well for synchronous code — but what do you do when the critical section involves an `async` operation? `lock` doesn't allow `await` inside it, and that's where the next tool comes in

---

# 🔥 Next Step

Now that you know how to protect synchronous critical sections, the next level is:

👉 **C# Fundamentals: SemaphoreSlim and Asynchronous Concurrency**

Here you'll learn to control concurrent access in `async` code, where `lock` simply doesn't work.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
