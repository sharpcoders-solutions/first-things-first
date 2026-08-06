# 🧠 C# Fundamentals: Rate Limiting in ASP.NET Core

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Background jobs for tasks outside the request cycle  
- Security following the OWASP Top 10  

Your API is protected against classic vulnerabilities, but there's still a problem: nothing stops a single client (by mistake or bad intent) from making thousands of requests per second and taking the service down for everyone.

👉 **Let's learn to limit request rates with Rate Limiting**

---

# 💡 What is Rate Limiting?

👉 **Rate Limiting = restricting how many requests a client can make within a time window**

Without it, your API is exposed to:

- Bugs in clients that accidentally generate request loops  
- Brute-force attacks against login (remember the JWT post?)  
- Aggressive scraping consuming all your server resources  

---

# 🏗️ Setting up ASP.NET Core's built-in rate limiting

```csharp
// Program.cs
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("default", opts =>
    {
        opts.PermitLimit = 100;
        opts.Window = TimeSpan.FromMinutes(1);
        opts.QueueLimit = 0;
    });

    options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
});

// ...

app.UseRateLimiter();
```

```csharp
[HttpGet]
[EnableRateLimiting("default")]
public IActionResult GetAll() => Ok(/* ... */);
```

👉 `429 Too Many Requests` is the correct HTTP status code for signaling this limit — remember the post about correct HTTP responses? This is another status code that's part of your API's contract

---

# 🪟 Limiting strategies

## 🔹 Fixed Window

```csharp
options.AddFixedWindowLimiter("default", opts =>
{
    opts.PermitLimit = 100;
    opts.Window = TimeSpan.FromMinutes(1);
});
```

👉 100 requests per minute, resetting at the start of each window — simple, but can allow spikes at window boundaries (100 at the end of one minute + 100 at the start of the next)

## 🔹 Sliding Window

```csharp
options.AddSlidingWindowLimiter("default", opts =>
{
    opts.PermitLimit = 100;
    opts.Window = TimeSpan.FromMinutes(1);
    opts.SegmentsPerWindow = 4;
});
```

👉 Splits the window into segments, smoothing out the spikes that Fixed Window allows

## 🔹 Token Bucket

```csharp
options.AddTokenBucketLimiter("default", opts =>
{
    opts.TokenLimit = 100;
    opts.TokensPerPeriod = 20;
    opts.ReplenishmentPeriod = TimeSpan.FromSeconds(10);
});
```

👉 Allows controlled bursts (using accumulated tokens), replenishing gradually — ideal for naturally "bursty" traffic, like users opening several tabs at once

## 🔹 Concurrency Limiter

```csharp
options.AddConcurrencyLimiter("uploads", opts =>
{
    opts.PermitLimit = 5;
});
```

👉 Doesn't limit by time, but by **simultaneous quantity** — great for expensive endpoints, like uploading large files

---

# 🎯 Limiting per user, not just globally

```csharp
options.AddPolicy("per-user", context =>
{
    var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value ?? "anonymous";

    return RateLimitPartition.GetFixedWindowLimiter(userId, _ => new FixedWindowRateLimiterOptions
    {
        PermitLimit = 50,
        Window = TimeSpan.FromMinutes(1)
    });
});
```

👉 Combined with the `User` you've been reading since the JWT post, each authenticated user gets their own quota — one abusive user doesn't consume everyone else's limit

---

# ⚠️ Common Mistakes

- Applying a single global limit, letting one abusive user hurt everyone else  
- Setting limits so low that legitimate users get blocked during normal use  
- Not differentiating critical endpoints (login) from lightweight ones (public listing), applying the same rule to all of them  
- Forgetting to log when a limit is hit, losing visibility into possible attacks  

---

# 📌 Conclusion

- Rate Limiting protects the API against excessive use, intentional or accidental  
- `429 Too Many Requests` is the correct HTTP status code for signaling the limit  
- Fixed Window, Sliding Window, Token Bucket, and Concurrency Limiter cover different traffic patterns  
- Limiting per user (not just globally) keeps one client from hurting all the others  

👉 With Rate Limiting, your API protects itself from its own success — high traffic stops being a risk of taking the service down

---

# 🔥 Next Step

Now that your API resists excessive use, the next level is:

👉 **C# Fundamentals: API Gateway with YARP**

Here you'll learn to centralize rate limiting, authentication, and routing in front of multiple services.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
