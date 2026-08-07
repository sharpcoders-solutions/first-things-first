# 🧠 C# Fundamentals: Unsafe Code and Pointers

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Performance with Span and Memory (post 49)  
- C# as a managed language, with the Garbage Collector handling memory  

Since the hardware and software post, you've known there's a memory layer underneath everything. C# usually hides this from you — but in extreme performance scenarios, you can access memory directly, the same way C and C++ do.

👉 **Let's learn Unsafe Code and Pointers**

---

# 💡 Managed vs Unsafe

👉 **Unsafe code = C# code that manipulates pointers directly, stepping outside Garbage Collector protection**

```csharp
public unsafe void BasicExample()
{
    int value = 42;
    int* pointer = &value;

    Console.WriteLine(*pointer); // 42, accessing the value through the pointer
    Console.WriteLine((long)pointer); // the memory address itself
}
```

👉 `&` gets a variable's memory address, `*` dereferences (accesses the pointed-to value) — the same operators you'd see in C

---

# 🏗️ Enabling unsafe code

```xml
<!-- .csproj -->
<PropertyGroup>
  <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
```

```csharp
unsafe
{
    // pointer code is only allowed inside an unsafe block
}
```

👉 The compiler requires this explicit flag — C# won't let you access memory directly by accident, it has to be a conscious decision

---

# 🎯 Pinning memory with `fixed`

```csharp
public unsafe void ProcessArray(int[] numbers)
{
    fixed (int* pointer = numbers)
    {
        for (int i = 0; i < numbers.Length; i++)
        {
            *(pointer + i) *= 2; // access and modify directly in memory
        }
    }
}
```

👉 The Garbage Collector can move objects in memory during a collection. `fixed` temporarily "pins" the array in place, guaranteeing the pointer stays valid while you use it

---

# ⚡ Where this actually matters: extreme performance

```csharp
public unsafe static void FastCopy(byte[] source, byte[] destination)
{
    fixed (byte* pSource = source)
    fixed (byte* pDestination = destination)
    {
        Buffer.MemoryCopy(pSource, pDestination, destination.Length, source.Length);
    }
}
```

👉 Remember the Span/Memory post? Pointers go further — they completely eliminate array bounds-checking overhead in performance-critical loops, something that only makes sense in scenarios like image processing, binary parsers, or interop with C

---

# 🔗 Interoperability with native code

```csharp
[DllImport("kernel32.dll")]
public static unsafe extern bool ReadProcessMemory(
    IntPtr process, IntPtr address, void* buffer, int size, out int bytesRead);
```

👉 Calling existing C/C++ libraries (P/Invoke) frequently requires pointers, because that's how those languages represent memory — C# needs to "speak the same language" at the boundary between the two worlds

---

# ⚠️ Common Mistakes

- Using unsafe code without a real performance need, losing all the memory safety C# provides by default  
- Forgetting `fixed`, letting the Garbage Collector move memory while a pointer still points to it — silently corrupting data  
- Not manually validating bounds, reintroducing the same buffer overflow bugs C# normally prevents  
- Assuming unsafe automatically makes code "faster" — in most cases, the JIT already optimizes managed code very well  

---

# 📌 Conclusion

- Unsafe code lets you manipulate pointers directly, stepping outside Garbage Collector protection  
- `AllowUnsafeBlocks` and the `unsafe` block make that choice explicit  
- `fixed` prevents the GC from moving memory while a pointer is in use  
- Real usage is restricted to extreme scenarios: critical performance and native interoperability  

👉 With unsafe code, you understand that C#'s memory safety is a design choice, not a limitation — and you know exactly when to opt out of it

---

# 🔥 Next Step

Now that you understand what's underneath the managed layer, the next level is:

👉 **C# Fundamentals: Reflection and Custom Attributes**

Here you'll learn to inspect and manipulate types at runtime, the foundation of frameworks like ASP.NET Core itself.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
