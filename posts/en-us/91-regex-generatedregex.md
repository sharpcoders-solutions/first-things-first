# 🧠 C# Fundamentals: Regex and GeneratedRegex

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- `required` and `init` for guaranteeing correctly initialized objects  
- Source generators (post 68), generating code at compile time  

You've probably used `Regex` to validate an email or extract a text pattern, without thinking much about the cost behind it. There's a classic performance gotcha in `Regex`, and since C# 11 there's a completely different (and better) way to handle it.

👉 **Let's dig into Regex, and the `GeneratedRegex` source generator**

---

# 💡 The basics: `Regex` for validating and extracting patterns

```csharp
using System.Text.RegularExpressions;

var regex = new Regex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$");

bool valid = regex.IsMatch("user@example.com"); // true
```

```csharp
var phoneRegex = new Regex(@"\((\d{3})\)\s?(\d{3})-(\d{4})");
var match = phoneRegex.Match("(415) 555-0123");

if (match.Success)
{
    var areaCode = match.Groups[1].Value; // "415"
    var prefix = match.Groups[2].Value;   // "555"
    var lineNumber = match.Groups[3].Value; // "0123"
}
```

👉 Capture groups (the parentheses in the pattern) let you extract specific parts of a text that matches the pattern — not just knowing it matches, but pulling out the relevant pieces

---

# 💸 The performance problem: regex compilation is expensive

```csharp
public bool ValidateEmail(string email)
{
    var regex = new Regex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$"); // ❌ recompiles the pattern EVERY time
    return regex.IsMatch(email);
}
```

👉 Creating a `new Regex(pattern)` involves interpreting the pattern and building an internal state machine — work that is **not** free. Doing this inside a frequently called method (validation on every API request, for example) wastes CPU repeating the same compilation work unnecessarily

---

# 🏗️ The classic solution: `static readonly Regex`

```csharp
public class Validator
{
    private static readonly Regex EmailRegex =
        new Regex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$", RegexOptions.Compiled);

    public bool ValidateEmail(string email) => EmailRegex.IsMatch(email);
}
```

👉 Storing the instance in a `static readonly` field guarantees the pattern gets compiled **exactly once**, when the class is initialized. `RegexOptions.Compiled` goes further, generating optimized IL specifically for that pattern — faster at execution, but with an even higher initialization cost

---

# ⚡ `GeneratedRegex`: compiled at build time, not runtime

```csharp
public partial class Validator
{
    [GeneratedRegex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$")]
    private static partial Regex EmailRegex();

    public bool ValidateEmail(string email) => EmailRegex().IsMatch(email);
}
```

👉 **`[GeneratedRegex]` = an attribute that instructs a source generator (remember post 68?) to generate, at **compile** time, all the C# code needed for that pattern — with nothing interpreted at runtime**

Unlike `RegexOptions.Compiled` (which dynamically generates IL on first execution), `GeneratedRegex` already delivers fully ready code in the compiled assembly — zero initialization cost, and the pattern is validated **at compile time**, catching regex syntax errors before the code even runs

---

# 🔍 Anatomy of `GeneratedRegex`

```csharp
public partial class Validator
{
    [GeneratedRegex(@"\((\d{3})\)\s?(\d{3})-(\d{4})", RegexOptions.IgnoreCase)]
    private static partial Regex PhoneRegex();
}
```

👉 The class needs to be `partial`, the method needs to be `partial` and `static`, and the `[GeneratedRegex]` attribute takes the pattern (and optionally `RegexOptions`) as an argument — the source generator produces the full implementation under the hood, visible as generated code if you inspect the build output

---

# ⚖️ When to use each approach

| | `new Regex(pattern)` | `static readonly` + `Compiled` | `[GeneratedRegex]` |
|---|---|---|---|
| Pattern used once, simple script | ✅ ok | Unnecessary | Unnecessary |
| Pattern used frequently in production | ❌ waste | ✅ good | ✅ better |
| .NET 7+ available | — | — | ✅ recommended |
| Pattern only known at runtime (variable) | ✅ only option | ✅ only option | ❌ not supported |

👉 **Practical rule: if the pattern is a literal known at compile time and the project runs on .NET 7 or later, use `[GeneratedRegex]` by default.** Only fall back to dynamic `new Regex()` when the pattern genuinely comes from an external source (configuration, database) and can't be known at compile time

---

# ⚠️ Common Mistakes

- Creating a `new Regex(pattern)` inside a frequently called method, unnecessarily recompiling the pattern on every call  
- Using `RegexOptions.Compiled` on rarely used patterns, paying a higher initialization cost with no real gain  
- Writing overly complex regex patterns, when a simple `string.Contains`/`StartsWith` check would solve it far more clearly  
- Ignoring `[GeneratedRegex]` in projects already on .NET 7+, missing out on a nearly free optimization  

---

# 📌 Conclusion

- `Regex` interprets and compiles the pattern at runtime, a real cost if repeated unnecessarily  
- `static readonly Regex` with `RegexOptions.Compiled` compiles once, at initialization  
- `[GeneratedRegex]` uses a source generator to compile the pattern at **build** time, with zero runtime cost  
- Use `GeneratedRegex` as the default on .NET 7+; reserve dynamic `new Regex()` for patterns only known at runtime  

👉 With optimized regex, the next step is looking at another library you've already used since the earliest API posts: `System.Text.Json`, and the advanced features that go far beyond basic `Serialize`/`Deserialize`

---

# 🔥 Next Step

Now that you've mastered regular expressions with maximum performance, the next level is:

👉 **C# Fundamentals: Advanced System.Text.Json**

Here you'll learn custom converters, serialization source generators, and other advanced features of .NET's native JSON serializer.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
