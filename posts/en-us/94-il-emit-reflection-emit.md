# 🧠 C# Fundamentals: Reflection.Emit and Runtime Code Generation (IL Emit)

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- `required` members and `init`-only properties for correctly initialized objects  
- Reflection and custom attributes, for **inspecting** types at runtime  

You already use reflection to inspect a type — discovering its properties, calling its methods. But there's a step beyond that: instead of just **reading** existing code, you can **generate new code**, in memory, while the program is running. That's what `Reflection.Emit` does.

👉 **Let's understand runtime code generation with IL Emit**

---

# 💡 What `Reflection.Emit` actually does

👉 **`Reflection.Emit` = an API that lets you build entire methods, types, and assemblies in memory, writing IL (Intermediate Language) directly, without going through the C# compiler**

```csharp
using System.Reflection;
using System.Reflection.Emit;

var method = new DynamicMethod("Add", typeof(int), new[] { typeof(int), typeof(int) });
var il = method.GetILGenerator();

il.Emit(OpCodes.Ldarg_0); // load the first parameter
il.Emit(OpCodes.Ldarg_1); // load the second parameter
il.Emit(OpCodes.Add);     // add the two values on top of the stack
il.Emit(OpCodes.Ret);     // return the result

var add = (Func<int, int, int>)method.CreateDelegate(typeof(Func<int, int, int>));
Console.WriteLine(add(3, 4)); // 7
```

👉 This code builds, **at runtime**, the equivalent of `int Add(int a, int b) => a + b;` — each `Emit` writes an IL instruction directly, the same intermediate language the C# compiler generates for any method you write normally

---

# 🤔 Why generate code at runtime, if you could just write C#?

```csharp
// If you know the types at compile time, just write normal C#:
int Add(int a, int b) => a + b;

// Reflection.Emit only makes sense when the type/logic only exists at runtime:
DynamicMethod CreatePropertyAccessor(PropertyInfo property)
{
    // dynamically generates an optimized getter for ANY property
    // discovered at runtime, without slow reflection on every call
}
```

👉 The real use case: when you need code that can only be determined at runtime — an ORM that generates optimized accessors for an entity's properties discovered via reflection, a mocking framework that creates fake interface implementations on demand, a serializer that builds type-specific delegates. Hand-writing that code for every possible type is unworkable; generating it dynamically solves that

---

# ⚡ The problem `Reflection.Emit` solves: raw reflection is slow

```csharp
PropertyInfo property = typeof(Product).GetProperty("Price")!;

// ❌ Slow: raw reflection, invoked on every call
object value = property.GetValue(product);

// ✅ Fast: generate a delegate once, reuse it afterward
Func<Product, decimal> compiledGetter = CreateGetter(property);
decimal value2 = compiledGetter(product); // as fast as a direct call
```

👉 Remember the assembly and reflection post, and the cost of `MethodInfo.Invoke`? `Reflection.Emit` is exactly the technique libraries like Dapper, AutoMapper, and high-performance serializers use to eliminate that cost: generate, once, a compiled delegate that accesses the property directly — after that, no more reflection happens on subsequent calls

---

# 🏗️ `AssemblyBuilder` and `TypeBuilder`: generating entire types, not just methods

```csharp
var assemblyName = new AssemblyName("DynamicAssembly");
var assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.Run);
var moduleBuilder = assemblyBuilder.DefineDynamicModule("DynamicModule");

var typeBuilder = moduleBuilder.DefineType("GeneratedClass", TypeAttributes.Public);
var fieldBuilder = typeBuilder.DefineField("_value", typeof(int), FieldAttributes.Private);

var propertyBuilder = typeBuilder.DefineProperty("Value", PropertyAttributes.None, typeof(int), null);
var getterBuilder = typeBuilder.DefineMethod("get_Value", MethodAttributes.Public, typeof(int), Type.EmptyTypes);

var ilGetter = getterBuilder.GetILGenerator();
ilGetter.Emit(OpCodes.Ldarg_0);
ilGetter.Emit(OpCodes.Ldfld, fieldBuilder);
ilGetter.Emit(OpCodes.Ret);

propertyBuilder.SetGetMethod(getterBuilder);

Type generatedType = typeBuilder.CreateType()!;
```

👉 `TypeBuilder` takes it a level further: instead of generating just a standalone method, you build a **complete** class — fields, properties, methods, all in memory, all with manually written IL. `Assembly.LoadFrom` (remember the AssemblyLoadContext post?) loads existing assemblies; `AssemblyBuilder` **creates** a brand-new assembly that never existed on disk

---

# ⚖️ `Reflection.Emit` vs Source Generators: same idea, different moment

| | Source Generators | `Reflection.Emit` |
|---|---|---|
| When it generates code | At **compile** time | At **runtime** |
| Compatible with Native AOT | ✅ yes | ❌ no |
| Can use information only available at runtime | ❌ no | ✅ yes |
| Debuggability | High — generated code is visible as C# | Low — raw IL, hard to inspect |

👉 Remember the source generators post? Both solve "generate code instead of hand-writing it," but at opposite moments of the lifecycle. Whenever the needed information exists at compile time (known types, declared attributes), prefer source generators — faster, more debuggable, and compatible with Native AOT. Reserve `Reflection.Emit` for the rare cases where the decision genuinely can only be made at runtime

---

# ⚠️ Common Mistakes

- Using `Reflection.Emit` for problems a source generator would solve with far less complexity and better debuggability  
- Dynamically generating code on a hot path without caching the result — generation itself has a cost, so the `DynamicMethod`/`TypeBuilder` should be created once and reused  
- Publishing with Native AOT or trimming without knowing `Reflection.Emit` simply doesn't work in those scenarios — the runtime needs to be able to compile IL on the fly, something Native AOT eliminates  
- Hand-writing IL without exhaustive testing — an `OpCode` in the wrong order doesn't produce a compile error, only a failure (or worse, subtly wrong behavior) at runtime  

---

# 📌 Conclusion

- `Reflection.Emit` generates entire methods, types, and assemblies in memory, writing IL directly at runtime  
- The real use case is eliminating the cost of repeated reflection, generating a compiled delegate once  
- `AssemblyBuilder`/`TypeBuilder` let you build complete classes dynamically, not just standalone methods  
- Prefer source generators whenever the information is available at compile time — reserve `Reflection.Emit` for decisions that only exist at runtime  

👉 With runtime code generation mastered, you close the loop on types and reflection — the next step is looking again at a library you've used since the very beginning, but now at its most advanced features: `System.Text.Json`

---

# 🔥 Next Step

Now that you generate code dynamically at runtime, the next level is:

👉 **C# Fundamentals: Advanced System.Text.Json**

Here you'll learn custom converters, serialization source generators, and other advanced features of .NET's native JSON serializer.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
