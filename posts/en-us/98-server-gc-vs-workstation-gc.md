# 🧠 C# Fundamentals: Server GC vs Workstation GC

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Generations, finalizers, and `WeakReference` — how the GC decides what to collect  
- ArrayPool and Object Pooling (post 73), techniques for reducing GC pressure  

You already know how the GC decides **what** to collect. But .NET offers two completely different modes for **how** that collection happens — a choice that can have a huge impact on your application's performance, and one most developers never configure consciously.

👉 **Let's understand Server GC vs Workstation GC**

---

# 💡 The two modes: optimized for opposite scenarios

👉 **Workstation GC = optimized for low latency, one processing core at a time. Server GC = optimized for high throughput, using multiple cores in parallel**

```xml
<!-- .csproj -->
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
</PropertyGroup>
```

👉 This single setting in the `.csproj` (or in `runtimeconfig.json`) fundamentally changes how the GC operates under the hood

---

# 🖥️ Workstation GC: the default for desktop and CLI apps

```
- A single heap, managed by a single GC thread
- Optimized for short pauses and responsiveness
- Default for: desktop apps, CLI, command-line tools
```

👉 In a desktop application (remember Blazor and .NET MAUI?), a long GC pause is immediately visible to the user — the UI freezes. Workstation GC prioritizes minimizing those pauses, even if it means collecting more frequently

---

# 🖧 Server GC: the default for web APIs and services

```
- One heap PER CPU CORE, each with its own GC thread
- Parallel collections across multiple cores simultaneously
- Default for: ASP.NET Core applications (already configured automatically)
```

👉 Server GC splits the managed heap into parts, one per available CPU core, and collects each part in parallel. This means **much higher total throughput** (more allocation/collection per second), at the cost of using more memory (one heap per core, not a single shared heap)

```csharp
// An ASP.NET Core API already uses Server GC by default since .NET Core 3.0+
// but older configurations, or custom hosts, may need the explicit setting
```

---

# ⚖️ Comparing both side by side

| | Workstation GC | Server GC |
|---|---|---|
| Heaps | Single | One per CPU core |
| Optimized for | Low latency | High throughput |
| Memory usage | Lower | Higher |
| Ideal scenario | Desktop, CLI, interactive apps | Web APIs, high-volume services |
| Collection parallelism | Limited | Full, per core |

---

# 🔄 `ConcurrentGarbageCollection`: reducing pauses in both modes

```xml
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
</PropertyGroup>
```

👉 **Concurrent GC = lets Generation 2 collections (the most expensive, remember the previous post?) run on a separate thread, while the application keeps executing**, reducing (but not eliminating) noticeable pauses even during large collections. This works with both Workstation and Server GC, and is generally worth keeping enabled

---

# 🎯 When to manually tune it

```xml
<!-- Container with few allocated cores: Server GC might allocate too many heaps -->
<PropertyGroup>
  <ServerGarbageCollection>false</ServerGarbageCollection>
</PropertyGroup>
```

👉 **Practical rule: in most cases, ASP.NET Core's default (Server GC) is already the right choice for APIs.** The most common exception: containers with few CPU cores allocated (remember the Kubernetes and resource limits post?) — in these cases, Server GC can allocate more heaps than makes sense for the actual amount of available CPU, and Workstation GC (or Server GC with a limited `HeapCount`) can behave better

```xml
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <GCHeapCount>2</GCHeapCount> <!-- explicitly limits the number of heaps -->
</PropertyGroup>
```

---

# 🔍 Measuring the real impact

```bash
dotnet-counters monitor --process-id <pid> System.Runtime
```

👉 Remember `BenchmarkDotNet` (from the performance post)? Just like you don't optimize without measuring, don't switch GC modes without measuring the real impact with tools like `dotnet-counters`, observing pause-time and throughput metrics before and after the change

---

# ⚠️ Common Mistakes

- Assuming Server GC is always better "because it's the production default," without considering the actual execution environment (containers with few cores, for example)  
- Switching GC mode without measuring before and after, based purely on intuition  
- Ignoring `ConcurrentGarbageCollection`, missing out on a nearly free pause reduction  
- Configuring Server GC on a desktop application, increasing memory usage with no real throughput gain relevant to that scenario  

---

# 📌 Conclusion

- Workstation GC prioritizes low latency with a single heap; Server GC prioritizes throughput with one heap per core  
- ASP.NET Core already uses Server GC by default since .NET Core 3.0+  
- `ConcurrentGarbageCollection` reduces pauses in both modes, running large collections in parallel  
- Containers with few cores can benefit from tuning `GCHeapCount` or using Workstation GC  

👉 With the Garbage Collector fully mastered — generations, finalizers, `WeakReference`, and now the two operating modes — the next step is stepping away from raw performance and into another fundamental aspect: how your code behaves across different cultures and languages

---

# 🔥 Next Step

Now that you understand .NET's two garbage collection modes, the next level is:

👉 **C# Fundamentals: Culture and Globalization**

Here you'll learn to handle number, date, and string formatting correctly for users from different countries and languages.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
