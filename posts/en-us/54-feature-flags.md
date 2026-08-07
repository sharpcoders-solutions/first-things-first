# 🧠 C# Fundamentals: Feature Flags and Dynamic Configuration

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- The entire foundation of the C# language, types, and how to build a complete .NET API  
- Modern language features: enums, tuples, anonymous types, and the evolution of C#  

Time for the second half of this track: building the capabilities that separate a system that "works" from one that's **operated with maturity** in production.

👉 **Let's start with Feature Flags**

---

# 💡 What is a Feature Flag?

👉 **Feature Flag = a conditional that turns a feature on or off without needing a new deployment**

```csharp
if (featureManager.IsEnabled("NewCheckout"))
{
    return NewCheckoutFlow();
}

return CurrentCheckoutFlow();
```

Without flags, enabling a new feature means deploying. With flags, the code is already in production, **turned off**, and you flip it on whenever you want — without touching infrastructure again.

---

# 🏗️ Using `Microsoft.FeatureManagement`

```bash
dotnet add package Microsoft.FeatureManagement.AspNetCore
```

```json
// appsettings.json
{
  "FeatureManagement": {
    "NewCheckout": true,
    "AdvancedReports": false
  }
}
```

```csharp
// Program.cs
builder.Services.AddFeatureManagement();
```

```csharp
public class CheckoutController : ControllerBase
{
    private readonly IFeatureManager _featureManager;

    public CheckoutController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }

    [HttpPost]
    public async Task<IActionResult> Complete()
    {
        if (await _featureManager.IsEnabledAsync("NewCheckout"))
            return Ok(NewFlow());

        return Ok(CurrentFlow());
    }
}
```

👉 `IFeatureManager` is injected via DI — the same mechanism you've mastered since the ASP.NET Core post, now controlling behavior instead of dependencies

---

# 🎯 Gradual rollout: releasing little by little

```csharp
public class PercentageFilter : IFeatureFilter
{
    public Task<bool> EvaluateAsync(FeatureFilterEvaluationContext context)
    {
        var percentage = context.Parameters.GetValue<int>("Percentage");
        return Task.FromResult(new Random().Next(100) < percentage);
    }
}
```

```json
{
  "FeatureManagement": {
    "NewCheckout": {
      "EnabledFor": [
        { "Name": "PercentageFilter", "Parameters": { "Percentage": 10 } }
      ]
    }
  }
}
```

👉 Instead of "on for everyone" or "off for everyone," you release to 10% of users, observe (remember the health checks and logging posts?), and gradually raise the percentage if everything stays healthy

---

# 🔀 Feature Flags vs regular configuration

## 🔹 Regular configuration (`appsettings.json`)
- Values that rarely change (connection strings, service URLs)  
- Set at deploy time, doesn't change in real time  

## 🔹 Feature Flag
- Controls **behavior**, not just values  
- Can change in real time, without restarting the application (with a dynamic configuration provider, like Azure App Configuration)  
- Has a lifecycle: it's born, tested, released, and **should be removed** once the feature becomes permanent  

---

# 🧹 The technical debt of forgotten flags

👉 **Every flag is temporary technical debt, by design**

```csharp
// ❌ Flag forgotten 8 months ago, no one remembers why it's there
if (await _featureManager.IsEnabledAsync("OldCheckoutTestFrom2023"))
```

A flag meant to last a two-week test that's still in the code a year later is a sign of a broken process — every flag should have an owner and an expected removal date

---

# ⚠️ Common Mistakes

- Leaving old flags in the code after the decision has already been made, accumulating conditional complexity  
- Using flags to hide broken code instead of regular configuration for stable values  
- Only testing with the flag on, forgetting to validate behavior with it off  
- Not documenting each flag's purpose and expected removal date  

---

# 📌 Conclusion

- Feature Flags decouple deployment from feature release  
- `IFeatureManager` controls flags via DI, the same way as any other dependency  
- Percentage rollout releases features gradually, with real observation  
- Every flag is temporary by nature — removing it is part of the process, not an extra step  

👉 With feature flags, shipping something new stops being a risky event and becomes a reversible, controlled decision

---

# 🔥 Next Step

Now that you can control what runs in production, the next level is:

👉 **C# Fundamentals: Background Jobs with Hangfire**

Here you'll learn to run tasks outside the lifecycle of an HTTP request.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
