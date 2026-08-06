# 🧠 C# Fundamentals: Options Pattern and Advanced Configuration

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Nullable Reference Types in depth  
- Basic `appsettings.json`, used since the feature flags post (51)  

You've read configuration with `builder.Configuration["Key"]` a few times throughout this track. That works, but it's fragile — no type checking, no autocomplete, errors only show up at runtime. The Options Pattern fixes this.

👉 **Let's learn the Options Pattern**

---

# 💡 The problem with weakly-typed configuration

```csharp
// ❌ Fragile: magic string, no type checking, no autocomplete
var timeout = int.Parse(builder.Configuration["Api:TimeoutSeconds"]);
```

👉 If someone renames the key in `appsettings.json` without updating this string, the error only shows up at runtime — nothing in the compiler warns you

---

# 🏗️ Setting up the Options Pattern

```json
// appsettings.json
{
  "ApiConfiguration": {
    "TimeoutSeconds": 30,
    "BaseUrl": "https://api.example.com",
    "MaxRetries": 3
  }
}
```

```csharp
public class ApiConfiguration
{
    public const string Section = "ApiConfiguration";

    public int TimeoutSeconds { get; set; }
    public string BaseUrl { get; set; } = default!;
    public int MaxRetries { get; set; }
}
```

```csharp
// Program.cs
builder.Services.Configure<ApiConfiguration>(
    builder.Configuration.GetSection(ApiConfiguration.Section));
```

👉 Strongly typed (remember the NRT post?), with autocomplete and compile-time type checking — a "wrong key" error becomes a compile error, not a production error

---

# 🎯 Consuming via dependency injection

```csharp
public class ApiClient
{
    private readonly ApiConfiguration _configuration;

    public ApiClient(IOptions<ApiConfiguration> options)
    {
        _configuration = options.Value;
    }

    public async Task<string> Fetch(string path)
    {
        using var http = new HttpClient { BaseAddress = new Uri(_configuration.BaseUrl) };
        http.Timeout = TimeSpan.FromSeconds(_configuration.TimeoutSeconds);

        var response = await http.GetAsync(path);
        return await response.Content.ReadAsStringAsync();
    }
}
```

👉 The same dependency injection pattern you've used since the ASP.NET Core post — `IOptions<T>` is just another dependency resolved by the container

---

# 🔄 IOptionsSnapshot: configuration that reloads

```csharp
public class ReloadableService
{
    private readonly IOptionsSnapshot<ApiConfiguration> _options;

    public ReloadableService(IOptionsSnapshot<ApiConfiguration> options)
    {
        _options = options;
    }

    public void Process()
    {
        // always gets the latest value from appsettings.json, without restarting the app
        Console.WriteLine(_options.Value.TimeoutSeconds);
    }
}
```

👉 `IOptions<T>` is resolved once, at startup. `IOptionsSnapshot<T>` reloads the value on every request — combined with `reloadOnChange: true` in `appsettings.json`, you change configuration without restarting the application, similar in spirit to the feature flags from post 51

---

# ✅ Validating configuration at startup

```csharp
builder.Services.AddOptions<ApiConfiguration>()
    .Bind(builder.Configuration.GetSection(ApiConfiguration.Section))
    .Validate(config => config.MaxRetries > 0, "MaxRetries must be greater than zero")
    .ValidateOnStart();
```

👉 `ValidateOnStart()` fails the application immediately at startup if the configuration is invalid — much better than discovering in production, three calls later, that `MaxRetries` was zero

---

# ⚠️ Common Mistakes

- Continuing to use `Configuration["Key"]` directly in new code, ignoring the benefits of strong typing  
- Using `IOptions<T>` when configuration needs to reload at runtime — `IOptionsSnapshot<T>` is the right choice there  
- Not validating configuration at startup, letting invalid values crash the application only when used  
- Mixing secrets (connection strings, API keys) directly into a versioned `appsettings.json` — use User Secrets or environment variables for that  

---

# 📌 Conclusion

- Options Pattern turns weakly-typed configuration into strongly-typed classes  
- `IOptions<T>` resolves once; `IOptionsSnapshot<T>` reloads on every request  
- `ValidateOnStart()` fails fast, at startup, instead of silently in production  
- The same familiar dependency injection mechanism makes consumption natural  

👉 With the Options Pattern, configuration stops being a set of magic strings and becomes a first-class citizen of your type system

---

# 🔥 Next Step

Now that you've mastered advanced configuration, the next level is:

👉 **C# Fundamentals: Multi-tenancy**

Here you'll learn to serve multiple isolated customers from a single application.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
