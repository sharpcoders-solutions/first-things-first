# 🧠 C# Fundamentals: Microservices — Introduction and When to Use Them

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- DDD and the idea of Bounded Context  
- Messaging to decouple systems in time  

You've already seen that each Bounded Context can have its own model. The natural next question is: should each one also be a **separate application**?

👉 **Let's understand what microservices are, and — more importantly — when they're actually worth it**

---

# 💡 Monolith vs Microservices

## 🔹 Monolith

👉 The entire application (API, business rules, data access) runs as **a single process**, a single deployment

```
[ Monolith: Orders + Inventory + Payments + Notifications ]
```

## 🔹 Microservices

👉 The application is split into **independent services**, each with its own database, deployment, and lifecycle

```
[ Orders Service ]   [ Inventory Service ]   [ Payments Service ]   [ Notifications Service ]
```

👉 Each service talks to the others via **HTTP/API** or **messaging** (exactly what you learned in the RabbitMQ post) — never directly accessing another service's database

---

# 🗺️ Where DDD fits in: Bounded Context as a natural boundary

Remember the "Customer" example meaning different things in Sales and Support? Every well-defined Bounded Context is a natural candidate to become a microservice:

```
Bounded Context "Sales"   → Sales Service
Bounded Context "Inventory" → Inventory Service
Bounded Context "Support" → Support Service
```

👉 Poorly split microservices usually come from a domain modeling mistake, not an infrastructure one — splitting without understanding the domain's boundaries tends to create services that call each other constantly, the worst of both worlds

---

# ✅ When microservices make sense

- **Large teams**, where different squads need to ship independently, without waiting on each other  
- **Uneven scalability**: a payments service might need 10x more instances than the notifications service  
- **Different technologies per domain**: a recommendation service might benefit from Python/ML, while the rest of the system stays in C#  
- **Failure isolation**: a problem in the notifications service shouldn't take down checkout  

---

# ❌ When microservices are an expensive mistake

👉 **The most common trap: adopting microservices before you actually need them**

- Small teams pay the cost of **operating** multiple services (deployment, monitoring, network communication) without the benefit of real scale  
- Debugging a problem that spans five services is orders of magnitude harder than debugging a well-organized monolith  
- Transactions spanning multiple services lose the simple consistency guarantee a single database provides  

**Practical rule:** start with a **well-modularized monolith** (using Bounded Context boundaries as internal code divisions, not deployment ones). Extract a microservice only when you feel a real, specific pain that the split solves — not because "that's how big companies do it."

---

# 🔗 How services communicate

## 🔹 Synchronous (HTTP/API)

```csharp
public class OrdersService
{
    private readonly HttpClient _inventoryClient;

    public async Task<bool> CheckAvailability(int productId, int quantity)
    {
        var response = await _inventoryClient.GetAsync($"/inventory/{productId}");
        var inventory = await response.Content.ReadFromJsonAsync<InventoryDto>();
        return inventory.Quantity >= quantity;
    }
}
```

👉 Simple, but creates temporal coupling: if the inventory service is down, creating the order fails too — this is exactly where the Polly post becomes essential, not optional

## 🔹 Asynchronous (messaging)

```csharp
_publisher.Publish(new OrderCreatedEvent(order.Id));
// the inventory service consumes this event independently, on its own time
```

👉 Reduces temporal coupling — the orders service doesn't wait for inventory to respond, exactly the pattern from the RabbitMQ post

---

# 🌐 API Gateway: a single entry point

```
Client → [ API Gateway ] → Orders Service
                         → Inventory Service
                         → Payments Service
```

👉 Instead of the front-end knowing the address of every individual microservice, an **API Gateway** centralizes authentication, routing, and response aggregation — the client talks to a single point, even though there are ten services behind it

---

# ⚠️ Common Mistakes

- Migrating to microservices without having the scale or organizational problem that would justify them  
- Creating "distributed monoliths": separate services that still share the same database, losing the isolation that's the whole point of the pattern  
- Not applying resilience (Polly) on calls between services, letting a cascading failure take down the entire system  
- Splitting services by technical layer (e.g., "database service") instead of by business domain  

---

# 📌 Conclusion

- Monoliths concentrate everything in one process; microservices split into independent services  
- Bounded Context (from DDD) is the natural boundary for splitting microservices sensibly  
- Microservices solve team and infrastructure scaling problems — they're not a goal in themselves  
- Synchronous communication creates temporal coupling; asynchronous messaging reduces it  
- API Gateway centralizes the entry point for multiple services  

👉 The most important decision about microservices isn't "how to implement them," it's "do I actually need this right now"

---

# 🔥 Next Step

Now that you understand when to split a system into services, the next level is:

👉 **C# Fundamentals: gRPC — Efficient Communication Between Services**

Here you'll learn a faster, strongly-typed alternative to REST for synchronous communication between microservices.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
