# 🧠 C# Fundamentals: Assembly, Metadata, and Reflection in Depth

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Culture and globalization for correct region-based formatting  
- Reflection and custom attributes (post 70), at the level of inspecting one class at a time  

Post 70 showed you reflection applied to a specific class. Now let's level up: how to inspect an **entire assembly**, discover every type it contains, and understand the metadata structure that makes all of this possible.

👉 **Let's explore Assembly, Metadata, and Reflection in depth**

---

# 💡 What is an Assembly, really?

👉 **Assembly = .NET's unit of deployment and versioning — a compiled `.dll` or `.exe` file, containing IL (Intermediate Language) code and metadata about everything it defines**

```csharp
Assembly assembly = Assembly.GetExecutingAssembly();

Console.WriteLine(assembly.FullName);
// MyProject, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null

Console.WriteLine(assembly.Location);
// C:\projects\myproject\bin\Debug\net10.0\MyProject.dll
```

👉 Every `.dll` you've referenced throughout this track — Serilog, EF Core, your own projects — is an assembly. Each one carries, alongside its compiled code, a complete metadata catalog: what types exist, what methods each type has, what attributes were applied

---

# 🔍 Exploring every type in an assembly

```csharp
Assembly assembly = Assembly.GetExecutingAssembly();

foreach (Type type in assembly.GetTypes())
{
    if (type.IsPublic && type.IsClass)
    {
        Console.WriteLine($"Public class: {type.FullName}");

        foreach (var method in type.GetMethods(BindingFlags.Public | BindingFlags.Instance))
        {
            Console.WriteLine($"  Method: {method.Name}");
        }
    }
}
```

👉 `GetTypes()` returns **every** type defined in that assembly — this is literally the technique behind frameworks that automatically "discover" controllers, MediatR handlers (post 45), or FluentValidation validators, without you needing to register each one manually

---

# 🏷️ Attribute metadata: going beyond the basics

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class ModuleAttribute : Attribute
{
    public string Name { get; }
    public ModuleAttribute(string name) => Name = name;
}

[Module("Sales")]
public class OrdersService { }

[Module("Inventory")]
public class StockService { }
```

```csharp
var typesWithModule = assembly.GetTypes()
    .Where(t => t.GetCustomAttribute<ModuleAttribute>() != null)
    .Select(t => new
    {
        Type = t.Name,
        Module = t.GetCustomAttribute<ModuleAttribute>()!.Name
    });

foreach (var item in typesWithModule)
    Console.WriteLine($"{item.Type} belongs to module {item.Module}");
```

👉 Remember the custom attributes post? Scaling this technique to an entire assembly, you can build automatic discovery systems — grouping classes by business module, generating documentation, or validating architecture conventions automatically

---

# 🌐 Loading assemblies dynamically

```csharp
Assembly externalAssembly = Assembly.LoadFrom(@"C:\plugins\myplugin.dll");

Type[] types = externalAssembly.GetExportedTypes();

foreach (var type in types.Where(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface))
{
    var instance = (IPlugin)Activator.CreateInstance(type)!;
    instance.Execute();
}
```

👉 `Assembly.LoadFrom` loads an assembly that wasn't referenced at compile time — the technical foundation for plugin systems, where the main application doesn't know about extensions until they're loaded at runtime

---

# ⚙️ `MethodInfo.Invoke`: calling dynamically discovered methods

```csharp
Type type = Type.GetType("MyProject.OrdersService")!;
object instance = Activator.CreateInstance(type)!;

MethodInfo method = type.GetMethod("ProcessOrder")!;
object? result = method.Invoke(instance, new object[] { 123 });
```

👉 This is reflection at its most dynamic: discovering a type by name (string), creating an instance, and calling a method — with no static reference to the type at compile time. Powerful, but remember the generic math post and the cost of reflection? This is significantly slower than a direct call, and it's exactly the kind of code that breaks in Native AOT without extra care

---

# ⚠️ Common Mistakes

- Using heavy reflection in a hot path, when caching `MethodInfo`/`Type` (computed once, reused afterward) would solve the performance problem  
- Loading external assemblies with no validation at all, creating a security vector for malicious code  
- Forgetting `Invoke` propagates exceptions wrapped in `TargetInvocationException`, requiring `.InnerException` to see the real error  
- Heavily relying on dynamic reflection in projects that also need to run on Native AOT, without testing that combination early  

---

# 📌 Conclusion

- An Assembly is .NET's deployment unit, carrying IL code and complete metadata about its types  
- `GetTypes()` lets you discover every type in an assembly — the foundation of auto-discovery frameworks  
- Custom attributes, combined with assembly-level reflection, enable convention systems and automatic documentation  
- `Assembly.LoadFrom` and `MethodInfo.Invoke` let you load and execute code entirely unknown at compile time  

👉 Speaking of loading assemblies dynamically — there's an even more advanced feature that lets you not just load, but also **unload** assemblies at runtime, the foundation of true plugin systems

---

# 🔥 Next Step

Now that you explore assemblies and metadata in depth, the next (and second-to-last) level is:

👉 **C# Fundamentals: AssemblyLoadContext and Plugin Systems**

Here you'll learn to load and unload plugins at runtime, without needing to restart the main application.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
