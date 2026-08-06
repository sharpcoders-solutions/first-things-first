# 🧠 C# Fundamentals: ArrayPool and Object Pooling

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Native AOT for eliminating the runtime JIT  
- Span and Memory for accessing memory without extra allocations (post 46)  

Allocating a new array every time you need a temporary buffer seems harmless, but in high-frequency loops it constantly pressures the Garbage Collector. ArrayPool and Object Pooling fix this by reusing memory.

👉 **Let's learn ArrayPool and Object Pooling**

---

# 💡 The problem: excessive allocation

```csharp
// ❌ Allocates a new array on every call
public byte[] ProcessData(int size)
{
    var buffer = new byte[size]; // new allocation, every time
    // ... process the buffer
    return buffer;
} // the array becomes garbage as soon as it goes out of scope
```

👉 Called a thousand times per second, this generates constant pressure on the Garbage Collector — remember the processes and memory post? Every collection pauses the application, even if briefly

---

# 🏗️ ArrayPool: reusing arrays

```csharp
public void ProcessData(int size)
{
    byte[] buffer = ArrayPool<byte>.Shared.Rent(size);

    try
    {
        // ... process the buffer normally
    }
    finally
    {
        ArrayPool<byte>.Shared.Return(buffer);
    }
}
```

👉 `Rent` borrows an array from the pool (reusing an already-allocated one, if available), and `Return` gives it back for future reuse — no new allocation on most calls

---

# ⚠️ Careful: the borrowed array may be larger

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(100);

Console.WriteLine(buffer.Length); // could be 128, not exactly 100!
```

👉 `Rent` guarantees an array with **at least** the requested size, but may return a larger one (powers of 2, for example) — always use the size you requested, not `buffer.Length`, when processing the data

---

# 🎯 Object Pooling: reusing entire objects

```csharp
public class OrderPoolPolicy : PooledObjectPolicy<OrderProcessor>
{
    public override OrderProcessor Create() => new OrderProcessor();

    public override bool Return(OrderProcessor obj)
    {
        obj.Reset(); // clean up state before returning to the pool
        return true;
    }
}
```

```csharp
var pool = new DefaultObjectPool<OrderProcessor>(new OrderPoolPolicy());

var processor = pool.Get();
try
{
    processor.Process(order);
}
finally
{
    pool.Return(processor);
}
```

👉 Unlike ArrayPool (which is only for arrays), `ObjectPool<T>` (from the `Microsoft.Extensions.ObjectPool` package) reuses any type of object that's expensive to create — the key is always resetting the state on `Return`, so the next consumer doesn't inherit stale data

---

# 🔍 Where ASP.NET Core already uses this

👉 ASP.NET Core itself uses Object Pooling internally for `StringBuilder` (remember post 46?) and HTTP response buffers — each request doesn't allocate these objects from scratch, they're borrowed and returned to the pool automatically

---

# ⚠️ Common Mistakes

- Forgetting `Return` (or not using `try/finally`), making the object never go back to the pool and negating the entire benefit  
- Not resetting the object's state before returning it, leaking data from one consumer to the next  
- Using pooling for objects that are cheap to create, adding complexity with no real performance gain  
- Assuming the borrowed array has exactly the requested size, causing subtle bugs  

---

# 📌 Conclusion

- Repeated allocations in high-frequency loops pressure the Garbage Collector  
- `ArrayPool<T>.Shared` reuses arrays via `Rent`/`Return`  
- `ObjectPool<T>` generalizes that pattern to any object that's expensive to create  
- ASP.NET Core already uses this mechanism internally for buffers and `StringBuilder`  

👉 With ArrayPool and Object Pooling, you reduce GC pressure exactly at the points where repeated allocation actually costs the most

---

# 🔥 Next Step

Now that you know how to reduce allocations by reusing memory, the next level is:

👉 **C# Fundamentals: Channels**

Here you'll learn to coordinate producers and consumers within the same process, without needing an external queue like RabbitMQ.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
