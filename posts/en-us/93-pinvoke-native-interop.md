# 🧠 C# Fundamentals: P/Invoke and Native Interoperability

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Advanced `System.Text.Json` and the serialization source generator  
- Unsafe code and pointers (post 66), including a quick introduction to P/Invoke  

The unsafe code post showed you a quick `[DllImport]` example. Time to understand P/Invoke in depth — the bridge that lets C# call native libraries written in C, C++, or any language exposing a C-compatible API.

👉 **Let's learn P/Invoke in depth**

---

# 💡 What is P/Invoke?

👉 **P/Invoke (Platform Invoke) = .NET's mechanism for calling functions from native libraries (`.dll`, `.so`, `.dylib`) from managed code**

```csharp
[DllImport("user32.dll")]
public static extern int MessageBox(IntPtr hWnd, string text, string title, uint type);

MessageBox(IntPtr.Zero, "Hello from C#!", "P/Invoke", 0);
```

👉 `[DllImport]` tells the runtime where to find the native function. The `extern` method has no body in C# — the implementation lives entirely in the native library, and the CLR bridges the two worlds at runtime

---

# 🔄 Marshaling: translating types between worlds

```csharp
[DllImport("mynativelib.dll", CharSet = CharSet.Unicode)]
public static extern int ProcessText(string input, StringBuilder output, int bufferSize);

var output = new StringBuilder(256);
ProcessText("input data", output, output.Capacity);
Console.WriteLine(output.ToString());
```

👉 **Marshaling = the process of converting types between .NET's managed representation and the native representation used by the C/C++ library**

`string` in C# isn't the same as `char*` in C — marshaling handles that conversion automatically in most cases, but `CharSet.Unicode` (or `Ansi`) controls exactly how that translation happens, something that needs to exactly match what the native library expects

---

# 📦 Structs and marshaling complex data

```csharp
[StructLayout(LayoutKind.Sequential)]
public struct NativePoint
{
    public int X;
    public int Y;
}

[DllImport("mylib.dll")]
public static extern void ProcessPoint(ref NativePoint point);

var point = new NativePoint { X = 10, Y = 20 };
ProcessPoint(ref point);
```

👉 `[StructLayout(LayoutKind.Sequential)]` guarantees the struct's fields sit in memory in exactly the same order and layout the native library expects — remember the value types post (post 43)? This is a case where the exact memory layout, usually an internal runtime concern, becomes your explicit responsibility

---

# 🎯 LibraryImport: the modern evolution of `DllImport`

```csharp
public static partial class Native
{
    [LibraryImport("mylib.dll")]
    public static partial int Add(int a, int b);
}
```

👉 **`[LibraryImport]` = the modern version of `[DllImport]`, based on a source generator (remember `GeneratedRegex` and `JsonSerializerContext`?)**

Just like the other source-generator-based features you've already seen, `[LibraryImport]` generates marshaling code at **compile** time, instead of using reflection at runtime — faster, and compatible with Native AOT (post 69), where traditional `[DllImport]` can have limitations

---

# 🔗 Callbacks: C calling C#

```csharp
public delegate void ProgressCallback(int percentage);

[DllImport("mylib.dll")]
public static extern void ProcessWithProgress(ProgressCallback callback);

ProcessWithProgress(percentage =>
{
    Console.WriteLine($"Progress: {percentage}%");
});
```

👉 Remember delegates (from the events post)? A C# delegate can be passed as a function pointer to native code — the C library calls back into C# during processing, useful for reporting progress on long-running operations

---

# ⚠️ Memory safety concerns

```csharp
[DllImport("mylib.dll")]
public static extern IntPtr AllocateBuffer(int size);

[DllImport("mylib.dll")]
public static extern void FreeBuffer(IntPtr buffer);

IntPtr buffer = AllocateBuffer(1024);
try
{
    // use the buffer
}
finally
{
    FreeBuffer(buffer); // MANDATORY — the GC doesn't manage native memory
}
```

👉 Remember the `IDisposable` post? Memory allocated on the native side is **never** tracked by .NET's Garbage Collector — forgetting to explicitly free it is a real memory leak, the same kind that existed in C before any managed runtime

---

# ⚠️ Common Mistakes

- Forgetting to free natively allocated memory, causing leaks the GC will never detect or collect  
- Configuring `CharSet` incorrectly, causing string corruption when crossing the managed/native boundary  
- Using traditional `[DllImport]` in Native AOT projects without testing, when `[LibraryImport]` would offer more compatibility  
- Ignoring calling convention differences between platforms, causing hard-to-debug crashes  

---

# 📌 Conclusion

- P/Invoke lets you call native C/C++ libraries directly from C#  
- Marshaling converts types between the managed and native representation automatically, with fine control via attributes  
- `[LibraryImport]` is the modern, source-generator-based evolution, faster and Native AOT-compatible  
- Natively allocated memory is never managed by the GC — freeing it explicitly is your responsibility  

👉 Speaking of the GC: you've seen it mentioned dozens of times throughout this track, but never in depth. Time to understand exactly how it decides when to collect an object

---

# 🔥 Next Step

Now that you know how to interact with native code, the next level is:

👉 **C# Fundamentals: WeakReference, Finalizers, and the Garbage Collector**

Here you'll learn how the GC actually decides when to collect an object, and how weak references avoid memory leaks in specific scenarios.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
