# 🧠 C# Fundamentals: Native AOT

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Source Generators for generating code at compile time  
- Docker and deploying .NET applications (post 35)  

Since the .NET architecture post (post 12), you've known that C# code compiles to IL, and the CLR translates it into machine code via JIT, at runtime. Native AOT eliminates that intermediate step entirely.

👉 **Let's learn Native AOT**

---

# 💡 Recap: how .NET normally works

```
C# code → IL (post 12) → CLR loads and JITs it → Machine code (at runtime)
```

👉 The JIT (Just-In-Time) compiler translates IL into machine code the moment the application starts — flexible, but it costs startup time and memory

---

# 🏗️ How Native AOT changes this flow

```
C# code → IL → AOT compilation (during build) → Native executable, no runtime
```

```xml
<!-- .csproj -->
<PropertyGroup>
  <PublishAot>true</PublishAot>
</PropertyGroup>
```

```bash
dotnet publish -c Release -r linux-x64
```

👉 The result is a self-contained native executable — no dependency on the .NET runtime being installed on the target machine, no JIT step, starting up almost instantly

---

# ⚡ The real gains

| Metric | Traditional JIT | Native AOT |
|---|---|---|
| Startup time | ~200-500ms | ~10-50ms |
| Memory usage | Higher (full runtime) | Lower (only what's needed) |
| Deploy size | Requires runtime installed | Self-contained executable |

👉 Remember the Docker post (35)? A Native AOT image can be dramatically smaller — no need for the .NET runtime base image, just the native executable and its minimal dependencies

---

# 🎯 Where this actually matters

- **Serverless / Azure Functions / AWS Lambda** (upcoming posts) — fast cold start is critical when you pay for execution time  
- **Distributed CLIs** — command-line tools that need to start instantly  
- **Kubernetes containers** with aggressive autoscaling, where new pods need to become ready quickly  

---

# ⚠️ The limitations

```csharp
// ❌ Dynamic Reflection doesn't work well with Native AOT
var type = Type.GetType("MyNamespace." + dynamicName);
var instance = Activator.CreateInstance(type);
```

👉 Remember the Reflection post (67)? Native AOT needs to know, **at compile time**, which types exist — code that dynamically discovers types at runtime (non-trimmable Reflection) doesn't work well, or needs special annotations

```csharp
// ✅ Source Generators (post 68) are Native AOT's best friend
[JsonSerializable(typeof(Order))]
public partial class SerializationContext : JsonSerializerContext { }
```

👉 It's no coincidence we talked about Source Generators in the previous post — they solve exactly what Reflection can't do at compile time, and they're the foundation for AOT-compatible libraries

---

# ⚠️ Common Mistakes

- Using Native AOT across an entire application without evaluating library compatibility — not every dependency works well with trimming  
- Expecting the same gains in applications with little startup workload — the benefit is greatest precisely in frequent cold starts  
- Ignoring trimming warnings during the build, which flag code potentially incompatible with AOT  
- Assuming Native AOT is always "better" — for long-running web applications, traditional JIT with tiered compilation sometimes still wins on throughput  

---

# 📌 Conclusion

- Native AOT compiles C# straight to native code, eliminating the runtime JIT  
- The main gain is near-instant startup and lower memory usage  
- Dynamic Reflection has limited support — Source Generators are the recommended path  
- It's especially valuable in serverless, CLIs, and containers with aggressive autoscaling  

👉 With Native AOT, .NET applications compete on equal footing with natively compiled languages in scenarios where cold start is critical

---

# 🔥 Next Step

Now that you know how to compile to native code, the next level is:

👉 **C# Fundamentals: ArrayPool and Object Pooling**

Here you'll learn to reduce pressure on the Garbage Collector by reusing objects instead of constantly allocating new ones.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
