# 🧠 C# Fundamentals: WeakReference, Finalizers, and the Garbage Collector

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- P/Invoke and how native memory is never tracked by the GC  
- The Garbage Collector, mentioned dozens of times throughout this track, but never explained in depth  

You know the GC frees memory automatically. But **how** does it decide what to collect, **when** does it release native memory through finalizers, and how do you keep a reference to an object without preventing it from being collected? Time to truly understand the GC.

👉 **Let's understand the Garbage Collector, generations, finalizers, and `WeakReference`**

---

# 💡 How the GC decides what to collect

👉 **An object becomes eligible for collection when there's no longer any "strong" reference reachable from the program's roots** — active local variables, static fields, objects still in use

```csharp
var product = new Product("Laptop");
// while "product" exists and is used, the object is NOT collected

product = null;
// now there are no more strong references — the object becomes eligible for collection
// (but the GC still decides WHEN to actually run)
```

👉 The GC doesn't collect the instant the last reference disappears — it runs periodically, based on memory pressure, not immediately after every `= null`

---

# 🗂️ Generations: why the GC doesn't sweep everything every time

```
Generation 0: freshly created objects, collected very frequently (fast, cheap)
Generation 1: objects that survived a Generation 0 collection (intermediate)
Generation 2: long-lived objects, collected rarely (more expensive, rarer)
```

👉 **The generational hypothesis: most objects die young.** A method's local variables, temporary objects — most never survive past a single Generation 0 collection. Instead of sweeping the **entire** heap on every collection, the GC focuses first on Generation 0, which is fast and covers most of the real garbage

```csharp
public void ProcessOrder()
{
    var dto = new OrderDto(); // probably dies in Generation 0
    // ... processes ...
} // dto goes out of scope, becomes garbage quickly
```

An object that survives a Generation 0 collection gets "promoted" to Generation 1, and so on — the logic being that if it already survived once, it's more likely to stay alive longer

---

# 🧹 Finalizers: the last resort for cleanup

```csharp
public class NativeResource
{
    private IntPtr _nativeHandle;

    public NativeResource()
    {
        _nativeHandle = AllocateNativeResource();
    }

    ~NativeResource() // finalizer — C++ destructor syntax
    {
        FreeNativeResource(_nativeHandle);
    }
}
```

👉 **Finalizer (`~ClassName()`) = a special method the GC calls before permanently collecting an object, giving it one last chance to release unmanaged resources**

Remember the `IDisposable` post? `Dispose()` is the **deterministic** way to release resources (you call it explicitly, `using` handles it). The finalizer is a **safety net**, called by the GC if `Dispose()` was never called — but at an unpredictable time, and with a real performance cost

---

# ⚠️ Why finalizers are expensive

```csharp
// An object with a finalizer needs TWO collections to be fully freed
// 1st collection: the GC detects it's garbage, but needs to run the finalizer first (special queue)
// 2nd collection: only after the finalizer runs is the memory actually freed
```

👉 An object with a finalizer isn't collected immediately, even with no references — it's placed in a special finalization queue, processed by a dedicated thread, and only truly freed on the **next** collection. That's why the recommended pattern (which you already saw in the `IDisposable` post) is implementing `Dispose()` as the main path and using `GC.SuppressFinalize(this)` to tell the GC "no need to run the finalizer, I already cleaned everything up manually"

---

# 🔗 `WeakReference<T>`: referencing without preventing collection

```csharp
public class Cache
{
    private readonly Dictionary<int, WeakReference<Product>> _cache = new();

    public void Add(int id, Product product)
    {
        _cache[id] = new WeakReference<Product>(product);
    }

    public Product? GetOrNull(int id)
    {
        if (_cache.TryGetValue(id, out var weakReference) &&
            weakReference.TryGetTarget(out var product))
        {
            return product; // still alive
        }

        return null; // already collected
    }
}
```

👉 **`WeakReference<T>` = a reference that points to an object without counting as a "strong" reference — the GC can collect the object normally, even while the `WeakReference` still exists**

This is perfect for caches: you want to reuse an object if it's still in memory, but you **don't** want the cache to be the reason an object never gets collected, competing with the application's real memory pressure

---

# ⚠️ Common Mistakes

- Implementing a finalizer without a real need, adding performance cost to objects that hold no unmanaged resources at all  
- Forgetting `GC.SuppressFinalize(this)` in `Dispose()`, making the object still go through the finalization queue unnecessarily  
- Using `WeakReference<T>` for everything "just in case," when in most cases a regular strong reference is exactly what you want  
- Manually calling `GC.Collect()` in production code, forcing an expensive full collection the GC would have handled more efficiently on its own  

---

# 📌 Conclusion

- An object becomes eligible for collection when there are no more reachable strong references  
- Generations (0, 1, 2) optimize the GC based on the hypothesis that most objects die young  
- Finalizers are a safety net for unmanaged resources, but cost an extra collection  
- `WeakReference<T>` references an object without preventing its collection — ideal for caches  

👉 Understanding generations sets up the next step: how .NET offers two completely different GC modes, optimized for opposite scenarios

---

# 🔥 Next Step

Now that you understand how the GC decides what to collect, the next level is:

👉 **C# Fundamentals: Server GC vs Workstation GC**

Here you'll learn the difference between .NET's two garbage collection modes, and how to choose the right one for your application.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
