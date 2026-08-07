# 🧠 C# Fundamentals: Source Generators

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Reflection for inspecting types at runtime  
- Roslyn Analyzers for creating custom analysis rules (post 65)  

Reflection is powerful, but it has a cost: it runs at runtime, every single time. Source Generators use the same Roslyn infrastructure, but generate real code at **compile** time — zero cost while the application is running.

👉 **Let's learn Source Generators**

---

# 💡 The problem Source Generators solve

```csharp
// With Reflection (post 70): slow, runs on every call
public static void SerializeWithReflection(object obj)
{
    var properties = obj.GetType().GetProperties(); // runtime cost
    // ...
}
```

```csharp
// With a Source Generator: the code already exists, compiled, no Reflection at all
public static void SerializeOrder(Order order)
{
    // code automatically generated during build,
    // with no Reflection calls at runtime
}
```

👉 `System.Text.Json` (which you've used since the API posts) has a Source Generator mode for exactly this reason — serialization without Reflection is significantly faster

---

# 🏗️ How a Source Generator works

```csharp
[Generator]
public class ToStringGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var classes = context.SyntaxProvider
            .CreateSyntaxProvider(
                predicate: (node, _) => node is ClassDeclarationSyntax,
                transform: (ctx, _) => (ClassDeclarationSyntax)ctx.Node)
            .Where(cls => HasAttribute(cls, "GenerateToString"));

        context.RegisterSourceOutput(classes, (spc, cls) =>
        {
            var code = GenerateToStringCode(cls);
            spc.AddSource($"{cls.Identifier}_ToString.g.cs", code);
        });
    }
}
```

👉 This looks like the Roslyn Analyzer from post 65 — but instead of just reporting a diagnostic, the generator **adds new C# source files** to the compilation, before the build even finishes

---

# 🎯 Using it in practice

```csharp
[GenerateToString]
public partial class Order
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
}
```

```csharp
// Automatically generated file: Order_ToString.g.cs
public partial class Order
{
    public override string ToString() => $"Order {{ Id = {Id}, Amount = {Amount} }}";
}
```

👉 You write a `partial class Order` with the attribute, and the compiler generates the rest — it looks like magic, but it's just regular C# code, written by other C# code, during compilation

---

# ⚡ Where this is already used in the real world

- **System.Text.Json** — Reflection-free serialization via `[JsonSerializable]`  
- **Regex** — `[GeneratedRegex]` compiles regular expressions at build time, instead of interpreting them at runtime  
- **ASP.NET Core Minimal APIs** — code generation for high-performance routing  

```csharp
public partial class MyRegex
{
    [GeneratedRegex(@"^\d{3}-\d{4}$")]
    public static partial Regex PhoneRegex();
}
```

👉 Remember the LINQ and performance post? `[GeneratedRegex]` is literally faster than `new Regex(...)` because the pattern is compiled during the build, not interpreted on every run

---

# ⚠️ Common Mistakes

- Writing Source Generators for simple problems that Reflection would already solve without any real performance issue  
- Not testing the generator in isolation, making incorrectly generated code hard to debug  
- Generating code that conflicts with code manually written in the same `partial class`  
- Underestimating the learning curve — Source Generators require understanding Roslyn's syntax tree, like in post 65  

---

# 📌 Conclusion

- Source Generators create real C# code during compilation, not at runtime  
- This eliminates Reflection overhead in high-performance scenarios  
- `System.Text.Json` and `[GeneratedRegex]` are real examples already used in this track  
- The technical foundation is the same as the Roslyn Analyzers from post 65, but generating code instead of just diagnostics  

👉 With Source Generators, you trade Reflection's runtime cost for a (one-time) compilation cost — the best of both worlds in critical performance scenarios

---

# 🔥 Next Step

Now that you know how to generate code at compile time, the next level is:

👉 **C# Fundamentals: Native AOT**

Here you'll learn to compile your C# application straight to native code, without depending on the .NET runtime being installed on the target machine.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
