# 🧠 C# Fundamentals: Messaging with RabbitMQ

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Resilience with Polly for synchronous calls  
- Caching to reduce load from repeated queries  

Up until now, every external system your API calls is synchronous: you ask, you wait, you get a response. But what about when the response can wait? Sending a confirmation email doesn't need to hold up the response to the user's order.

👉 **That's where messaging comes in**

---

# 💡 The problem with synchronous communication

```csharp
[HttpPost]
public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
{
    var order = await _orderService.CreateAsync(request);

    await _emailService.SendConfirmationAsync(order);      // what if the email service is slow?
    await _inventoryService.UpdateAsync(order);              // what if the inventory service is down?

    return Ok(order);
}
```

👉 The user waits for the entire flow — create + notify + update inventory — even though only creating the order is actually urgent. If any of these steps fails or takes too long, the whole experience suffers

---

# 📬 What is a message queue?

👉 **Message queue = an intermediary that receives messages from a producer and delivers them to one or more consumers, asynchronously and decoupled**

```
Producer → [ Queue (RabbitMQ) ] → Consumer
```

The producer publishes the message and **moves on immediately** — it doesn't wait for the consumer to process it. This is the same idea as the `async`/`await` you already know, just applied between **different systems**, not between methods in the same process.

---

# 🏗️ Publishing a message

```bash
dotnet add package RabbitMQ.Client
```

```csharp
public class OrderPublisher
{
    public void Publish(OrderCreatedEvent evt)
    {
        var factory = new ConnectionFactory { HostName = "localhost" };

        using var connection = factory.CreateConnection();
        using var channel = connection.CreateModel();

        channel.QueueDeclare(queue: "orders-created", durable: true, exclusive: false, autoDelete: false);

        var body = Encoding.UTF8.GetBytes(JsonSerializer.Serialize(evt));

        channel.BasicPublish(exchange: "", routingKey: "orders-created", body: body);
    }
}
```

```csharp
[HttpPost]
public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
{
    var order = await _orderService.CreateAsync(request);

    _publisher.Publish(new OrderCreatedEvent(order.Id, order.Customer));

    return Ok(order); // responds immediately, without waiting for email or inventory
}
```

👉 The API responds to the user as soon as the order is created. Sending the email and updating inventory happen **afterward**, independently, consumed by a separate process

---

# 📥 Consuming messages

```csharp
public class EmailConsumer : BackgroundService
{
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var factory = new ConnectionFactory { HostName = "localhost" };
        var connection = factory.CreateConnection();
        var channel = connection.CreateModel();

        channel.QueueDeclare(queue: "orders-created", durable: true, exclusive: false, autoDelete: false);

        var consumer = new EventingBasicConsumer(channel);
        consumer.Received += async (model, evt) =>
        {
            var message = Encoding.UTF8.GetString(evt.Body.ToArray());
            var orderEvent = JsonSerializer.Deserialize<OrderCreatedEvent>(message);

            await _emailService.SendConfirmationAsync(orderEvent);

            channel.BasicAck(evt.DeliveryTag, multiple: false);
        };

        channel.BasicConsume(queue: "orders-created", autoAck: false, consumer: consumer);

        return Task.CompletedTask;
    }
}
```

👉 `BackgroundService` runs continuously, listening to the queue. `BasicAck` confirms the message was processed successfully — if the consumer crashes before acknowledging, the message **goes back to the queue** and gets delivered again

---

# 🔀 Queues vs direct HTTP calls: when to use each

## 🔹 Use a direct HTTP call when:
- The response is needed **immediately** by the user  
- The operation is simple and fast  

## 🔹 Use a message queue when:
- The operation can happen **later**, without blocking the user  
- You want a temporary failure in the consumer to not affect the producer  
- Multiple systems need to react to the same event (email, inventory, analytics, all consuming the same message)  

👉 The queue also works as a **buffer**: if the consumer goes down for a few minutes, messages just pile up and get processed once it's back — nothing gets lost

---

# ⚠️ Common Mistakes

- Using `autoAck: true` without thinking through the consequences: if the process crashes mid-processing, the message is lost, not reprocessed  
- Putting logic that needs an immediate response (e.g., validating payment) inside an asynchronous queue  
- Not handling duplicate messages — queues generally guarantee "at least once delivery," so the consumer must be able to process the same message twice without side effects  
- Ignoring consumer failures, letting messages pile up indefinitely with no alert  

---

# 📌 Conclusion

- Message queues decouple producer and consumer in time, without requiring an immediate response  
- Publishing a message frees the API to respond to the user without waiting on secondary tasks  
- `BasicAck`/`autoAck: false` guarantees unprocessed messages go back to the queue  
- Queues work as a buffer: consumers being down doesn't bring down the whole system  

👉 With messaging, your system stops depending on everything happening on the same synchronous timeline, gaining resilience and scalability

---

# 🔥 Next Step

Now that you know how to decouple systems with queues, the next level is:

👉 **C# Fundamentals: CQRS and MediatR**

Here you'll learn to separate write and read operations, organizing your application's business logic even better.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
