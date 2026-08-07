# 🧠 C# Fundamentals: SemaphoreSlim and Asynchronous Concurrency

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- `lock` and `Monitor` for protecting synchronous critical sections  
- Deadlocks and how to avoid them by acquiring locks in a consistent order  

`lock` solves synchronization between threads — but try putting an `await` inside a `lock` block, and the compiler will complain. Asynchronous code needs a different tool to control concurrent access.

👉 **Let's learn `SemaphoreSlim`, the right tool for concurrency in `async` code**

---

# 💡 Why `lock` doesn't work with `async`

```csharp
private readonly object _lockObj = new();

public async Task ProcessAsync()
{
    lock (_lockObj)
    {
        await DoSomethingAsync(); // ❌ Compile error: can't use 'await' inside 'lock'
    }
}
```

👉 `lock` was designed to hold a lock from start to finish of a synchronous block, on the same thread. An `async` method can **switch threads** between one `await` and the next (remember the async/await post?) — and `Monitor`, behind `lock`, is tightly bound to the notion of "the thread holding the lock." Mixing the two breaks that guarantee

---

# 🎯 `SemaphoreSlim`: the asynchronous version of a lock

```csharp
private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);

public async Task ProcessAsync()
{
    await _semaphore.WaitAsync();
    try
    {
        await DoSomethingAsync(); // ✅ works perfectly in here
    }
    finally
    {
        _semaphore.Release();
    }
}
```

👉 **`SemaphoreSlim` = a counter that allows a limited number of threads (or tasks) to access a section at the same time**

With `SemaphoreSlim(1, 1)` (initial count 1, max 1), you get the async equivalent of a `lock`: only one task at a time passes `WaitAsync()`. The crucial difference is that `WaitAsync()` is `await`-compatible, without holding an OS thread hostage while it waits

---

# 🔢 Beyond 1: limiting real concurrency

```csharp
private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(3, 3); // at most 3 at once

public async Task<Product> GetProductAsync(int id)
{
    await _semaphore.WaitAsync();
    try
    {
        return await _httpClient.GetFromJsonAsync<Product>($"/products/{id}");
    }
    finally
    {
        _semaphore.Release();
    }
}
```

👉 Unlike `lock` (always 1 at a time), `SemaphoreSlim` can allow **N** simultaneous operations. This is exactly what you'd use to limit how many concurrent HTTP calls your code makes to an external API, avoiding overloading it or blowing past rate limits (remember post 53?)

---

# 🚦 Real use case: limiting parallel calls

```csharp
public async Task<List<Product>> GetSeveralAsync(IEnumerable<int> ids)
{
    var semaphore = new SemaphoreSlim(5); // at most 5 HTTP calls at once

    var tasks = ids.Select(async id =>
    {
        await semaphore.WaitAsync();
        try
        {
            return await GetProductAsync(id);
        }
        finally
        {
            semaphore.Release();
        }
    });

    return (await Task.WhenAll(tasks)).ToList();
}
```

👉 Without the semaphore, `Task.WhenAll` (from the async/await post) would fire off every call at once — for 1000 IDs, that's 1000 simultaneous HTTP calls, likely bringing down the remote service or blowing past a rate limit. The semaphore guarantees only 5 run at a time, naturally queuing the rest

---

# ⏱️ `WaitAsync` with timeout and cancellation

```csharp
public async Task<bool> TryProcessAsync(CancellationToken cancellationToken)
{
    if (await _semaphore.WaitAsync(TimeSpan.FromSeconds(5), cancellationToken))
    {
        try
        {
            await DoSomethingAsync();
            return true;
        }
        finally
        {
            _semaphore.Release();
        }
    }

    return false; // didn't get a slot in time
}
```

👉 Just like `Monitor.TryEnter`, `WaitAsync` accepts a timeout — and also accepts a `CancellationToken` (which you'll explore in depth in the next stretch of this track), letting you cancel the wait from outside

---

# ⚖️ `SemaphoreSlim` vs `lock`: when to use each

| | `lock` | `SemaphoreSlim` |
|---|---|---|
| Synchronous code | ✅ ideal | Works, but heavier than needed |
| `async` code | ❌ won't compile with `await` inside | ✅ ideal |
| Limiting to N concurrent (N > 1) | ❌ only allows 1 | ✅ supports any N |
| Overhead | Very low | Somewhat higher |

👉 **Practical rule: use `lock` for purely synchronous critical sections. Use `SemaphoreSlim` whenever you need `await` inside the protected section, or when you need to limit more than one simultaneous operation**

---

# ⚠️ Common Mistakes

- Trying to use `lock` around code with `await`, running straight into the compile error  
- Forgetting `Release()` in the `finally`, leaking semaphore "slots" until it locks everything up permanently  
- Creating a new `SemaphoreSlim` on every method call instead of reusing a shared instance — this protects nothing, since each call would have its own independent counter  
- Using `SemaphoreSlim(1, 1)` when a simple `lock` would do with less overhead, in code that never needs `await` in the critical section  

---

# 📌 Conclusion

- `lock` can't contain `await` — the thread switch breaks the guarantee behind `Monitor`  
- `SemaphoreSlim.WaitAsync()` is the asynchronous equivalent of acquiring a lock, `await`-compatible  
- With a count greater than 1, `SemaphoreSlim` limits concurrency to N simultaneous operations, not just 1  
- `WaitAsync` accepts a timeout and a `CancellationToken`, just like `Monitor.TryEnter`  

👉 With `lock`/`Monitor` for synchronous code and `SemaphoreSlim` for asynchronous code, you have the two fundamental tools for controlling concurrent access in any scenario — the perfect foundation to head back into the practical world of advanced asynchronous APIs

---

# 🔥 Next Step

Now that you've mastered both synchronous and asynchronous synchronization, the next level is:

👉 **C# Fundamentals: Unsafe Code and Pointers**

Here you'll learn to step outside C#'s managed safety when extreme performance demands direct memory control.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
