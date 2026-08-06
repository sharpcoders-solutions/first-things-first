# 🧠 C# Fundamentals: Health Checks and Monitoring

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Structured logging with Serilog  
- CI/CD and automated deployment  

Your logs tell the story after something has already happened. But how does an orchestrator (Kubernetes, Azure, a load balancer) know, **right now**, whether your application is healthy enough to receive traffic?

👉 **That's exactly what health checks are for**

---

# 💡 What is a Health Check?

👉 **Health check = an endpoint that reports whether the application (and its dependencies) is working**

Without one, an orchestrator only knows the process is "running" — it doesn't know if the database is reachable, if an external API is down, or if the application is silently stuck.

---

# 🏗️ Setting up the basics

```csharp
// Program.cs
builder.Services.AddHealthChecks();

// ...

app.MapHealthChecks("/health");
```

```bash
curl http://localhost:8080/health
# Healthy
```

👉 Just this configuration already exposes an endpoint that responds `200 OK` with `Healthy` as long as the application is up — simple, but it still doesn't check anything beyond the process itself running

---

# 🗄️ Checking real dependencies

The real value shows up when you check external dependencies:

```bash
dotnet add package AspNetCore.HealthChecks.SqlServer
dotnet add package AspNetCore.HealthChecks.Redis
```

```csharp
builder.Services.AddHealthChecks()
    .AddSqlServer(connectionString, name: "database")
    .AddRedis(redisConnectionString, name: "cache")
    .AddUrlGroup(new Uri("https://external-api.com/status"), name: "external-api");
```

👉 Now the health check reflects the system's real health: if the database goes down, the `/health` endpoint reports it automatically — with no manual verification logic written by you

---

# 📊 Detailed JSON responses

```csharp
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, result) =>
    {
        context.Response.ContentType = "application/json";

        var response = new
        {
            status = result.Status.ToString(),
            checks = result.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description
            })
        };

        await context.Response.WriteAsync(JsonSerializer.Serialize(response));
    }
});
```

```json
{
  "status": "Unhealthy",
  "checks": [
    { "name": "database", "status": "Healthy", "description": null },
    { "name": "cache", "status": "Unhealthy", "description": "Connection timeout" }
  ]
}
```

👉 Instead of a simple "works or doesn't," now you know **exactly which** dependency is failing

---

# 🔀 Liveness vs Readiness: the distinction environments like Kubernetes require

## 🔹 Liveness — "is the application alive?"

```csharp
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false // doesn't check external dependencies
});
```

👉 If this fails, the orchestrator **restarts** the container — used to detect internal hangs

## 🔹 Readiness — "is the application ready to receive traffic?"

```csharp
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});
```

👉 If this fails, the orchestrator **stops sending traffic** to that container, without necessarily restarting it — used when an external dependency (database, cache) is temporarily unavailable

**Practical rule:** liveness checks "does the process hang?"; readiness checks "am I ready to serve requests right now?". Mixing the two up can make the orchestrator restart containers unnecessarily during a momentary dependency hiccup.

---

# ⚠️ Common Mistakes

- Making the liveness check verify external dependencies, causing cascading restarts when the database gets slow  
- Not setting up health checks and only discovering infrastructure problems when users complain  
- Publicly exposing the `/health` endpoint with sensitive details about internal infrastructure  
- Ignoring the health check result in the CI/CD pipeline, publishing a version that already starts with a broken dependency  

---

# 📌 Conclusion

- Health checks report, in real time, whether the application and its dependencies are healthy  
- Checking real dependencies (database, cache, external APIs) is what gives the endpoint real value  
- Liveness detects internal hangs; Readiness detects whether the application can receive traffic right now  
- Detailed JSON responses make it easier to pinpoint exactly what's failing  

👉 With health checks, your infrastructure stops guessing whether the application is okay and starts **knowing**

---

# 🔥 Next Step

Now that your application reports its own health, the next level is:

👉 **C# Fundamentals: Caching in C# (In-Memory and Distributed with Redis)**

Here you'll learn to reduce database load and speed up responses by storing frequently accessed data.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
