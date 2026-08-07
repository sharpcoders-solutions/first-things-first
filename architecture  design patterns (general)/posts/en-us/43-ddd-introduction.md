# 🧠 C# Fundamentals: Domain-Driven Design (DDD) — Introduction

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- CQRS separating commands from queries  
- Clean Architecture organizing code into layers  

You already know **where** to put each type of code. DDD is about a different, deeper question: how do you make sure your classes truly represent the business rules, not just loose fields and methods?

👉 **Let's get to know the core concepts of Domain-Driven Design**

---

# 💡 What is DDD?

👉 **DDD = an approach to modeling software that mirrors, as closely as possible, the language and rules of the real business**

The core idea: talk to whoever understands the business (not just the code), and use the **same terms** they use in your classes, methods, and variable names. This is called the **Ubiquitous Language** — everyone, technical or not, speaks the same language about the system.

---

# 🧱 Entities: identity matters

```csharp
public class Order
{
    public int Id { get; private set; } // identity

    public string Customer { get; private set; }
    public decimal Total { get; private set; }
}
```

👉 **Entity = an object defined by its identity, not by its values**

Two orders with the same customer and the same total are still **different orders** if they have different IDs. You already saw this distinction in the records post: entities use identity-based equality, unlike records, which use value-based equality

---

# 💎 Value Objects: when only the value matters

```csharp
public record Address(string Street, string City, string ZipCode);

public record Money(decimal Amount, string Currency)
{
    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Cannot add different currencies");

        return new Money(a.Amount + b.Amount, a.Currency);
    }
}
```

👉 **Value Object = an object entirely defined by its value, with no identity of its own**

Two `Address` instances with the same data **are the same address**, no matter where they were created. Here, `record` is the perfect tool — value-based equality is exactly the behavior a Value Object needs

---

# 🛡️ Aggregates: protecting consistency

```csharp
public class Order
{
    private readonly List<OrderItem> _items = new();
    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();

    public decimal Total => _items.Sum(i => i.Subtotal);

    public void AddItem(Product product, int quantity)
    {
        if (quantity <= 0)
            throw new ArgumentException("Quantity must be greater than zero");

        _items.Add(new OrderItem(product, quantity));
    }

    public void RemoveItem(int itemId)
    {
        var item = _items.FirstOrDefault(i => i.Id == itemId);
        if (item is null)
            throw new InvalidOperationException("Item not found in the order");

        _items.Remove(item);
    }
}
```

👉 **Aggregate = a set of related objects treated as a single unit of consistency, with a "root" (aggregate root) controlling access**

`Order` is the aggregate root. No one adds an `OrderItem` directly to the list from outside — everything goes through `Order`'s methods, which guarantee the rules (positive quantity, item must exist to be removed) are always respected

👉 This is encapsulation (from the classes and objects post) raised to the level of a whole set of objects, not just a single isolated class

---

# ⚙️ Domain Services: when a rule doesn't belong to a single entity

```csharp
public class TransferService
{
    public void Transfer(BankAccount source, BankAccount destination, decimal amount)
    {
        source.Withdraw(amount);
        destination.Deposit(amount);
    }
}
```

👉 Some business rules involve **more than one entity** and don't naturally belong to any single one of them. A Domain Service exists exactly for that — holding no state, just orchestrating a rule that spans multiple objects

---

# 📣 Domain Events: communicating what happened

```csharp
public record OrderCreatedEvent(int OrderId, string Customer);

public class Order
{
    public void Confirm()
    {
        Status = "Confirmed";
        // fires the event for whoever wants to react (send an email, update inventory...)
    }
}
```

👉 Remember `event` from the delegates post? Domain Events apply that same idea to the domain: "something important happened" (`OrderCreated`, `PaymentConfirmed`), and other parts of the system react without the entity needing to know who's listening — the same philosophy as the messaging you saw in the RabbitMQ post, here applied inside the domain itself

---

# 🗺️ Bounded Context: not everything is one single system

👉 **Bounded Context = a clear boundary within which a specific domain model makes sense**

The word "Customer" can mean different things in the **Sales** context (who buys) versus the **Support** context (who opens tickets). Instead of forcing a single giant `Customer` class that serves everything, DDD recognizes that each context can have its own model, focused on what matters for that slice of the system.

👉 This connects directly to what you'll learn about microservices: each Bounded Context is a natural candidate to become an independent service

---

# ⚠️ Common Mistakes

- Creating an "Anemic Domain Model": entities that are just `get`/`set`, with all the business logic scattered across external services — this is the opposite of what DDD proposes  
- Applying tactical DDD (Entities, Value Objects, Aggregates) without understanding the strategic DDD (Ubiquitous Language, Bounded Context) behind it  
- Modeling giant aggregates, trying to fit the entire system inside a single unit of consistency  
- Using DDD on simple, CRUD-like domains, where the extra complexity brings no real benefit  

---

# 📌 Conclusion

- Entities have identity; Value Objects are defined only by their value  
- Aggregates protect consistency through a root that controls access  
- Domain Services handle rules that span multiple entities  
- Domain Events communicate important changes without coupling whoever fires them to whoever reacts  
- Bounded Context recognizes that the same term can mean different things in different parts of the system  

👉 With DDD, your code stops being just a technical structure and starts truly telling the story of the business it represents

---

# 🔥 Next Step

Now that you know how to model complex domains, the next level is:

👉 **C# Fundamentals: Microservices — Introduction and When to Use Them**

Here you'll learn when (and when not) it's worth splitting an application into independent services.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
