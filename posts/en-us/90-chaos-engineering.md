# 🧠 C# Fundamentals: Chaos Engineering

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Blue-Green and Canary for reducing deployment risk  
- Polly (post 40) for failure resilience — retry, circuit breaker, timeout  

You configured Polly to handle failures. But how do you know that configuration actually works under real failure conditions, before a production incident becomes the first test? Chaos Engineering solves this.

👉 **Let's learn Chaos Engineering**

---

# 💡 The problem with never testing real failures

```csharp
// Configured since post 40, but... does it actually work?
services.AddResiliencePipeline("default", builder =>
{
    builder.AddRetry(new RetryStrategyOptions { MaxRetryAttempts = 3 });
    builder.AddCircuitBreaker(new CircuitBreakerStrategyOptions());
});
```

👉 You wrote the resilience configuration, unit tests pass (post 30), but you've never seen the real behavior when the payment service actually goes down in production, with real traffic, under real load

---

# 🏗️ What is Chaos Engineering?

👉 **Chaos Engineering = deliberately causing failures, in a controlled environment, to discover weaknesses before they happen by accident**

```
Hypothesis: "If the payment service goes down, the circuit breaker 
(post 40) should open after 5 consecutive failures, and the system 
should degrade gracefully, without bringing down the entire checkout"

Experiment: deliberately take down the payment service, in staging,
and observe whether the hypothesis holds
```

---

# 🎯 Types of failure to inject

## 🔹 Artificial latency

```csharp
public class ChaosLatencyMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        if (_chaosFlag.IsActive("artificial-latency")) // remember post 51?
        {
            await Task.Delay(TimeSpan.FromSeconds(5));
        }

        await next(context);
    }
}
```

👉 Simulates a slow dependency — tests whether your timeouts (Polly, post 40) actually trigger when they should

## 🔹 Dependency failure

```csharp
if (_chaosFlag.IsActive("payment-service-failure"))
{
    throw new HttpRequestException("Simulated failure");
}
```

👉 Simulates the payment service being down — tests whether the circuit breaker actually opens and whether the fallback (also from post 40) actually works

## 🔹 Infrastructure failure

```bash
kubectl delete pod orders-api-7d9f8-x2j4k
```

👉 Remember Kubernetes (post 86)? Deliberately killing a Pod tests whether the `Deployment` actually brings up another one automatically, with no noticeable loss of service

---

# 🔍 Where observability comes in

```
During the chaos experiment:
  Checkout error rate: 2% (acceptable, graceful degradation)
  Circuit breaker opened after 5 failures: ✅ confirmed
  Recovery time after the Pod came back: 8 seconds
```

👉 Remember OpenTelemetry (post 55)? Without real-time metrics, you can't tell the difference between "the system behaved as expected" and "the system broke silently" during the experiment

---

# ⚖️ Chaos Engineering in production vs staging

## 🔹 Staging
- Safe environment to start with, no impact on real users  
- May not exactly replicate production load conditions  

## 🔹 Production (with caution)
- Netflix popularized this with Chaos Monkey — randomly killing instances in production  
- Requires high operational maturity, starting with a small, controlled "blast radius"  

---

# ⚠️ Common Mistakes

- Running chaos experiments without a clear hypothesis, not knowing exactly what's being validated  
- Running in production without fast rollback capability (remember Blue-Green from the previous post?) in case the experiment gets out of hand  
- Not having enough observability to measure the experiment's real impact  
- Only testing obvious failures, ignoring combined scenarios (latency + partial failure + traffic spike all at once)  

---

# 📌 Conclusion

- Chaos Engineering deliberately causes failures to validate resilience before a real incident  
- It tests specific hypotheses: "does the circuit breaker actually open?", "does the Pod actually recover?"  
- Observability (post 55) is a prerequisite for measuring whether the system behaved as expected  
- Start in staging, with a small blast radius, before considering production  

👉 With Chaos Engineering, you discover your system's weaknesses in a controlled experiment, not in a 3am incident

---

# 🔥 Next Step

Now that you test resilience on purpose, the next level is:

👉 **C# Fundamentals: Load Testing with k6**

Here you'll learn to simulate real load before it actually happens, discovering your system's limits.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
