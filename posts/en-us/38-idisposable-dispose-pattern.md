# 🧠 C# Fundamentals: IDisposable and the Dispose Pattern

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Extension Methods and Custom LINQ  
- Authentication and Authorization with JWT  

You've written `using var context = new AppDbContext(...)` dozens of times since the EF Core post, without stopping to understand what `using` is actually doing. Time to open that box.

👉 **Let's learn IDisposable and the Dispose Pattern**

---

# 💡 The problem: resources the Garbage Collector doesn't know about

Remember the processes and memory post? The Garbage Collector manages **managed** memory — ordinary C# objects. But database connections, open files, and network sockets are **unmanaged** resources, controlled by the operating system. The GC doesn't know when to release them.

```csharp
// ❌ The connection might never be explicitly closed
var connection = new SqlConnection(connectionString);
connection.Open();
// if an exception happens here, the connection stays open indefinitely
```

---

# 🏗️ The `IDisposable` interface

```csharp
public interface IDisposable
{
    void Dispose();
}
```

👉 Any class that manages an unmanaged resource implements this interface — `Dispose()` is the explicit signal for "I'm done, you can release this now"

```csharp
var connection = new SqlConnection(connectionString);
try
{
    connection.Open();
    // use the connection
}
finally
{
    connection.Dispose(); // always runs, even with an exception
}
```

---

# 🎯 `using`: the `try/finally` you already use

```csharp
using var connection = new SqlConnection(connectionString);
connection.Open();
// Dispose() is called automatically at the end of scope
```

👉 This is exactly the `using var context = new AppDbContext(...)` you've written since the EF Core post — the compiler turns this into the same `try/finally` you just saw, just without needing to write it manually

```csharp
// Explicit block form, equivalent
using (var connection = new SqlConnection(connectionString))
{
    connection.Open();
} // Dispose() called here
```

---

# 🔨 Implementing `IDisposable` in your own class

```csharp
public class TemporaryFile : IDisposable
{
    private readonly FileStream _stream;
    private bool _disposed;

    public TemporaryFile(string path)
    {
        _stream = new FileStream(path, FileMode.Create);
    }

    public void WriteData(byte[] data) => _stream.Write(data, 0, data.Length);

    public void Dispose()
    {
        if (_disposed) return;

        _stream.Dispose();
        _disposed = true;
    }
}
```

```csharp
using var file = new TemporaryFile("data.tmp");
file.WriteData(bytes);
// Dispose() releases the FileStream automatically
```

👉 The `_disposed` flag prevents calling `Dispose()` twice — a common precaution, since `using` can coexist with a manual call somewhere else in the code

---

# ⚡ Async: `IAsyncDisposable`

```csharp
public class AsyncConnection : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await ReleaseResourcesAsync();
    }
}
```

```csharp
await using var connection = new AsyncConnection();
```

👉 Remember the async/await post? Releasing a resource sometimes requires an asynchronous operation (closing a network connection, for example) — `await using` is the async equivalent of regular `using`, calling `DisposeAsync()` instead of `Dispose()`

---

# ⚠️ Common Mistakes

- Forgetting `using` (or a manual `Dispose()`) on resources like `SqlConnection`, `FileStream`, and `HttpClient` outside a `using` block, gradually leaking operating system resources  
- Implementing `IDisposable` without actually having an unmanaged resource to release — the interface exists for a specific purpose, not as a generic "cleanup" pattern  
- Calling methods on the object after `Dispose()`, resulting in an `ObjectDisposedException`  
- Not propagating `IDisposable` upward when a class holds another disposable object as a field — whoever creates the field is usually responsible for disposing it  

---

# 📌 Conclusion

- `IDisposable` signals that a class controls unmanaged resources, outside the Garbage Collector's reach  
- `using` is syntactic sugar for a `try/finally` calling `Dispose()` automatically  
- `IAsyncDisposable`/`await using` cover the case where releasing the resource requires an asynchronous operation  
- Implementing the interface only makes sense when there's a real resource to release  

👉 With IDisposable, the `using var` you've typed dozens of times since the EF Core post stops being memorized syntax and becomes a mechanism you understand end to end

---

# 🔥 Next Step

Now that you know how to release resources correctly, the next level is:

👉 **C# Fundamentals: Streams and File I/O in C#**

Here you'll learn to work with files and data streams efficiently, using the same resource discipline you just learned.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
