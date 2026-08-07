# 🧠 C# Fundamentals: IAsyncEnumerable and Async Streams

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- `CancellationToken` for cooperative cancellation of asynchronous operations  
- `yield return` and custom iterators (post 58)  

You already know how to use `yield return` to generate lazy sequences, and you already know how to use `async`/`await` for asynchronous operations. But what do you do when you need both at once — a sequence generated bit by bit, where each item requires an asynchronous operation?

👉 **Let's learn `IAsyncEnumerable<T>` and async streams**

---

# 💡 The problem: `IEnumerable<T>` isn't compatible with `async`

```csharp
public IEnumerable<Product> GetPagedProducts()
{
    int page = 0;
    while (true)
    {
        var products = _repository.GetPage(page); // ❌ what if this needed to be await?
        if (!products.Any()) yield break;

        foreach (var product in products)
            yield return product;

        page++;
    }
}
```

👉 Remember the iterators post (55)? `yield return` inside an `IEnumerable<T>` method doesn't allow `await` along the way — the state machine generated for synchronous iterators is different from the one generated for `async` methods

---

# 🎯 `IAsyncEnumerable<T>`: combining both worlds

```csharp
public async IAsyncEnumerable<Product> GetPagedProductsAsync()
{
    int page = 0;
    while (true)
    {
        var products = await _repository.GetPageAsync(page); // ✅ await works here
        if (!products.Any()) yield break;

        foreach (var product in products)
            yield return product;

        page++;
    }
}
```

👉 **`async IAsyncEnumerable<T>` = a method that combines `yield return` (lazy generation) with `await` (asynchronous operations), producing values one at a time, asynchronously**

The compiler generates a state machine that supports both behaviors simultaneously — something impossible to express before this feature existed

---

# 🔄 Consuming with `await foreach`

```csharp
await foreach (var product in GetPagedProductsAsync())
{
    Console.WriteLine(product.Name);
}
```

👉 **`await foreach` = the asynchronous version of `foreach`, built specifically to consume `IAsyncEnumerable<T>`**

Each iteration awaits the next page being fetched before continuing — without blocking the thread while that happens, exactly like any other `await` you already use

---

# 🚫 Cancellation in async streams

```csharp
public async IAsyncEnumerable<Product> GetPagedProductsAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    int page = 0;
    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        var products = await _repository.GetPageAsync(page, cancellationToken);
        if (!products.Any()) yield break;

        foreach (var product in products)
            yield return product;

        page++;
    }
}

// When consuming:
await foreach (var product in GetPagedProductsAsync(cts.Token))
{
    // ...
}
```

👉 Remember the previous post on `CancellationToken`? The `[EnumeratorCancellation]` attribute is required because, without it, the compiler doesn't automatically know how to connect the token passed to `await foreach` to the generator method's parameter

---

# 🌐 Real use case: streaming API results

```csharp
[HttpGet("orders/stream")]
public async IAsyncEnumerable<OrderDto> StreamOrdersAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken)
{
    await foreach (var order in _context.Orders.AsAsyncEnumerable().WithCancellation(cancellationToken))
    {
        yield return new OrderDto(order.Id, order.Amount);
    }
}
```

👉 ASP.NET Core supports `IAsyncEnumerable<T>` as a direct endpoint return type — results are serialized and sent to the client **as they're generated**, instead of waiting for the entire query to finish and materializing everything into a list before responding. This reduces time-to-first-byte, especially valuable for large data volumes

---

# ⚖️ `IAsyncEnumerable<T>` vs `Task<List<T>>`

```csharp
// ❌ Waits for EVERYTHING to finish before returning anything
public async Task<List<Product>> GetAllAsync()
{
    var all = new List<Product>();
    int page = 0;
    while (true)
    {
        var products = await _repository.GetPageAsync(page);
        if (!products.Any()) break;
        all.AddRange(products);
        page++;
    }
    return all;
}

// ✅ Delivers each item as soon as it's available
public async IAsyncEnumerable<Product> GetAllStreamAsync() { /* ... */ }
```

👉 **Practical rule: use `Task<List<T>>` when you need all results at once to process them. Use `IAsyncEnumerable<T>` when the consumer can (and should) start processing items before all of them are available** — the same lazy evaluation philosophy from the `yield return` post, now applied to the asynchronous world

---

# ⚠️ Common Mistakes

- Materializing an entire `IAsyncEnumerable<T>` with `.ToListAsync()` too early, losing the streaming benefit it offers  
- Forgetting the `[EnumeratorCancellation]` attribute, causing cancellation to not propagate correctly into the generator  
- Using `IAsyncEnumerable<T>` for small sequences that are already fully available, where `Task<List<T>>` would be simpler and equally efficient  
- Mixing a regular `foreach` (without `await`) with an `IAsyncEnumerable<T>`, causing a compile error due to type incompatibility  

---

# 📌 Conclusion

- `IAsyncEnumerable<T>` combines `yield return` with `await`, something impossible with regular `IEnumerable<T>`  
- `await foreach` consumes async streams without blocking the thread between items  
- `[EnumeratorCancellation]` connects the consuming `CancellationToken` to the generator  
- ASP.NET Core supports `IAsyncEnumerable<T>` as a direct endpoint return, truly streaming the response  

👉 With async streams mastered, the next step is understanding a specific `Task` optimization — when even allocating a `Task` object can be too costly for extremely performance-sensitive code

---

# 🔥 Next Step

Now that you know how to combine lazy iteration with asynchronous code, the next level is:

👉 **C# Fundamentals: Task vs ValueTask**

Here you'll learn when to swap `Task<T>` for `ValueTask<T>` to eliminate unnecessary allocations in high-frequency asynchronous code.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
