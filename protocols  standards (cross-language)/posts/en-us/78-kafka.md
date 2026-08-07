# 🧠 C# Fundamentals: Kafka

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Multi-tenancy for isolating customers in the same application  
- RabbitMQ (post 41) for messaging between services  

RabbitMQ is excellent for task queues and point-to-point communication. But when volume reaches millions of events per second, and you need multiple independent consumers reading the **same** event stream, that's where Kafka comes in.

👉 **Let's learn Kafka**

---

# 💡 Kafka isn't a queue, it's a distributed log

👉 **Kafka = an immutable, ordered event log, which multiple consumers can read independently**

## 🔹 RabbitMQ (post 41): message consumed, disappears from the queue

```
Queue: [msg1, msg2, msg3]
Consumer reads msg1 → queue: [msg2, msg3] (msg1 is gone)
```

## 🔹 Kafka: message persists in the log

```
Topic: [event1, event2, event3, event4, ...]
Consumer A is at offset 2
Consumer B is at offset 4
Both can reread the entire history, independently
```

👉 This is reminiscent of Event Sourcing from post 56 — Kafka is literally an event log, and each consumer keeps its own pointer (offset) of where it stopped reading

---

# 🏗️ Producing events

```bash
dotnet add package Confluent.Kafka
```

```csharp
var config = new ProducerConfig { BootstrapServers = "localhost:9092" };

using var producer = new ProducerBuilder<string, string>(config).Build();

await producer.ProduceAsync("orders-created", new Message<string, string>
{
    Key = order.Id.ToString(),
    Value = JsonSerializer.Serialize(order)
});
```

👉 The `Key` determines the partition — messages with the same key always go to the same partition, guaranteeing order for that specific set of events (for example, all events for a single order)

---

# 🎯 Consuming events

```csharp
var config = new ConsumerConfig
{
    BootstrapServers = "localhost:9092",
    GroupId = "notifications-service",
    AutoOffsetReset = AutoOffsetReset.Earliest
};

using var consumer = new ConsumerBuilder<string, string>(config).Build();
consumer.Subscribe("orders-created");

while (true)
{
    var result = consumer.Consume();
    var order = JsonSerializer.Deserialize<Order>(result.Message.Value);

    Console.WriteLine($"Notifying about order {order.Id}");
    consumer.Commit(result);
}
```

👉 `GroupId` groups consumers — multiple instances with the same `GroupId` split the partitions among themselves (horizontal scalability), while different groups read the same topic completely independently

---

# 🔀 Partitions: real parallelism

```
Topic "orders-created" with 3 partitions:
  Partition 0: [event1, event4, event7, ...]
  Partition 1: [event2, event5, event8, ...]
  Partition 2: [event3, event6, event9, ...]
```

👉 Each partition can be processed in parallel by different consumers in the same group — this is what lets Kafka scale to millions of events per second, something a traditional queue (post 41) wasn't designed to do

---

# ⚖️ Kafka vs RabbitMQ: when to use each

## 🔹 RabbitMQ
- Task queues, commands that should be processed once  
- Complex message routing (exchanges, routing keys)  
- Moderate volume  

## 🔹 Kafka
- High-volume event streams (clicks, logs, metrics, IoT)  
- Multiple independent consumers need to reread the same stream  
- Cases that approach Event Sourcing (post 56) at scale  

---

# ⚠️ Common Mistakes

- Using Kafka for simple task queues, when RabbitMQ would solve it with much less operational complexity  
- Choosing a poor partition `Key`, concentrating all the volume in a single partition and losing parallelism  
- Not committing the offset correctly, reprocessing duplicate events or losing events  
- Underestimating Kafka's operational complexity — it requires Zookeeper (or KRaft) and more infrastructure than a traditional queue  

---

# 📌 Conclusion

- Kafka is a distributed log, not a traditional queue — events persist and can be reread  
- Multiple consumer groups read the same topic independently  
- Partitions enable real parallelism, essential for very high volumes  
- RabbitMQ and Kafka solve different problems — the choice depends on the consumption pattern, not just volume  

👉 With Kafka, systems that need to process events at massive scale gain infrastructure specifically designed for it

---

# 🔥 Next Step

Now that you've mastered messaging at scale, the next level is:

👉 **C# Fundamentals: Event-Driven Architecture**

Here you'll learn to connect everything you've learned about events — Event Sourcing, Saga, Outbox, and Kafka — into a cohesive architecture.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
