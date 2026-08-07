# 🧠 C# Fundamentals: Custom Awaiters and the Awaitable Pattern

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- `Task<T>` vs `ValueTask<T>` and the cost of allocations in asynchronous code  
- Operator overloading and how to teach the compiler new behaviors (post 59)  

You've used `await` since the earliest posts on asynchronous programming, always on top of `Task` or `Task<T>`. But did you know `await` isn't magic exclusive to `Task`? Any type can be "awaitable," as long as it follows a specific pattern — it doesn't even need to implement any interface.

👉 **Let's understand what actually makes something awaitable with `await`**

---

# 💡 The secret: `await` doesn't require an interface, it requires a pattern

```csharp
// await X works if X has a GetAwaiter() method
// that returns something with IsCompleted, GetResult(), and implements INotifyCompletion
```

👉 **Unlike what a lot of people assume, `await` doesn't require the type to implement `Task` or any specific interface — the compiler looks, at compile time, for a method pattern known as the "awaitable pattern"**

This is called **structural duck typing**: if the type "looks" awaitable (has the right methods), the compiler accepts it, even without formal inheritance or an interface — the same spirit of flexibility you already saw in the operator overloading post, where the compiler recognizes a signature pattern, not a specific type

---

# 🏗️ Building an awaiter from scratch

```csharp
public class CustomTask
{
    public Awaiter GetAwaiter() => new Awaiter(this);

    public class Awaiter : INotifyCompletion
    {
        private readonly CustomTask _task;

        public Awaiter(CustomTask task) => _task = task;

        public bool IsCompleted => /* checks whether it's already done */ true;

        public void GetResult() { /* returns the result, or throws if it failed */ }

        public void OnCompleted(Action continuation) { /* schedules the continuation */ }
    }
}
```

👉 The three required members are: **`IsCompleted`** (a `bool` property, indicates whether it's already done), **`GetResult()`** (returns the result, or `void`), and **`OnCompleted(Action)`** (from `INotifyCompletion`, schedules what to run when done). `GetAwaiter()` is the method that connects your type to this awaiter object

---

# 🎯 A practical example: awaiting a `SemaphoreSlim` more naturally

```csharp
public static class SemaphoreSlimExtensions
{
    public static SemaphoreSlimAwaiter GetAwaiter(this SemaphoreSlim semaphore) =>
        new SemaphoreSlimAwaiter(semaphore);
}

public readonly struct SemaphoreSlimAwaiter : INotifyCompletion
{
    private readonly SemaphoreSlim _semaphore;

    public SemaphoreSlimAwaiter(SemaphoreSlim semaphore) => _semaphore = semaphore;

    public bool IsCompleted => false;

    public void GetResult() { }

    public void OnCompleted(Action continuation) =>
        _semaphore.WaitAsync().ContinueWith(_ => continuation());
}

// Now this works:
await mySemaphore; // instead of: await mySemaphore.WaitAsync();
```

👉 Remember the extension methods post? `GetAwaiter` can be added as an extension method to **any** type, even one you don't control — making `SemaphoreSlim` (from the asynchronous concurrency post) directly awaitable, without needing to explicitly call `.WaitAsync()`

---

# ⚙️ `Task.GetAwaiter()` under the hood

```csharp
await myTask;

// Is, in practice, roughly syntactic sugar for:
var awaiter = myTask.GetAwaiter();
if (!awaiter.IsCompleted)
{
    // pauses execution here, registers the continuation, and resumes when ready
}
var result = awaiter.GetResult();
```

👉 `Task<T>` and `Task` simply implement this same pattern internally — there's no special magic built into the language just for `Task`. All the `async`/`await` machinery you've used since the earliest posts is built entirely on top of this three-member pattern

---

# 🚀 `ConfigureAwait(false)`: another custom awaiter you already use

```csharp
await _httpClient.GetAsync(url).ConfigureAwait(false);
```

👉 `ConfigureAwait(false)` returns a `ConfiguredTaskAwaitable`, a different type with its own awaiter, which controls whether the continuation should return to the original synchronization context or not. It's another example of the awaitable pattern being used inside .NET itself to vary `await`'s behavior, without needing a new keyword

---

# 🎯 When you'd actually create a custom awaiter

👉 **In everyday practice, you'll almost never write an awaiter from scratch — but understanding the pattern explains why `await` works in places that seem "special"**

Rare cases where it makes sense:

- Building a library API where `await myObject` reads more naturally than `await myObject.SomeMethodAsync()`  
- Integrating with low-level APIs (like game loops or custom frameworks) that need continuation behavior different from the default  
- Understanding third-party libraries that expose custom "awaitable" types, like certain UI APIs  

---

# ⚠️ Common Mistakes

- Thinking `await` only works with `Task`, without realizing it's a structural pattern, not a type restriction  
- Creating custom awaiters for simple problems, when `Task`/`ValueTask` would already solve it with far less code  
- Forgetting to implement `INotifyCompletion` correctly, breaking `await`'s continuation mechanism  
- Confusing `GetAwaiter()` (which enables `await`) with `GetEnumerator()` (which enables `foreach`, from post 58) — similar patterns, but for completely different purposes  

---

# 📌 Conclusion

- `await` works on top of a structural pattern (`GetAwaiter`, `IsCompleted`, `GetResult`, `OnCompleted`), not a required interface  
- Any type can become awaitable, even via an extension method on a type you don't control  
- `Task<T>` and `ConfigureAwait(false)` are examples of .NET itself using this same pattern internally  
- Creating custom awaiters is rare in practice, but understanding the pattern explains the inner workings of everything you already use  

👉 With CancellationToken, IAsyncEnumerable, Task vs ValueTask, and now the awaitable pattern, you've completed the most advanced picture of asynchronous programming in C# — the perfect foundation to head back into the world of real-time communication

---

# 🔥 Next Step

Now that you understand the full inner workings of `await`, the next level is:

👉 **C# Fundamentals: SignalR**

Here you'll learn real-time communication between server and client, beyond the traditional request/response model.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
