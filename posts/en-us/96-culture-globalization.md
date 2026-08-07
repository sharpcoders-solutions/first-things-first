# 🧠 C# Fundamentals: Culture and Globalization

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Server GC vs Workstation GC and how to configure the right collection mode  
- `decimal` and number formatting, used without much discussion since the earliest e-commerce posts  

A price formatted as `1,234.56` in the US shows up as `1.234,56` in Brazil — the same number, a different representation. If your application ever has users in more than one country, ignoring this creates subtle, frustrating bugs.

👉 **Let's learn Culture and Globalization in C#**

---

# 💡 What is a `CultureInfo`?

👉 **`CultureInfo` = an object representing a specific region's formatting conventions: decimal separator, date format, currency symbol, and more**

```csharp
var brazilCulture = new CultureInfo("pt-BR");
var usCulture = new CultureInfo("en-US");

decimal price = 1234.56m;

Console.WriteLine(price.ToString("C", brazilCulture)); // R$ 1.234,56
Console.WriteLine(price.ToString("C", usCulture));      // $1,234.56
```

👉 The same `decimal`, the same formatting call — only the `CultureInfo` changes, and the result reflects exactly the convention expected in each region

---

# ⚠️ The danger of `CultureInfo.CurrentCulture`

```csharp
// ❌ Depends on the culture configured on the server/machine where the code runs
decimal value = decimal.Parse("1234.56"); // works on a server with en-US culture...
                                            // ...but breaks or misparses on a server set to a different culture
```

👉 `CultureInfo.CurrentCulture` reflects the **current thread's** culture, which usually comes from the operating system's configuration or the HTTP request — this means the same code can behave completely differently depending on where it runs, one of the sneakiest bugs in applications that parse numbers or dates

---

# 🔒 `CultureInfo.InvariantCulture`: predictability for internal data

```csharp
// ✅ Always behaves the same way, regardless of where the code runs
decimal value = decimal.Parse("1234.56", CultureInfo.InvariantCulture);
string json = value.ToString(CultureInfo.InvariantCulture);
```

👉 **`InvariantCulture` = a "neutral" culture, based on American conventions, that never changes regardless of the environment**

**Practical rule: use `InvariantCulture` for any data that isn't displayed directly to a user** — JSON serialization (remember the `System.Text.Json` post?), cache keys, logs, service-to-service communication, configuration files. Reserve the user's culture exclusively for **display** formatting

---

# 🌐 Detecting the user's culture in an ASP.NET Core API

```csharp
// Program.cs
var localizationOptions = new RequestLocalizationOptions()
    .SetDefaultCulture("en-US")
    .AddSupportedCultures("en-US", "pt-BR", "es-ES")
    .AddSupportedUICultures("en-US", "pt-BR", "es-ES");

app.UseRequestLocalization(localizationOptions);
```

```csharp
[HttpGet("products/{id}")]
public IActionResult GetProduct(int id)
{
    var product = _repository.GetById(id);
    var formattedPrice = product.Price.ToString("C", CultureInfo.CurrentCulture);
    return Ok(new { product.Name, Price = formattedPrice });
}
```

👉 The localization middleware reads the HTTP request's `Accept-Language` header and automatically configures `CultureInfo.CurrentCulture` for that request — each request can have a different culture, without interfering with the others

---

# 📅 Dates and cultures

```csharp
var date = new DateOnly(2026, 3, 15);

Console.WriteLine(date.ToString("d", new CultureInfo("pt-BR"))); // 15/03/2026
Console.WriteLine(date.ToString("d", new CultureInfo("en-US"))); // 3/15/2026
```

👉 Remember the `DateOnly`/`TimeOnly` post? The same date can display in completely different orders (day/month/year vs month/day/year) depending on the culture — another classic source of confusion when ignored

---

# 🔤 String comparison: another culture trap

```csharp
// ❌ Culture-sensitive comparison, can behave unexpectedly
bool equal = "Straße".Equals("STRASSE", StringComparison.CurrentCultureIgnoreCase);

// ✅ Ordinal comparison, predictable, with no linguistic rules
bool equalOrdinal = "abc".Equals("ABC", StringComparison.OrdinalIgnoreCase);
```

👉 **Practical rule: use `StringComparison.Ordinal`/`OrdinalIgnoreCase` for technical comparisons** (keys, identifiers, filenames) **and culture-sensitive comparison only for text genuinely displayed to a user and compared linguistically** — mixing the two is a common source of bugs that are hard to reproduce, because they depend on the culture configured in each environment

---

# ⚠️ Common Mistakes

- Using `CultureInfo.CurrentCulture` (implicitly, without specifying) when `Parse`/`ToString`-ing internal data, breaking when the code runs in an environment with a different culture  
- Comparing strings without specifying `StringComparison`, letting the behavior silently depend on the system's culture  
- Completely ignoring globalization until the product already has international users, forcing a big, risky refactor later  
- Formatting data sent between services (internal APIs, queues) using the user's culture, instead of `InvariantCulture`  

---

# 📌 Conclusion

- `CultureInfo` controls how numbers, dates, and currencies are formatted for display  
- `CurrentCulture` reflects the execution environment — unpredictable for internal data  
- `InvariantCulture` is the correct choice for serialization, logs, and inter-system communication  
- `StringComparison.Ordinal` should be the default for technical, non-linguistic comparisons  

👉 With globalization mastered, the next step is looking at the tools that let you inspect and manipulate C# code itself at runtime — assemblies, metadata, and advanced reflection

---

# 🔥 Next Step

Now that you format data correctly for any region in the world, the next level is:

👉 **C# Fundamentals: Assembly, Metadata, and Reflection in Depth**

Here you'll learn to inspect entire assemblies, read custom metadata, and understand how the runtime organizes compiled types.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
