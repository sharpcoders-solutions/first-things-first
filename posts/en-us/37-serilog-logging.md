# 🧠 C# Fundamentals: Structured Logging with Serilog

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to automate build, test, and deploy with CI/CD  
- How to containerize and publish your application  

Your API is live, publishing itself. But when something goes wrong at three in the morning, how do you find out **what**, **where**, and **why**?

👉 **That's exactly what structured logging is for**

---

# 💡 `Console.WriteLine` isn't enough

You've used `Console.WriteLine` since the first program post — great for learning, terrible for production:

- No severity levels (everything looks the same: error, warning, information)  
- Not searchable or filterable  
- Disappears when the container restarts, unless it's sent somewhere persistent  

👉 **Structured logging** fixes this: every log entry becomes an event with searchable data, not just a loose line of text

---

# 🏗️ Setting up Serilog

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

```csharp
// Program.cs
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

builder.Host.UseSerilog();
```

## 🔹 The "sink" concept

👉 **Sink = where the log gets sent**

Console, file, database, or services like Seq, Elasticsearch, and Application Insights — you can send the same log to multiple destinations at once, without changing a single line of the code that generates the log

---

# 🎯 Log levels

```csharp
_logger.LogTrace("Extremely granular detail");
_logger.LogDebug("Useful information during development");
_logger.LogInformation("Order {OrderId} processed successfully", order.Id);
_logger.LogWarning("Low stock for product {ProductId}", product.Id);
_logger.LogError(ex, "Failed to process payment for order {OrderId}", order.Id);
_logger.LogCritical("Database unreachable");
```

## 🔹 When to use each level

- `Trace`/`Debug` → details only relevant during development  
- `Information` → normal events in the application's flow  
- `Warning` → something unexpected, but nothing broke  
- `Error` → an operation failed (usually paired with an exception)  
- `Critical` → the application is at risk of going down  

👉 In production, you'll usually set the minimum level to `Information`, silencing `Trace` and `Debug` so they don't generate noise

---

# 🧩 True structured logging: the `{placeholders}`

```csharp
_logger.LogInformation("Order {OrderId} processed successfully", order.Id);
```

👉 Notice this is **not** string interpolation (`$"Order {order.Id}"`). The `{OrderId}` becomes a **searchable property** in the log, not just formatted text

This enables queries like "show me every log where `OrderId = 42`" in observability tools — something impossible if everything turned into a loose string

---

# 🔌 Injecting the logger via DI

```csharp
public class ProductsController : ControllerBase
{
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(ILogger<ProductsController> logger)
    {
        _logger = logger;
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        _logger.LogInformation("Fetching product {ProductId}", id);

        var product = _repository.GetById(id);

        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }

        return Ok(product);
    }
}
```

👉 `ILogger<T>` already comes registered in ASP.NET Core's dependency injection container — once again, the same DI mechanism you've mastered since the API-building post

---

# 🌐 Request logging middleware

```csharp
app.UseSerilogRequestLogging();
```

👉 A single line automatically logs **every** HTTP request: method, route, response status, and execution time — no need to add manual logging to every endpoint

---

# ⚠️ Common Mistakes

- Using string interpolation (`$"..."`) instead of placeholders, losing the log's searchable structure  
- Logging sensitive data (passwords, tokens, card numbers) without realizing it  
- Leaving the minimum level as `Debug` in production, generating unnecessary log volume  
- Catching an exception and logging nothing — the error simply disappears without a trace  

---

# 📌 Conclusion

- Structured logging turns logs into searchable data, not just text  
- Log levels (`Information`, `Warning`, `Error`...) classify each event's severity  
- `{Placeholders}` become queryable properties, unlike string interpolation  
- `ILogger<T>` is injected via DI, exactly like any other dependency in your application  

👉 With structured logging, your application stops being a black box and starts telling you, in detail, what's happening inside

---

# 🔥 Next Step

Now that you can see what your application is doing, the next level is:

👉 **C# Fundamentals: Health Checks and Monitoring**

Here you'll learn to make your application report, in an automated way, whether it (and its dependencies) are healthy.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
