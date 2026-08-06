# 🧠 C# Fundamentals: Load Testing with k6

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Chaos Engineering for testing failure resilience  
- Kubernetes autoscaling (post 86), reacting to load automatically  

Chaos Engineering tests "what happens when something breaks." Load Testing tests a different question: "how many simultaneous users can my system handle before degrading?" Finding that out in a controlled test is much better than finding out on Black Friday.

👉 **Let's learn Load Testing with k6**

---

# 💡 What k6 tests that regular tests don't

```csharp
// Integration test (post 59): validates behavior, one request at a time
[Fact]
public async Task PostOrder_ShouldReturn201()
{
    var response = await _client.PostAsJsonAsync("/orders", newOrder);
    Assert.Equal(HttpStatusCode.Created, response.StatusCode);
}
```

👉 This test guarantees the endpoint **works**. It says nothing about what happens when 1000 users call that same endpoint at once

---

# 🏗️ Writing a load test with k6

```javascript
// load.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },   // ramps up gradually to 50 users
    { duration: '1m', target: 50 },    // holds 50 users for 1 minute
    { duration: '30s', target: 0 },    // ramps down gradually to 0
  ],
};

export default function () {
  const response = http.get('https://myapi.com/orders/123');

  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);
}
```

```bash
k6 run load.js
```

👉 k6 simulates real users making concurrent requests, with ramp-up and ramp-down stages — similar to an application's real traffic pattern, not an artificial instant burst

---

# 🎯 What to look at in the results

```
     http_req_duration..............: avg=145ms  p(95)=380ms  p(99)=720ms
     http_req_failed.................: 0.42%
     http_reqs.......................: 15234  (253.9/s)
```

👉 Remember the OpenTelemetry post (55)? The same p95/p99 latency concepts show up here — the average (`avg`) hides outliers, so `p(95)` and `p(99)` show the real experience of the most affected users

---

# 🔍 Finding the breaking point

```javascript
export const options = {
  stages: [
    { duration: '1m', target: 100 },
    { duration: '1m', target: 500 },
    { duration: '1m', target: 1000 },
    { duration: '1m', target: 2000 }, // keep increasing until something breaks
  ],
};
```

👉 A stress test increases load until the system degrades — the goal isn't to "pass," it's to discover exactly where the limit is, so you know ahead of time when autoscaling (post 86) needs to kick in

---

# 🔌 Connecting with autoscaling and performance

```yaml
# HorizontalPodAutoscaler (post 86)
minReplicas: 2
maxReplicas: 10
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        averageUtilization: 70
```

👉 The load test validates whether this autoscaler actually reacts in time — if k6 shows that 500 concurrent users tank latency before the HPA creates new Pods, that's a configuration that needs tuning, discovered in a test, not in production

---

# ⚠️ Common Mistakes

- Running load tests directly against production without warning the team, generating false incident alerts  
- Only testing the "happy path," ignoring heavier endpoints (reports, complex searches)  
- Not isolating the test environment — test load competing with real traffic distorts results  
- Ignoring tests after running them once — capacity changes as code evolves, load testing should be continuous, not a one-off  

---

# 📌 Conclusion

- Load Testing discovers the system's real limits, under simulated load  
- k6 simulates concurrent users with ramp-up and ramp-down stages, similar to real traffic  
- p95/p99 metrics reveal the experience of the most affected users, not just the average  
- The results directly validate whether autoscaling (post 86) reacts in time  

👉 With Load Testing, you discover your system's limits in a controlled environment, well before they show up at the worst possible moment

---

# 🔥 Next Step

Now that you know your system's limits under load, the next level is:

👉 **C# Fundamentals: Full Observability (Metrics, Traces, Logs)**

Here you'll consolidate everything you've learned about observability into one unified, complete view.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
