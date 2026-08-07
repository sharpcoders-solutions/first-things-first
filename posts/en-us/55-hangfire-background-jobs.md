# 🧠 C# Fundamentals: Background Jobs with Hangfire

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Feature flags to control behavior without a new deployment  
- Messaging to decouple systems in time  

Not every task should happen during an HTTP request. Generating a heavy report, sending a bulk email, cleaning up old data — none of that should make the user wait.

👉 **Let's learn to run background tasks with Hangfire**

---

# 💡 The problem with putting everything in the request

```csharp
[HttpPost]
public async Task<IActionResult> GenerateMonthlyReport()
{
    var report = await ProcessWholeMonthData(); // could take minutes
    return Ok(report);
}
```

👉 An HTTP request that takes minutes wrecks the user experience and risks timing out. Some tasks need to exist **outside** the lifecycle of a request

---

# 🏗️ Setting up Hangfire

```bash
dotnet add package Hangfire.AspNetCore
dotnet add package Hangfire.SqlServer
```

```csharp
// Program.cs
builder.Services.AddHangfire(config => config
    .UseSqlServerStorage(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddHangfireServer();

// ...

app.UseHangfireDashboard("/hangfire");
```

👉 Hangfire persists jobs in the database — if the application restarts (remember the Docker post?), pending jobs **aren't lost**, unlike an in-memory queue

---

# 🔥 Fire-and-forget: run once, immediately

```csharp
[HttpPost]
public IActionResult CreateOrder(CreateOrderRequest request)
{
    var order = _service.Create(request);

    BackgroundJob.Enqueue(() => _emailService.SendConfirmation(order.Id));

    return Ok(order); // responds right away, the email runs afterward
}
```

👉 The same spirit as the messaging post: the request responds immediately, and the secondary task runs independently, in another process/thread

---

# ⏰ Recurring jobs

```csharp
RecurringJob.AddOrUpdate(
    "clean-abandoned-carts",
    () => _cartService.CleanAbandoned(),
    Cron.Daily);
```

👉 Replaces the need for OS-level scheduled tasks (cron jobs, Task Scheduler) — the schedule lives inside the C# code itself, versioned along with the rest of the application

---

# ⏳ Delayed and chained jobs

```csharp
// Runs 24 hours from now
BackgroundJob.Schedule(() => _emailService.SendReminder(orderId), TimeSpan.FromHours(24));

// Only runs after the previous job finishes successfully
var jobId = BackgroundJob.Enqueue(() => ProcessPayment(orderId));
BackgroundJob.ContinueJobWith(jobId, () => SendInvoice(orderId));
```

👉 `ContinueJobWith` creates a dependency chain between jobs — useful when a step only makes sense after the previous one finished successfully

---

# 🔁 Automatic retry

```csharp
[AutomaticRetry(Attempts = 3)]
public void ProcessPayment(int orderId)
{
    // if it throws an exception, Hangfire retries automatically
}
```

👉 Remember the Polly post? Here, retry happens at the job level, not the HTTP call — Hangfire retries with automatic backoff, and once attempts are exhausted, marks the job as failed on the dashboard

---

# 📊 The dashboard: visibility with no extra code

Visiting `/hangfire`, you see, without writing any additional code:

- Jobs running, scheduled, and completed  
- Failed jobs, with the full stack trace  
- History of recurring executions  

👉 Combined with the structured logging (Serilog) you already set up, this gives full visibility into what's happening outside the application's synchronous flow

---

# ⚠️ Common Mistakes

- Putting critical, urgent tasks (like validating a real-time payment) into the background, when the user needs an immediate response  
- Not configuring `AutomaticRetry`, letting transient failures kill the job with no second chance  
- Forgetting to protect `/hangfire` with authentication, publicly exposing the dashboard  
- Running long jobs without checking a `CancellationToken`, making graceful application shutdown harder  

---

# 📌 Conclusion

- Background jobs take slow tasks out of the HTTP request's lifecycle  
- Hangfire persists jobs in the database, surviving application restarts  
- Recurring jobs replace external schedulers, with the schedule living in the code itself  
- Automatic retry and job chaining cover most real-world asynchronous processing scenarios  

👉 With Hangfire, your application clearly separates what needs an immediate response from what can (and should) happen in the background

---

# 🔥 Next Step

Now that you know how to process tasks in the background, the next level is:

👉 **C# Fundamentals: Rate Limiting in ASP.NET Core**

Here you'll learn to protect your API against excessive use, whether by mistake or bad intent.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
