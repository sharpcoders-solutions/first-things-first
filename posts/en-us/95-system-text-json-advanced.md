# 🧠 C# Fundamentals: Advanced System.Text.Json

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Reflection.Emit and dynamic runtime code generation (IL Emit)  
- Basic `System.Text.Json`, used implicitly since the very first API post  

Every API you've built throughout this track serializes and deserializes JSON automatically, without you thinking much about it. But `System.Text.Json` has an advanced layer — custom converters and its own source generator — that solves real control and performance problems.

👉 **Let's explore advanced `System.Text.Json`**

---

# 💡 Recap: the basics you already use without noticing

```csharp
public record Product(int Id, string Name, decimal Price);

string json = JsonSerializer.Serialize(new Product(1, "Laptop", 3500m));
var product = JsonSerializer.Deserialize<Product>(json);
```

👉 ASP.NET Core already uses `System.Text.Json` under the hood in every `[HttpPost]`/`[HttpGet]` you've written — the `record` (post 30) maps naturally to JSON, field by field

---

# 🎨 Custom converters: controlling serialization manually

```csharp
public class MoneyConverter : JsonConverter<decimal>
{
    public override decimal Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        var text = reader.GetString();
        return decimal.Parse(text!.Replace("$", "").Replace(",", ""), CultureInfo.InvariantCulture);
    }

    public override void Write(Utf8JsonWriter writer, decimal value, JsonSerializerOptions options)
    {
        writer.WriteStringValue($"${value:N2}");
    }
}
```

```csharp
public record Product(int Id, string Name, [property: JsonConverter(typeof(MoneyConverter))] decimal Price);

// Serializes Price as "$3,500.00" instead of 3500.00
```

👉 **`JsonConverter<T>` = full control over how a specific type is read from and written to JSON**, essential when the default format doesn't match what an external API expects, or when you need a custom representation (like formatting money with a currency symbol)

---

# 🔧 Configuring global serialization options

```csharp
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    WriteIndented = true,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};

string json = JsonSerializer.Serialize(product, options);
```

```csharp
// Program.cs — applying it globally in the API
builder.Services.Configure<Microsoft.AspNetCore.Http.Json.JsonOptions>(options =>
{
    options.SerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
});
```

👉 `PropertyNamingPolicy` resolves the common gap between C# convention (`PascalCase`) and JavaScript convention (`camelCase`), without needing to rename any of your C# class properties

---

# ⚡ `JsonSerializerContext`: a serialization source generator

```csharp
[JsonSerializable(typeof(Product))]
[JsonSerializable(typeof(List<Product>))]
public partial class JsonContext : JsonSerializerContext { }
```

```csharp
string json = JsonSerializer.Serialize(product, JsonContext.Default.Product);
var readProduct = JsonSerializer.Deserialize(json, JsonContext.Default.Product);
```

👉 **`JsonSerializerContext` = a source generator (remember post 71 and `GeneratedRegex`?) that generates, at compile time, all the serialization code for the declared types**

Without this feature, `System.Text.Json` uses **reflection** (post 70) to discover properties at runtime — it works, but has a performance cost and, more importantly, **doesn't work with Native AOT** (post 72), because reflecting over types that no longer exist as metadata simply fails

---

# 🚀 Why this matters: Native AOT requires the source generator

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
  <JsonSerializerIsReflectionEnabledByDefault>false</JsonSerializerIsReflectionEnabledByDefault>
</PropertyGroup>
```

👉 Remember the Native AOT post? When you compile a native executable, reflection metadata gets aggressively stripped to reduce size and startup time. `JsonSerializerContext` generates code that **doesn't depend on reflection**, making JSON serialization compatible with Native AOT — without it, your Native AOT app simply breaks when trying to serialize

---

# 🔍 Custom property names and order

```csharp
public record Product(
    [property: JsonPropertyName("product_id")] int Id,
    [property: JsonPropertyName("product_name")] string Name,
    [property: JsonPropertyOrder(1)] decimal Price
);
```

👉 `JsonPropertyName` overrides a specific property's name in the JSON, useful when you need to follow an external API's convention (like `snake_case`) without renaming the corresponding C# property

---

# ⚠️ Common Mistakes

- Writing a custom `JsonConverter<T>` for a problem `PropertyNamingPolicy` or `JsonPropertyName` would already solve more simply  
- Publishing with Native AOT without configuring `JsonSerializerContext`, causing serialization failures at runtime that only surface after deploy  
- Ignoring `JsonIgnoreCondition.WhenWritingNull`, sending unnecessary `null` fields in API responses  
- Mixing reflection-based and source-generator-based serialization in the same project without understanding each one's performance implications  

---

# 📌 Conclusion

- `JsonConverter<T>` gives full control over how a specific type gets serialized  
- `JsonSerializerOptions` configures global behavior, like `camelCase` and indentation  
- `JsonSerializerContext` generates serialization code at compile time, without reflection  
- Native AOT requires the source generator — reflection-based serialization doesn't work in that scenario  

👉 With JSON mastered in depth, the next step is going even lower in the stack: how to interact with native libraries written in C/C++ from C#

---

# 🔥 Next Step

Now that you've mastered advanced JSON serialization, the next level is:

👉 **C# Fundamentals: P/Invoke and Native Interoperability**

Here you'll learn to call native C/C++ libraries directly from C#, going beyond what you saw in the unsafe code post.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
