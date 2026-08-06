# 🧠 C# Fundamentals: Trunk-Based Development

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Git Workflow with branches and Pull Requests  
- Feature Flags for releasing features without a new deployment  

Post 11 taught you the classic flow: feature branch, PR, review, merge. That works well, but on large teams it can lead to branches living for weeks, full of merge conflicts. Trunk-Based Development proposes a different path.

👉 **Let's learn Trunk-Based Development**

---

# 💡 The problem with long-lived branches

```
main
  └─ feature/new-checkout (alive for 3 weeks)
       └─ 40 commits behind main
       └─ massive conflicts when trying to merge
```

👉 The longer a branch stays separated from `main`, the bigger the gap between them — and the higher the risk of a painful merge, full of hard-to-resolve conflicts

---

# 🏗️ What is Trunk-Based Development?

👉 **Trunk-Based Development = everyone commits directly (or nearly directly) to the main branch, with a cadence of hours, not weeks**

```
main (trunk)
  ├─ Maria's commit (9am)
  ├─ João's commit (9:15am)
  ├─ Valentina's commit (9:40am)
  └─ Maria's commit (10:05am)
```

👉 Instead of a feature branch living for weeks, developers make small, frequent commits directly to the trunk (or on very short-lived branches, lasting a few hours)

---

# 🎯 How does this work without breaking `main`?

## 🔹 Feature Flags hide incomplete work

```csharp
if (await _featureManager.IsEnabledAsync("NewCheckout"))
{
    return NewCheckoutFlow(); // work in progress, but already on trunk
}

return CurrentCheckout();
```

👉 Remember post 51? Instead of a separate branch hiding incomplete work, the feature flag hides it — the code is already integrated, just switched off

## 🔹 CI/CD runs on every commit

```yaml
on:
  push:
    branches: [main]
```

👉 Remember the GitHub Actions post (post 36)? Every commit to trunk automatically triggers a build and tests — problems get caught in minutes, not weeks later

## 🔹 Short-lived branches, when they exist

```bash
git checkout -b fix/critical-bug
# ... fix, commit, open PR
# ... merged within hours, not days
git branch -d fix/critical-bug
```

👉 When a branch is needed, it lives for **hours**, not weeks — the goal is always to get back to trunk as fast as possible

---

# ⚖️ GitFlow vs Trunk-Based

## 🔹 GitFlow (post 11)
- Feature, develop, and release branches — more structure  
- Better for teams shipping spaced-out versions (libraries, products with defined release cycles)  

## 🔹 Trunk-Based
- True continuous integration — small, frequent conflicts, not large and rare ones  
- Requires mature Feature Flags and CI/CD to work well  
- Common on teams doing continuous deployment, multiple times a day  

---

# ⚠️ Common Mistakes

- Adopting Trunk-Based without solid CI/CD, constantly breaking `main` for everyone  
- Not using Feature Flags, forcing incomplete code to hide in long branches anyway  
- Making giant commits "because everything's on trunk now," losing the granularity the model calls for  
- Applying Trunk-Based on teams without automated testing discipline — without a safety net, the trunk breaks frequently  

---

# 📌 Conclusion

- Long-lived branches accumulate distance and create painful merges  
- Trunk-Based Development integrates code on a cadence of hours, not weeks  
- Feature Flags hide incomplete work without needing separate branches  
- Robust CI/CD is a prerequisite, not optional, for this model to work  

👉 With Trunk-Based Development, integration stops being a stressful event at the end of a cycle and becomes a natural part of everyday work

---

# 🔥 Next Step

Now that you know advanced branching strategies, the next level is:

👉 **C# Fundamentals: Unsafe Code and Pointers**

Here you'll learn to step outside C#'s managed safety when extreme performance demands direct memory control.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
