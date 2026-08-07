# 🧠 C# Fundamentals: Azure Functions

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- .NET MAUI for cross-platform native apps  
- Continuous deployment of complete .NET applications, always running (containers, VMs)  

Every application you've built throughout this track stays "on" the whole time, waiting for requests. What if you only need to run code sporadically — image processing when a file is uploaded, a task at 3am? Serverless solves this.

👉 **Let's learn Azure Functions**

---

# 💡 What is Serverless?

👉 **Serverless = you write only the function; the cloud handles the server, scales automatically, and you pay only for actual execution time**

```csharp
public class ProcessImage
{
    [Function("ProcessImage")]
    public async Task Run(
        [BlobTrigger("uploads/{name}")] Stream image,
        string name,
        FunctionContext context)
    {
        var logger = context.GetLogger("ProcessImage");
        logger.LogInformation($"Processing {name}");

        // generate thumbnail, resize, etc.
    }
}
```

👉 No `Program.cs` with `WebApplicationBuilder`, no server running 24/7 — the function only executes when an event happens (in this case, a file upload)

---

# 🏗️ Types of triggers

## 🔹 HTTP Trigger

```csharp
[Function("GetOrder")]
public HttpResponseData Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = "orders/{id}")] HttpRequestData request,
    int id)
{
    var response = request.CreateResponse(HttpStatusCode.OK);
    response.WriteAsJsonAsync(new { Id = id, Status = "Confirmed" });
    return response;
}
```

👉 Similar to a REST endpoint (post 34), but without all the ASP.NET Core infrastructure running continuously — it only activates when someone calls it

## 🔹 Timer Trigger

```csharp
[Function("NightlyCleanup")]
public void Run([TimerTrigger("0 0 3 * * *")] TimerInfo timer)
{
    // runs every night at 3am, no need for a server "on" and waiting
}
```

👉 Remember Hangfire (post 55)? This is conceptually similar, but without needing to keep a process running all the time — the cloud "wakes up" the function only at the right moment

## 🔹 Queue Trigger

```csharp
[Function("ProcessOrder")]
public async Task Run([QueueTrigger("pending-orders")] string message)
{
    var order = JsonSerializer.Deserialize<Order>(message);
    // process the order
}
```

👉 Similar to a RabbitMQ consumer (post 44), but without you managing the consumption infrastructure — the function scales automatically as the queue grows

---

# ⚡ Where Native AOT (post 72) connects

```xml
<PropertyGroup>
  <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

👉 Remember the Native AOT post? Cold start is critical in serverless — every new function instance needs to start quickly, and the same fast-startup techniques we discussed in that post apply directly here

---

# 💰 The cost model changes everything

```
Traditional application (Docker/Kubernetes):
  You pay 24/7, even if the application sits idle 90% of the time

Azure Functions:
  You pay only for the milliseconds of actual execution + number of invocations
```

👉 For sporadic or unpredictable workloads, this completely changes the cost equation compared to traditional, always-on continuous deployment approaches

---

# ⚠️ Common Mistakes

- Using Functions for constant, predictable workloads, where a traditional application would be cheaper  
- Not accounting for cold start in latency-sensitive scenarios — the first invocation after an idle period is slower  
- Writing functions with shared in-memory state, expecting it to persist between executions (each invocation can run on a different instance)  
- Ignoring execution time limits — functions have a timeout, they're not suited for long-running processing without breaking it into steps  

---

# 📌 Conclusion

- Serverless eliminates the need to manage a server, scaling automatically per event  
- HTTP, Timer, and Queue triggers cover the most common activation patterns  
- The pay-per-execution cost model favors sporadic or unpredictable workloads  
- Native AOT and fast cold-start techniques are especially relevant here  

👉 With Azure Functions, you write only the business logic, leaving all the scale and availability infrastructure to the cloud

---

# 🔥 Next Step

Now that you know serverless on Azure, the next level is:

👉 **C# Fundamentals: AWS Lambda with .NET**

Here you'll learn the same serverless philosophy, now on the world's largest public cloud.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
