# 🧠 C# Fundamentals: Saga Pattern for Distributed Transactions

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Event Sourcing for storing a complete history of changes  
- Microservices — each with its own database  

If each microservice has its own database, what happens when an order needs to: reserve stock, charge payment, and schedule delivery — three different databases? A traditional `BEGIN/COMMIT` transaction doesn't span services. That's where the Saga pattern comes in.

👉 **Let's learn the Saga pattern**

---

# 💡 The problem: transactions that cross services

```csharp
// ❌ This doesn't work across different microservices
using var transaction = await _context.Database.BeginTransactionAsync();
_stockService.Reserve(orderId);      // inventory service's database
_paymentService.Charge(orderId);     // payment service's database
_deliveryService.Schedule(orderId);  // delivery service's database
await transaction.CommitAsync();     // there's no distributed transaction here
```

👉 Each service has its own database (remember the microservices post?) — there's no single `COMMIT` that ties the three together

---

# 🏗️ Saga = a sequence of compensatable local transactions

👉 **Saga = a chain of steps, where each step has an action and a compensation to undo it if something fails later**

## 🔹 Choreographed Saga (events)

```csharp
public class OrdersService
{
    public async Task CreateOrder(int orderId)
    {
        await _orders.Create(orderId);
        await _publisher.Publish(new OrderCreated(orderId));
    }
}

public class StockService
{
    public async Task OnOrderCreated(OrderCreated @event)
    {
        var reserved = await _stock.Reserve(@event.OrderId);

        if (reserved)
            await _publisher.Publish(new StockReserved(@event.OrderId));
        else
            await _publisher.Publish(new StockUnavailable(@event.OrderId));
    }
}
```

👉 Remember RabbitMQ (post 41)? Each service reacts to the previous one's event and publishes its own — there's no central "conductor," services coordinate through messaging

## 🔹 Orchestrated Saga (a central coordinator)

```csharp
public class OrderOrchestrator
{
    public async Task Execute(int orderId)
    {
        try
        {
            await _stock.Reserve(orderId);
            await _payment.Charge(orderId);
            await _delivery.Schedule(orderId);
        }
        catch (Exception)
        {
            await Compensate(orderId);
            throw;
        }
    }

    private async Task Compensate(int orderId)
    {
        await _delivery.CancelSchedule(orderId);
        await _payment.Refund(orderId);
        await _stock.ReleaseReservation(orderId);
    }
}
```

👉 A central orchestrator knows the entire sequence and decides what to do on each failure — easier to understand and debug, but creates a central point of coordination

---

# ⚖️ Choreographed vs orchestrated

## 🔹 Choreographed
- No central point — decoupled services, each only knows its own events  
- Hard to visualize the full flow just by reading the code  

## 🔹 Orchestrated
- Full flow visible in one place  
- The orchestrator becomes a dependency every service needs to know about  

👉 For simple sagas (2-3 steps), choreographed works well. For complex sagas with many branches, orchestrated is usually easier to maintain

---

# 🔄 Compensation: the heart of the pattern

👉 **Every action needs an equivalent, idempotent compensation**

| Action | Compensation |
|---|---|
| Reserve stock | Release reservation |
| Charge payment | Refund |
| Schedule delivery | Cancel schedule |

👉 Unlike a database `ROLLBACK`, compensation is explicit business code — refunding a payment isn't "undoing," it's a new operation that neutralizes the previous one

---

# ⚠️ Common Mistakes

- Treating a Saga like a traditional ACID transaction — during execution, the system sits in visible intermediate states (eventual, not immediate, consistency)  
- Forgetting to make compensations idempotent, causing duplicate refunds on retry  
- Choosing orchestration for everything, creating a "central" service that knows too much about the others  
- Not monitoring sagas stuck midway — without observability (remember post 55?), an incomplete saga can go unnoticed  

---

# 📌 Conclusion

- Saga coordinates transactions that cross multiple services through compensatable local steps  
- Choreographed uses events; orchestrated uses a central coordinator  
- Every action needs an idempotent compensation to undo its effect on failure  
- Sagas bring eventual consistency, not the immediate atomicity of a traditional database transaction  

👉 With the Saga pattern, distributed systems gain a reliable way to coordinate operations that no single database can tie together

---

# 🔥 Next Step

Now that you can coordinate distributed transactions, the next level is:

👉 **C# Fundamentals: Outbox Pattern**

Here you'll learn to guarantee that a database change and an event publication happen reliably, together.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
