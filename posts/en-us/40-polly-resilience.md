# 🧠 C# Fundamentals: Resilience with Polly

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- In-memory and distributed caching  
- Health checks for your dependencies  

You already know how to **detect** when a dependency is failing. But detecting isn't the same as **handling** the failure well. An unstable external API, a network that's slow for a few seconds — that's normal, and your application needs to survive it without breaking.

👉 **That's exactly what resilience is for, and Polly is the standard library in the .NET ecosystem**

---

# 💡 What is software resilience?

👉 **Resilience = the application's ability to keep working (or fail gracefully) in the face of temporary failures**

Transient failures are inevitable: network timeouts, a momentarily overloaded service, connection instability. The question isn't **whether** they'll happen, it's **how your application reacts** when they do.

---

# 🏗️ Installing Polly

```bash
dotnet add package Microsoft.Extensions.Http.Polly
```

```csharp
// Program.cs
builder.Services.AddHttpClient<ExternalApiClient>()
    .AddPolicyHandler(GetRetryPolicy());

static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)));
}
```

👉 Polly integrates directly with `HttpClient` via dependency injection — the same configuration you've already seen in previous posts, now protecting external HTTP calls

---

# 🔁 Retry: trying again

```csharp
var policy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(attempt * 2));

await policy.ExecuteAsync(() => _httpClient.GetAsync("/api/data"));
```

## 🔹 Exponential backoff

👉 Instead of retrying immediately (which can make an already overloaded service worse), the interval between attempts **grows**: 2s, 4s, 8s... giving the service time to recover

---

# ⚡ Circuit Breaker: stopping the insistence

```csharp
var circuitBreaker = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(
        handledEventsAllowedBeforeBreaking: 5,
        durationOfBreak: TimeSpan.FromSeconds(30));
```

## 🔹 How it works

- After **5 consecutive failures**, the circuit "opens" — new calls fail **immediately**, without even trying  
- After 30 seconds, the circuit enters test mode, allowing one call through to see if the service recovered  

👉 Without a circuit breaker, your application would keep hammering an already-down service, making things worse. With it, you stop knocking on a door you know won't answer — and your own application stays more responsive, since it's no longer stuck waiting on repeated timeouts

---

# ⏱️ Timeout: not waiting forever

```csharp
var timeout = Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(5));
```

👉 Without an explicit timeout, a slow dependency can hang your application indefinitely — remember the async/await post? A thread stuck waiting is a thread not serving other users

---

# 🔗 Combining policies

```csharp
var combinedPolicy = Policy.WrapAsync(
    circuitBreaker,
    retryPolicy,
    timeout);

await combinedPolicy.ExecuteAsync(() => _httpClient.GetAsync("/api/data"));
```

👉 The policies combine: timeout limits each attempt, retry tries again on failure, and circuit breaker prevents repeated attempts against a service already known to be unavailable

---

# 🛟 Fallback: having a plan B

```csharp
var fallback = Policy<List<Product>>
    .Handle<Exception>()
    .FallbackAsync(new List<Product>()); // returns an empty list instead of crashing

var products = await fallback.ExecuteAsync(() => FetchProductsFromExternalApi());
```

👉 Not every failure needs to turn into an error for the end user. A fallback returns a default result (an empty list, a cached value, a generic response) when the main operation fails

---

# ⚠️ Common Mistakes

- Retrying errors that **aren't transient** (e.g., a `400 Bad Request` will never succeed just by trying again)  
- Configuring retries with no backoff, hammering an already unstable service even faster  
- Not combining timeout with retry, letting each attempt hang indefinitely  
- Setting the circuit breaker's thresholds so low that the circuit opens over normal, momentary instability  

---

# 📌 Conclusion

- Resilience is about handling transient failures well, not avoiding them  
- Retry with exponential backoff gives the service time to recover  
- Circuit breaker avoids repeatedly hitting a service already known to be down  
- Timeout prevents a slow dependency from hanging your application indefinitely  
- Fallback guarantees a plan B when the main operation fails  

👉 With Polly, your application stops assuming everything will go fine and starts explicitly preparing for when it doesn't

---

# 🔥 Next Step

Now that your application handles failures well, the next level is:

👉 **C# Fundamentals: Messaging with RabbitMQ**

Here you'll learn to decouple systems in time, using message queues instead of direct, synchronous calls.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
