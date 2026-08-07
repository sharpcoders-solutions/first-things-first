# 🧠 C# Fundamentals: SIMD and Hardware Intrinsics in C#

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- `volatile`, `MemoryBarrier`, and how threads actually see shared memory  
- `Span<T>` and `Memory<T>`, from the performance post, for working with contiguous data without extra allocations  

You've already optimized memory with `Span<T>`. But there's another dimension of performance, practically unexplored so far: processing **several numeric values at once**, in a single CPU instruction, instead of a traditional `for` loop processing one value per iteration.

👉 **Let's understand SIMD and Hardware Intrinsics in C#**

---

# 💡 What is SIMD?

👉 **SIMD = Single Instruction, Multiple Data — a category of CPU instruction that applies the same operation to several numeric values simultaneously, in a single cycle**

```csharp
// Traditional loop: one addition per iteration
for (int i = 0; i < array.Length; i++)
{
    result[i] = array[i] + other[i];
}
```

👉 Modern processors (x86-64, ARM) have registers capable of holding 128, 256, or even 512 bits — enough for several `int`s or `float`s at once. A traditional loop uses only a fraction of that capacity per iteration; SIMD uses the whole register at once

---

# 🔢 `Vector<T>`: portable SIMD, without worrying about the exact CPU

```csharp
using System.Numerics;

void AddVectors(float[] a, float[] b, float[] result)
{
    int vectorSize = Vector<float>.Count; // varies by hardware (e.g., 8 on AVX2)
    int i = 0;

    for (; i <= a.Length - vectorSize; i += vectorSize)
    {
        var va = new Vector<float>(a, i);
        var vb = new Vector<float>(b, i);
        (va + vb).CopyTo(result, i);
    }

    // Leftover elements that don't fill a whole vector: process normally
    for (; i < a.Length; i++)
    {
        result[i] = a[i] + b[i];
    }
}
```

👉 **`Vector<T>` = a type representing N numeric values processed as a single unit, where N (`Vector<T>.Count`) automatically adapts to the hardware the code runs on** — the same code compiles to SSE, AVX2, or AVX-512 instructions depending on the processor, without you writing different versions

---

# ⚡ Checking hardware acceleration

```csharp
if (Vector.IsHardwareAccelerated)
{
    Console.WriteLine($"SIMD accelerated, processing {Vector<float>.Count} floats at a time");
}
else
{
    Console.WriteLine("No hardware acceleration, falling back to scalar");
}
```

👉 `Vector<T>` always works, even on hardware without SIMD support — in that case, the implementation simply falls back to scalar operations under the hood. `IsHardwareAccelerated` lets you decide whether it's worth using the vector API or simplifying to a regular loop in scenarios where maximum portability matters more than raw performance

---

# 🎯 `System.Runtime.Intrinsics`: full control, CPU-specific

```csharp
using System.Runtime.Intrinsics;
using System.Runtime.Intrinsics.X86;

unsafe void AddWithAvx2(float* a, float* b, float* result, int size)
{
    if (!Avx2.IsSupported) throw new PlatformNotSupportedException();

    int i = 0;
    for (; i <= size - 8; i += 8)
    {
        var va = Avx.LoadVector256(a + i);
        var vb = Avx.LoadVector256(b + i);
        var vr = Avx.Add(va, vb);
        Avx.Store(result + i, vr);
    }
}
```

👉 While `Vector<T>` abstracts the hardware away, `System.Runtime.Intrinsics` (`Avx2`, `Sse2`, `AdvSimd` for ARM) exposes instructions **specific** to each processor family, mapped almost 1:1 to the actual assembly — more control and potentially more performance, at the cost of writing (and maintaining) one version per CPU architecture

---

# ⚖️ `Vector<T>` vs specific intrinsics: which to choose

| | `Vector<T>` | `System.Runtime.Intrinsics` |
|---|---|---|
| Portability | High — same code on any CPU | Low — needs an `IsSupported` check per family |
| Control | Limited to the generic operations available | Full — access to specific instructions |
| Complexity | Low | High |
| When to use | Most bulk numeric processing scenarios | Very low-level libraries (codecs, cryptography, ML) |

👉 **Practical rule: start with `Vector<T>`. Only drop down to `System.Runtime.Intrinsics` when profiling shows a specific instruction (like `Avx2.Fma` for fused-multiply-add) solves a real bottleneck `Vector<T>` can't reach**

---

# 📊 A real use case: summing a large array

```csharp
float SumArray(float[] values)
{
    var sumVector = Vector<float>.Zero;
    int i = 0;

    for (; i <= values.Length - Vector<float>.Count; i += Vector<float>.Count)
    {
        sumVector += new Vector<float>(values, i);
    }

    float sum = Vector.Sum(sumVector);

    for (; i < values.Length; i++)
    {
        sum += values[i];
    }

    return sum;
}
```

👉 For large arrays (thousands of elements), this version can be several times faster than a `foreach` with LINQ's `Sum()` — the trade-off is readability: manually vectorized code is always more complex than the naive version, and it's only worth it once profiling proves that part is genuinely a bottleneck

---

# ⚠️ Common Mistakes

- Manually vectorizing code the JIT already optimizes automatically (auto-vectorization), adding complexity with no real gain  
- Using `System.Runtime.Intrinsics` without checking `IsSupported`, breaking on hardware that lacks that specific instruction  
- Forgetting the "leftover" part of an array that doesn't fill a whole `Vector<T>.Count`, causing incomplete or incorrect results  
- Optimizing with SIMD before confirming, with a real benchmark (remember the performance post?), that this section is actually a relevant bottleneck  

---

# 📌 Conclusion

- SIMD processes multiple numeric values in a single CPU instruction instead of one at a time  
- `Vector<T>` offers portable SIMD, automatically adapting to the available hardware  
- `System.Runtime.Intrinsics` exposes CPU-specific instructions for maximum control and performance  
- Manual vectorization is a last-resort optimization, reserved for bottlenecks proven by profiling  

👉 With vectorized processing in your toolkit, you close out the block on numeric types and performance — next up, we come back to generic math, unifying everything you've learned about numeric types under a single abstraction

---

# 🔥 Next Step

Now that you process multiple numeric values in parallel within a single core, the next level is:

👉 **C# Fundamentals: Generic Math**

Here you'll learn to write a single generic method that works with `int`, `double`, `decimal`, and any other numeric type, using the static abstract interface members you already know.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
