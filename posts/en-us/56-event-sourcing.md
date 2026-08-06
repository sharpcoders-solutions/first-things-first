# 🧠 C# Fundamentals: Event Sourcing — Introduction

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Observability for tracing requests across services  
- DDD, with entities, aggregates, and domain events  

In the traditional model, the database only stores the **current state**: an order is "Shipped," full stop. You've lost the entire history of how it got there. Event Sourcing solves this by storing every change as an immutable event.

👉 **Let's learn Event Sourcing**

---

# 💡 What is Event Sourcing?

👉 **Event Sourcing = storing the complete sequence of events that led to the current state, instead of just the current state**

## 🔹 Traditional model (CRUD)

```sql
UPDATE Orders SET Status = 'Shipped' WHERE Id = 123
```

👉 The `UPDATE` overwrites the previous value — "Confirmed" and "Paid" are gone forever

## 🔹 Event Sourcing

```csharp
public record OrderCreated(int OrderId, DateTime When);
public record OrderConfirmed(int OrderId, DateTime When);
public record OrderPaid(int OrderId, decimal Amount, DateTime When);
public record OrderShipped(int OrderId, string TrackingCode, DateTime When);
```

👉 Every state change becomes an event, recorded and never deleted. The current state is **derived** from the sequence of events — remember the records from post 27? They're perfect for representing immutable events

---

# 🏗️ Rebuilding state from events

```csharp
public class Order
{
    public int Id { get; private set; }
    public string Status { get; private set; } = "New";
    public decimal AmountPaid { get; private set; }

    public static Order RebuildFromEvents(IEnumerable<object> events)
    {
        var order = new Order();

        foreach (var @event in events)
        {
            order.Apply(@event);
        }

        return order;
    }

    private void Apply(object @event)
    {
        switch (@event)
        {
            case OrderCreated e:
                Id = e.OrderId;
                Status = "Created";
                break;
            case OrderConfirmed:
                Status = "Confirmed";
                break;
            case OrderPaid e:
                Status = "Paid";
                AmountPaid = e.Amount;
                break;
            case OrderShipped:
                Status = "Shipped";
                break;
        }
    }
}
```

👉 This looks like the pattern matching from post 27 — each event knows how to transform the state, and the final state is just the result of "replaying" all events in order

---

# 📦 Storing events: the Event Store

```csharp
public class EventStore
{
    private readonly List<StoredEvent> _events = new();

    public void Save(int aggregateId, object @event)
    {
        _events.Add(new StoredEvent(aggregateId, @event, DateTime.UtcNow));
    }

    public IEnumerable<object> GetEvents(int aggregateId) =>
        _events.Where(e => e.AggregateId == aggregateId).Select(e => e.Event);
}
```

👉 In practice, the event store is usually a specialized database (EventStoreDB) or an append-only table in a relational database — you never `UPDATE` or `DELETE`, only `INSERT`

---

# 🎯 The real advantages

- **Complete, free auditing** — you know exactly when and why every change happened  
- **State reconstruction at any point in time** — "what did the order look like at 2pm yesterday?"  
- **Production debugging** — reproduce the exact sequence of events that led to a bug  

---

# ⚠️ Common Mistakes

- Applying Event Sourcing across the entire system, when only aggregates with valuable history (orders, financial transactions) truly benefit  
- Not versioning events — when the shape of `OrderCreated` changes, old events still need to be readable correctly  
- Rebuilding state from scratch on every read without using snapshots, making aggregates with many events slow to load  
- Underestimating the complexity — Event Sourcing trades CRUD simplicity for audit power; use it where history actually matters  

---

# 📌 Conclusion

- Event Sourcing stores the complete sequence of events, not just the final state  
- Current state is derived by applying events in order  
- The Event Store is append-only — events are never modified or deleted  
- The real payoff is complete auditing and the ability to reconstruct state at any point in time  

👉 With Event Sourcing, your system stops asking only "what's the current state?" and starts being able to answer "how did we get here?"

---

# 🔥 Next Step

Now that you can store the complete history of changes, the next level is:

👉 **C# Fundamentals: Saga Pattern for Distributed Transactions**

Here you'll learn to coordinate transactions that span multiple services, without a traditional database transaction.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
