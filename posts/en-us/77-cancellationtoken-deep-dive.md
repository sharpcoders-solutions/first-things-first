# 🧠 C# Fundamentals: CancellationToken in Depth

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- The Options Pattern for strongly-typed configuration  
- `async`/`await` since the earliest posts on asynchronous programming  

You've already seen `CancellationToken` as a parameter in async methods throughout this track, almost always passed along without much explanation. Time to truly understand how cooperative cancellation works in C#.

👉 **Let's dig into `CancellationToken`**

---

# 💡 The problem: how do you interrupt an operation in progress?

```csharp
public async Task<List<Product>> GetProductsAsync()
{
    // If the user closes the page in the middle of this search, how do we let the code know?
    return await _context.Products.ToListAsync();
}
```

👉 Threads can't simply be "killed" safely from outside — that would leave resources in an inconsistent state (open files, pending transactions). C# solves this with **cooperative cancellation**: whoever is doing the work needs to periodically check whether it's been asked to stop, and stop voluntarily

---

# 🏗️ `CancellationTokenSource` and `CancellationToken`

```csharp
using var cts = new CancellationTokenSource();

var task = GetProductsAsync(cts.Token);

// Somewhere else in the code, maybe in response to a "cancel" click
cts.Cancel();
```

👉 **`CancellationTokenSource` = the one with the power to cancel. `CancellationToken` = the read-only "signal" passed along to whoever does the work**

You never create a `CancellationToken` directly — it always comes from a `CancellationTokenSource`, which decides **when** to trigger the cancellation

---

# 🔍 Checking for cancellation inside the code

```csharp
public async Task ProcessBatchAsync(List<Order> orders, CancellationToken cancellationToken)
{
    foreach (var order in orders)
    {
        cancellationToken.ThrowIfCancellationRequested(); // throws OperationCanceledException if cancelled

        await ProcessOrderAsync(order, cancellationToken);
    }
}
```

👉 `ThrowIfCancellationRequested()` is the most common pattern: checks the token and, if cancellation was requested, immediately throws `OperationCanceledException` — cleanly and predictably stopping the loop

---

# ⏱️ Cancelling via timeout

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));

try
{
    await GetProductsAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("The operation exceeded the 10-second time limit");
}
```

👉 Passing a `TimeSpan` directly into `CancellationTokenSource`'s constructor schedules automatic cancellation after that time — a simple way to implement a timeout without manually managing a `Timer`

---

# 🔗 Combining multiple tokens

```csharp
public async Task RunWithTimeoutAsync(CancellationToken externalToken)
{
    using var ctsTimeout = new CancellationTokenSource(TimeSpan.FromSeconds(30));
    using var ctsCombined = CancellationTokenSource.CreateLinkedTokenSource(
        externalToken, ctsTimeout.Token);

    await DoWorkAsync(ctsCombined.Token);
}
```

👉 `CreateLinkedTokenSource` combines multiple tokens into one — the operation cancels if **any** of the original tokens gets cancelled. This is essential when you receive a token from outside (from ASP.NET Core, for example, when the HTTP client disconnects) and still want to apply your own internal timeout

---

# 🌐 Where tokens actually come from: ASP.NET Core

```csharp
[HttpGet]
public async Task<IActionResult> GetOrders(CancellationToken cancellationToken)
{
    var orders = await _context.Orders.ToListAsync(cancellationToken);
    return Ok(orders);
}
```

👉 ASP.NET Core automatically injects a `CancellationToken` tied to the HTTP connection — if the client closes the browser tab or cancels the request, this token gets cancelled, and passing it along to `ToListAsync` stops the database query immediately, instead of wasting work on a response nobody will receive

---

# ⚠️ Common Mistakes

- Accepting a `CancellationToken` parameter, but never passing it along to internal async calls — the parameter becomes decoration, with no real effect  
- Catching `OperationCanceledException` and treating it as a generic error, when it represents an intentional cancellation, not a failure  
- Ignoring the `CancellationToken` injected by ASP.NET Core, continuing to process a request the client has already abandoned  
- Not using `using` on the `CancellationTokenSource`, leaking the internal `Timer` when a timeout is configured  

---

# 📌 Conclusion

- Cancellation in C# is cooperative — whoever does the work needs to check the token periodically  
- `CancellationTokenSource` triggers cancellation; `CancellationToken` is the read-only signal that gets propagated  
- `ThrowIfCancellationRequested()` stops execution by throwing `OperationCanceledException`  
- `CreateLinkedTokenSource` combines multiple tokens, cancelling if any one of them is cancelled  
- ASP.NET Core automatically injects a token tied to the HTTP request's lifecycle  

👉 With cancellation under control, the next step is seeing how to consume asynchronous data sequences — combining everything you know about iterators with the `async` world

---

# 🔥 Next Step

Now that you've mastered cooperative cancellation, the next level is:

👉 **C# Fundamentals: IAsyncEnumerable and Async Streams**

Here you'll learn to combine `yield return` with `async`, creating sequences that produce asynchronous values, one at a time.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
