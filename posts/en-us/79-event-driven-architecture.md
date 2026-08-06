# 🧠 C# Fundamentals: Event-Driven Architecture

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned individual pieces: Event Sourcing (56), Saga (57), Outbox (58), and Kafka (78). Time to see how they fit together in a real system, event-driven from end to end.

👉 **Let's connect it all in an Event-Driven Architecture**

---

# 💡 What changes in an event-driven architecture?

👉 **Instead of services calling each other directly (request/response), services react to events that have already happened**

## 🔹 Traditional architecture (coupled)

```csharp
public async Task CreateOrder(Order order)
{
    await _stockService.Reserve(order);      // direct coupling
    await _paymentService.Charge(order);      // if one goes down, everything blocks
    await _notificationService.Send(order);
}
```

## 🔹 Event-driven (decoupled)

```csharp
public async Task CreateOrder(Order order)
{
    await _repository.Save(order);
    await _eventPublisher.Publish(new OrderCreated(order.Id)); // and done
}
```

👉 The orders service doesn't know (and doesn't need to know) who reacts to the event — stock, payment, and notifications react independently, each at its own pace

---

# 🏗️ Putting the pieces together: the full flow

```
1. Customer creates an order
   └─ OrdersService saves to the database + writes the event to the Outbox table (post 58)
   
2. OutboxDispatcher publishes the event to Kafka (post 78)
   └─ Topic "orders-created"

3. Multiple consumers react independently:
   ├─ StockService reserves the product
   ├─ PaymentService starts the charge
   └─ NotificationService sends a confirmation email

4. If something fails, the Saga (post 57) coordinates compensation
   └─ StockService releases the reservation
   └─ PaymentService refunds, if already charged

5. Every order state change becomes a persisted event (post 56)
   └─ Full history: OrderCreated → StockReserved → PaymentConfirmed → OrderShipped
```

👉 This is exactly the sum of the previous four posts — Outbox guarantees the event is never lost, Kafka distributes at scale, Saga coordinates failures, and Event Sourcing keeps the complete history

---

# 🎯 Real advantages of this combination

- **Temporal decoupling** — services don't need to all be available at the same time  
- **Independent scalability** — the notification service can scale separately from the payment service  
- **Resilience to partial failures** — if the notification service goes down, orders keep being processed; notifications get sent once it's back  
- **Natural auditing** — the complete event history already exists, with no extra effort  

---

# ⚠️ The trade-off: eventual consistency

```csharp
// A freshly created order might not have stock reserved yet
var order = await _repository.Find(orderId);
Console.WriteLine(order.Status); // "Created" — the reservation event is still being processed
```

👉 Remember what we discussed in the Saga post? Event-driven architectures trade immediate consistency for eventual consistency — the UI needs to be designed to handle visible intermediate states, not assume everything has already happened instantly

---

# ⚠️ Common Mistakes

- Adopting event-driven architecture for the entire system, when parts of it are naturally synchronous and simple  
- Not investing in observability (post 55) — without distributed tracing, debugging an event flow that spans 5 services is extremely difficult  
- Forgetting that events are a public contract between services — changing a published event's shape breaks every consumer  
- Ignoring idempotency in each consumer, assuming every event arrives exactly once  

---

# 📌 Conclusion

- Event-driven architecture replaces direct calls with reactions to published events  
- Outbox, Kafka, Saga, and Event Sourcing combine to form a complete, resilient flow  
- The gain is decoupling, independent scalability, and natural auditing  
- The cost is eventual consistency — a conscious trade-off, not an unwanted side effect  

👉 With event-driven architecture, you design systems that keep working even when individual parts fail temporarily

---

# 🔥 Next Step

Now that you can connect event-driven architectures, the next level is:

👉 **C# Fundamentals: GraphQL with HotChocolate**

Here you'll learn an alternative to REST for more flexible API queries, letting the client decide exactly what data it needs.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
