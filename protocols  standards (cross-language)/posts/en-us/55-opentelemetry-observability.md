# 🧠 C# Fundamentals: Observability with OpenTelemetry

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- API Gateway as a single entry point  
- Structured logging with Serilog  
- Health Checks for monitoring availability  

Each isolated piece (logs, metrics, health checks) tells part of the story. But when a request flows through the gateway, across three microservices, and into a database, you need to see the whole path — not scattered fragments.

👉 **Let's learn Observability with OpenTelemetry**

---

# 💡 The three pillars of observability

👉 **Observability = being able to understand what's happening inside the system by looking from outside**

- **Logs** — discrete events ("order created", "payment failed")  
- **Metrics** — aggregated numbers over time (requests/second, average latency)  
- **Traces** — the complete path of a request across multiple services  

OpenTelemetry (OTel) is the open standard that unifies all three, without locking you into a specific vendor.

---

# 🏗️ Setting up OpenTelemetry

```bash
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

```csharp
// Program.cs
builder.Services.AddOpenTelemetry()
    .ConfigureResource(resource => resource.AddService("orders-service"))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddOtlpExporter())
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddRuntimeInstrumentation()
        .AddOtlpExporter());
```

👉 `AddOtlpExporter` sends data to a collector (like Jaeger, Grafana Tempo, or Application Insights) — the C# code doesn't need to know where the data ends up

---

# 🔗 Distributed traces: following the request

```csharp
using var activity = _activitySource.StartActivity("ProcessOrder");
activity?.SetTag("order.id", orderId);

var stock = await _stockClient.CheckAvailability(orderId);
var payment = await _paymentClient.Process(orderId);
```

👉 Each `Activity` (span) represents a step. When the orders service calls the inventory service over HTTP, the `TraceId` automatically travels in the request header — in the collector, you see the whole tree:

```
ProcessOrder (orders-service) — 340ms
  └─ CheckAvailability (inventory-service) — 120ms
  └─ ProcessPayment (payment-service) — 180ms
```

👉 Remember gRPC and RabbitMQ? A distributed trace shows exactly where those 340ms were spent, even across three different services

---

# 📊 Custom metrics

```csharp
var meter = new Meter("OrdersService");
var ordersCounter = meter.CreateCounter<int>("orders.created");

ordersCounter.Add(1, new KeyValuePair<string, object?>("status", "success"));
```

👉 Unlike logs (individual events), metrics are aggregated — "how many orders were created in the last 5 minutes" is a metrics question, not a log question

---

# 🔍 Correlating logs, metrics, and traces

```csharp
builder.Logging.AddOpenTelemetry(opts =>
{
    opts.IncludeFormattedMessage = true;
    opts.IncludeScopes = true;
});
```

👉 Combined with Serilog (post 37), every log line automatically gains the current request's `TraceId` — when investigating an error, you jump straight from the log to the full trace that caused it

---

# ⚠️ Common Mistakes

- Instrumenting only one service and thinking you already have observability — the real value is seeing the **entire** chain  
- Not propagating trace context across asynchronous calls (queues, jobs), breaking the tracing chain  
- Creating high-cardinality metrics (like using the user ID as a tag), blowing up storage costs  
- Confusing observability with monitoring — monitoring tells you something broke, observability helps you understand **why**  

---

# 📌 Conclusion

- Observability combines logs, metrics, and traces to understand distributed systems  
- OpenTelemetry is the open, vendor-independent standard for instrumentation  
- Distributed traces show the complete path of a request across multiple services  
- Correlating logs with `TraceId` dramatically speeds up problem investigation  

👉 With OpenTelemetry, distributed systems stop being a black box and start telling their own story

---

# 🔥 Next Step

Now that you can trace requests across services, the next level is:

👉 **C# Fundamentals: Event Sourcing — Introduction**

Here you'll learn to store the complete history of state changes, not just the current state.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
