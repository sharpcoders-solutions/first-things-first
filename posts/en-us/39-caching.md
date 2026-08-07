# 🧠 C# Fundamentals: Caching in C# (In-Memory and Distributed with Redis)

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Health checks and monitoring  
- Data persistence with EF Core  

Every database query has a cost. If the same information is requested a hundred times a minute and rarely changes, fetching it from the database every single time is pure waste.

👉 **That's exactly what caching is for**

---

# 💡 What is caching?

👉 **Caching = temporarily storing an already-computed result, so you don't have to redo the work**

```csharp
// Without cache: every request hits the database
var products = await _context.Products.ToListAsync();

// With cache: it only hits the database the first time
```

The gain is direct: less database load, faster responses. The cost: cached data can become **stale** for a while — caching is always a trade-off between performance and freshness.

---

# 🏗️ `IMemoryCache`: in-memory caching

```csharp
builder.Services.AddMemoryCache();
```

```csharp
public class ProductsController : ControllerBase
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;

    public ProductsController(IMemoryCache cache, IProductRepository repository)
    {
        _cache = cache;
        _repository = repository;
    }

    [HttpGet]
    public IActionResult GetAll()
    {
        var products = _cache.GetOrCreate("all-products", entry =>
        {
            entry.SetAbsoluteExpiration(TimeSpan.FromMinutes(5));
            return _repository.ListAll();
        });

        return Ok(products);
    }
}
```

👉 `GetOrCreate` checks whether the key already exists in the cache; if not, it runs the function and stores the result for the given duration. On subsequent calls within those 5 minutes, the repository **isn't queried at all**

## 🔹 When `IMemoryCache` is enough

👉 Great for applications with a single instance — the cache lives in the process's memory, so each instance would have its own separate cache

---

# 🌐 Distributed cache: the multi-instance problem

When your API runs across several instances (multiple containers or machines behind a load balancer), `IMemoryCache` becomes a problem: each instance has its own cache, isolated from the others.

```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
});
```

```csharp
public class ProductsController : ControllerBase
{
    private readonly IDistributedCache _cache;

    public ProductsController(IDistributedCache cache)
    {
        _cache = cache;
    }

    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var cached = await _cache.GetStringAsync("all-products");

        if (cached is not null)
        {
            return Ok(JsonSerializer.Deserialize<List<Product>>(cached));
        }

        var products = await _repository.ListAllAsync();

        await _cache.SetStringAsync(
            "all-products",
            JsonSerializer.Serialize(products),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            });

        return Ok(products);
    }
}
```

👉 `IDistributedCache` stores data in a shared external service (Redis) — every instance of your API reads from and writes to the **same** cache, solving the isolated-memory problem

---

# ⏳ Expiration strategies

```csharp
// Expires at a fixed time, regardless of usage
new DistributedCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
};

// Only expires if it goes 2 minutes without being accessed
new DistributedCacheEntryOptions
{
    SlidingExpiration = TimeSpan.FromMinutes(2)
};
```

## 🔹 Which one to choose

- **Absolute** → data that changes on predictable intervals (e.g., a rate updated hourly)  
- **Sliding** → data accessed frequently, but without a fixed time it changes  

👉 Combining both is common: absolute expiration as a hard cap, sliding to keep renewing while the data is still being used

---

# ♻️ Invalidating the cache

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateProductRequest request)
{
    var product = new Product(request.Name, request.Price);
    await _repository.AddAsync(product);

    await _cache.RemoveAsync("all-products"); // invalidates the stale cache

    return CreatedAtAction(nameof(GetAll), product);
}
```

👉 Whenever the underlying data changes, the corresponding cache needs to be invalidated — forgetting this is the most common cause of cache-related bugs ("why isn't the data updating?")

---

# ⚠️ Common Mistakes

- Using `IMemoryCache` in an application with multiple instances, causing inconsistent data between them  
- Accidentally caching sensitive or user-specific data under a shared key  
- Forgetting to invalidate the cache after a write, serving stale data indefinitely  
- Caching everything, even data that changes every second, with no real performance gain  

---

# 📌 Conclusion

- Caching trades freshness for performance by storing already-computed results  
- `IMemoryCache` works well for a single instance; `IDistributedCache` (Redis) solves the multi-instance case  
- Absolute and sliding expiration control how long data stays cached  
- Invalidating the cache after a write is essential to avoid serving stale data  

👉 With caching applied well, your application responds faster and reduces database load, without compromising data reliability

---

# 🔥 Next Step

Now that you know how to speed up your application with caching, the next level is:

👉 **C# Fundamentals: Resilience with Polly**

Here you'll learn to make your application gracefully handle temporary failures in external dependencies.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
