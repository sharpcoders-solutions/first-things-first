# 🧠 C# Fundamentals: AssemblyLoadContext and Plugin Systems

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Assembly, metadata, and reflection in depth, including `Assembly.LoadFrom`  
- WeakReference and how the GC collects objects with no more strong references (post 94)  

You saw how to load an assembly dynamically. But there's a real problem with `Assembly.LoadFrom`: once loaded, that assembly is **never** unloaded — it stays stuck in memory forever, even if you no longer need it. `AssemblyLoadContext` solves exactly that.

👉 **Let's learn `AssemblyLoadContext` and build a real plugin system**

---

# 💡 The problem: loaded assemblies never go away

```csharp
// This is never unloaded, even if "assembly" goes out of scope
Assembly assembly = Assembly.LoadFrom(@"C:\plugins\myplugin.dll");
```

👉 .NET's default loading context (`AssemblyLoadContext.Default`) keeps **every** assembly loaded into it for the application's entire lifetime. For a plugin you want to update, replace, or remove without restarting the main application, that's a real problem

---

# 🏗️ Creating an isolated, unloadable `AssemblyLoadContext`

```csharp
public class PluginLoadContext : AssemblyLoadContext
{
    private readonly AssemblyDependencyResolver _resolver;

    public PluginLoadContext(string pluginPath) : base(isCollectible: true)
    {
        _resolver = new AssemblyDependencyResolver(pluginPath);
    }

    protected override Assembly? Load(AssemblyName assemblyName)
    {
        string? path = _resolver.ResolveAssemblyToPath(assemblyName);
        return path != null ? LoadFromAssemblyPath(path) : null;
    }
}
```

👉 **`isCollectible: true` = the parameter that makes this context unloadable by the Garbage Collector**, unlike the default context. `AssemblyDependencyResolver` handles resolving the plugin's dependencies (other `.dll`s it needs), isolated from the rest of the application

---

# 🔌 Loading and using a plugin

```csharp
public interface IPlugin
{
    string Name { get; }
    void Execute();
}
```

```csharp
var context = new PluginLoadContext(@"C:\plugins\MyPlugin.dll");
Assembly pluginAssembly = context.LoadFromAssemblyPath(@"C:\plugins\MyPlugin.dll");

Type pluginType = pluginAssembly.GetTypes()
    .First(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface);

var plugin = (IPlugin)Activator.CreateInstance(pluginType)!;
plugin.Execute();
```

👉 Remember the assembly and reflection post? The type discovery technique is the same — the difference is the assembly now lives in an isolated context, which can be discarded afterward

---

# 🗑️ Unloading the plugin

```csharp
WeakReference UnloadContext(PluginLoadContext context)
{
    var weakReference = new WeakReference(context, trackResurrection: true);
    context.Unload();
    return weakReference;
}

var reference = UnloadContext(context);
context = null;
plugin = null;

for (int i = 0; i < 10 && reference.IsAlive; i++)
{
    GC.Collect();
    GC.WaitForPendingFinalizers();
}

Console.WriteLine(reference.IsAlive ? "Still loaded" : "Successfully unloaded");
```

👉 Remember the `WeakReference` post? That's exactly the right tool for **verifying** whether the unload actually happened — `Unload()` signals intent, but the real unloading only happens once the GC collects **every** reference to types from that assembly. If you're still holding a live reference (a static variable, an unsubscribed event), the context never gets collected — the same leak problem you already saw with poorly managed `event`s

---

# ⚠️ The biggest risk: references that block unloading

```csharp
// ❌ This blocks unloading forever
public static class PluginRegistry
{
    public static List<IPlugin> ActivePlugins = new(); // static strong reference
}

PluginRegistry.ActivePlugins.Add(plugin); // the plugin will never be collected while it's here
```

👉 Any strong reference to a plugin type — in a static collection, in an event subscribed without unsubscribing, in a still-active `Timer` — prevents the `AssemblyLoadContext` from being collected. Before calling `Unload()`, you need to guarantee **nothing** in the rest of the application still references types from that plugin

---

# 🎯 Real use cases

- **Code editors with extensions** (like Visual Studio itself) — third-party plugins loaded and updated without restarting the editor  
- **Dynamic business rule systems** — loading a new rule version without redeploying the entire application  
- **Testing and hot-reload tools** — reloading modified code without restarting the whole process  

👉 Outside these specific scenarios, most applications never need `AssemblyLoadContext` — it's a powerful but niche feature, reserved for when a plugin architecture is a real product requirement

---

# ⚠️ Common Mistakes

- Forgetting static references or unsubscribed events, preventing unloading from working even after calling `Unload()`  
- Not using `AssemblyDependencyResolver`, causing version conflicts between the plugin's dependencies and the main application's  
- Assuming `Unload()` unloads immediately — the process is asynchronous and depends on the GC collecting every remaining reference  
- Using `AssemblyLoadContext` for a simple scenario `Assembly.LoadFrom` would already solve, adding complexity with no real need for unloading  

---

# 📌 Conclusion

- `Assembly.LoadFrom` loads assemblies that are never unloaded, a real problem for short-lived plugins  
- `AssemblyLoadContext` with `isCollectible: true` allows unloading an assembly's context, freeing memory  
- `AssemblyDependencyResolver` isolates a plugin's dependencies from the rest of the application  
- Any remaining strong reference blocks unloading — the same care you already saw with `WeakReference` and memory leaks  

👉 With `AssemblyLoadContext`, you close the loop on everything you've learned about assemblies, reflection, and the Garbage Collector — building systems that truly load and unload code at runtime

---

# 🔥 Next Step

You're nearing the end of this track. After mastering dynamic plugins, the next (second-to-last) level is:

👉 **C# Fundamentals: Keeping Up with .NET's Evolution**

Here you'll learn to stay current with .NET's release cycle, and where to find reliable information about every new version.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
