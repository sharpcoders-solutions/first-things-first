# 🧠 C# Fundamentals: Blue-Green and Canary Deployment

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Terraform for versioning infrastructure  
- CI/CD with GitHub Actions (post 36), automating deploys  

Up to now, "deploying" meant replacing the old version with the new one, all at once. That works, but if the new version has a bug, every user gets hit immediately. Blue-Green and Canary reduce that risk.

👉 **Let's learn Blue-Green and Canary Deployment**

---

# 💡 The problem with "all at once" deployment

```
Version 1.0 running → deploy → Version 1.1 running (100% of traffic, instantly)
```

👉 If version 1.1 has a critical bug, 100% of users feel the impact at the same time — and even a fast rollback still leaves a window of full impact

---

# 🏗️ Blue-Green Deployment

```
Blue environment (current production, v1.0) ← 100% of traffic
Green environment (new version, v1.1) ← 0% of traffic, but already running and testable
```

```yaml
# Kubernetes Service (post 86) points to "blue"
apiVersion: v1
kind: Service
metadata:
  name: orders-api
spec:
  selector:
    app: orders-api
    version: blue  # switches to "green" at cutover time
```

👉 You bring up the new version (Green) in parallel, fully testable, without receiving real traffic. When confident, you switch the `Service` to point to Green — the cutover is instant, and rollback is just pointing back to Blue

---

# 🎯 Canary Deployment

```yaml
# 95% of traffic to the stable version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-api-stable
spec:
  replicas: 19  # 95% of 20 total replicas

---
# 5% of traffic to the new version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-api-canary
spec:
  replicas: 1  # 5% of 20 total replicas
```

👉 Instead of a full switchover, you release the new version to a small slice of real traffic — if the metrics (remember post 55, OpenTelemetry?) stay healthy, you gradually increase the percentage up to 100%

---

# 🔍 Observability is what makes this safe

```
Canary at 5% of traffic:
  Error rate: 0.1% (normal)
  p99 latency: 120ms (normal)
  → Increase to 25%

Canary at 25% of traffic:
  Error rate: 8% (abnormal!)
  → Abort and revert to 0%
```

👉 Remember the OpenTelemetry post (55)? Without real-time metrics comparing the new version against the stable one, a Canary Deployment is just a slower Blue-Green — the decision to advance or roll back depends entirely on being able to observe real behavior

---

# ⚖️ Blue-Green vs Canary: when to use each

## 🔹 Blue-Green
- Fast cutover, all or nothing  
- Instant rollback (just switch routing back)  
- Requires temporarily duplicated capacity (two full environments running)  

## 🔹 Canary
- Gradual exposure, risk limited to a fraction of users  
- Lets you compare real metrics between versions before committing 100%  
- Slower to complete the full cutover  

---

# ⚠️ Common Mistakes

- Doing Blue-Green without migrating database data/schema in a way that's compatible with both versions simultaneously  
- Running Canary without automated metrics, deciding to advance "by feel" instead of objective data  
- Forgetting stateful sessions (sticky sessions) during the transition, breaking the experience for users mid-operation  
- Not automating the rollback — if it depends on someone waking up at 3am and reacting manually, the strategy's value drops a lot  

---

# 📌 Conclusion

- Blue-Green switches traffic all at once, with instant rollback  
- Canary releases the new version gradually, limiting the blast radius of bugs  
- Both strategies depend on real observability to be safe  
- The choice depends on how much risk you accept versus how much extra capacity you have available  

👉 With Blue-Green and Canary, putting new code into production stops being a full-risk event and becomes a controlled, reversible process

---

# 🔥 Next Step

Now that you reduce deployment risk, the next level is:

👉 **C# Fundamentals: Chaos Engineering**

Here you'll learn to test your system's resilience by deliberately causing failures, before they happen by accident.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
