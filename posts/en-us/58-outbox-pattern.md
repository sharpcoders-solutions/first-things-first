# 🧠 C# Fundamentals: Outbox Pattern

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Saga for coordinating distributed transactions  
- RabbitMQ for publishing events between services  

There's a subtle problem hiding in code that looks correct: saving to the database and publishing a message are two separate operations. What happens if one succeeds and the other fails?

👉 **Let's learn the Outbox pattern**

---

# 💡 The dual-write problem

```csharp
// ❌ Two operations that can fail independently
public async Task CreateOrder(Order order)
{
    await _context.Orders.AddAsync(order);
    await _context.SaveChangesAsync();                  // operation 1: database

    await _bus.Publish(new OrderCreated(order.Id));      // operation 2: messaging
}
```

👉 If `SaveChangesAsync` succeeds but the process crashes before `Publish`, the order exists in the database, but **no other service ever finds out**. Stock never gets reserved, payment never gets charged

---

# 🏗️ The solution: the Outbox table

👉 **Outbox = write the event in the same database transaction, and actually publish it afterward, asynchronously**

```csharp
public class OutboxMessage
{
    public Guid Id { get; set; }
    public string Type { get; set; } = default!;
    public string ContentJson { get; set; } = default!;
    public DateTime CreatedAt { get; set; }
    public DateTime? ProcessedAt { get; set; }
}
```

```csharp
public async Task CreateOrder(Order order)
{
    await _context.Orders.AddAsync(order);

    _context.OutboxMessages.Add(new OutboxMessage
    {
        Id = Guid.NewGuid(),
        Type = nameof(OrderCreated),
        ContentJson = JsonSerializer.Serialize(new OrderCreated(order.Id)),
        CreatedAt = DateTime.UtcNow
    });

    await _context.SaveChangesAsync(); // ✅ a single, atomic transaction
}
```

👉 Since the order and the outbox message are saved in the **same EF Core transaction** (remember post 32?), either both are written, or neither is — never a broken intermediate state

---

# 📤 Publishing the messages: the dispatch process

```csharp
public class OutboxDispatcher : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken cancellationToken)
    {
        while (!cancellationToken.IsCancellationRequested)
        {
            var pending = await _context.OutboxMessages
                .Where(m => m.ProcessedAt == null)
                .Take(50)
                .ToListAsync(cancellationToken);

            foreach (var message in pending)
            {
                await _bus.Publish(message.Type, message.ContentJson);
                message.ProcessedAt = DateTime.UtcNow;
            }

            await _context.SaveChangesAsync(cancellationToken);
            await Task.Delay(TimeSpan.FromSeconds(5), cancellationToken);
        }
    }
}
```

👉 This is a `BackgroundService`, the same mechanism used in the Hangfire post — a separate process reads the outbox table and actually publishes, retrying on failure, never losing the message

---

# 🔁 At-least-once guarantee, not exactly-once

👉 **The Outbox pattern guarantees the message will be published at least once — not exactly once**

```csharp
public async Task OnOrderCreated(OrderCreated @event)
{
    if (await _processed.HasProcessed(@event.OrderId))
        return; // idempotency: ignore duplicate

    await _stock.Reserve(@event.OrderId);
    await _processed.MarkAsProcessed(@event.OrderId);
}
```

👉 If the dispatcher fails after publishing but before marking as processed, the message can be resent — that's why the consumer needs to be idempotent, just like we discussed in the RabbitMQ post

---

# ⚠️ Common Mistakes

- Publishing the message before calling `SaveChangesAsync`, reintroducing the dual-write problem  
- Not cleaning up old processed messages, letting the outbox table grow indefinitely  
- Forgetting to make consumers idempotent, assuming outbox guarantees single delivery  
- Running multiple dispatcher instances without concurrency control, processing the same message twice at once  

---

# 📌 Conclusion

- The dual-write problem happens when saving to the database and publishing an event are separate operations  
- The Outbox pattern writes the event in the same database transaction, guaranteeing atomicity  
- A separate process asynchronously dispatches the pending messages  
- The guarantee is at-least-once — consumers need to be idempotent  

👉 With the Outbox pattern, your system guarantees no event is ever lost, even when failures happen at the worst possible moment

---

# 🔥 Next Step

Now that your messages never get lost, the next level is:

👉 **C# Fundamentals: Integration Testing with WebApplicationFactory**

Here you'll learn to test your entire API, from HTTP down to the database, with real automated tests.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
