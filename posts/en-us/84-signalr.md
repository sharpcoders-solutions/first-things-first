# 🧠 C# Fundamentals: SignalR

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Custom awaiters and how `await` works under the hood  
- The entire request/response model since post 34 — the client always asks, the server always answers  

What if the server needs to tell the client something, without waiting for a question? A chat, a notification that an order was updated, a live scoreboard — none of that fits well into request/response. SignalR solves this.

👉 **Let's learn SignalR**

---

# 💡 The problem with traditional request/response

```csharp
// ❌ The client has to keep asking repeatedly
setInterval(async () => {
    const response = await fetch('/orders/123/status');
    // checks if it changed...
}, 5000);
```

👉 Polling (asking repeatedly) wastes requests and still has delay — the client only finds out about the change on the next check, not the instant it happens

---

# 🏗️ Setting up a SignalR Hub

```bash
dotnet add package Microsoft.AspNetCore.SignalR
```

```csharp
public class OrdersHub : Hub
{
    public async Task JoinGroup(string orderId)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, $"order-{orderId}");
    }
}
```

```csharp
// Program.cs
builder.Services.AddSignalR();

// ...

app.MapHub<OrdersHub>("/hubs/orders");
```

👉 A `Hub` keeps connections open (via WebSockets, with automatic fallback) — unlike a REST controller (post 34), which responds and closes the connection, the Hub keeps the channel open for the server to speak whenever it wants

---

# 🎯 Sending updates from the server to the client

```csharp
public class OrdersService
{
    private readonly IHubContext<OrdersHub> _hubContext;

    public async Task UpdateStatus(int orderId, string newStatus)
    {
        await _repository.UpdateStatus(orderId, newStatus);

        await _hubContext.Clients
            .Group($"order-{orderId}")
            .SendAsync("StatusUpdated", newStatus);
    }
}
```

👉 It's the same spirit behind any event-driven system — when something happens on the server, it **pushes** the notification, instead of waiting for the client to ask

---

# 🔍 Consuming it on the client side

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hubs/orders")
    .build();

connection.on("StatusUpdated", (newStatus) => {
    console.log(`Order updated: ${newStatus}`);
});

await connection.start();
await connection.invoke("JoinGroup", "123");
```

👉 The client registers in a specific group (order 123), and receives real-time updates only for that order — no asking, no polling

---

# 🔄 Transports: WebSockets with automatic fallback

```
1st attempt: WebSockets (bidirectional, most efficient)
2nd attempt: Server-Sent Events (if WebSockets isn't available)
3rd attempt: Long Polling (maximum compatibility)
```

👉 SignalR automatically picks the best available transport — you write the code once, and it adapts to the client's environment (corporate proxy blocking WebSockets, old browser, etc.)

---

# ⚖️ SignalR vs the previous patterns

## 🔹 REST (post 34)
- Client asks, server answers — ideal for one-off operations  

## 🔹 Asynchronous REST with `Task`/`ValueTask` (posts 81–83)
- Client still waits for a synchronous reply to the request — only the internal efficiency changes  

## 🔹 SignalR
- Server pushes data without the client asking — ideal for real time  

👉 They don't compete with each other — a real application often combines all three: REST/GraphQL for normal operations, SignalR only for the flows that need instant updates

---

# ⚠️ Common Mistakes

- Using SignalR for everything, when most operations don't need real time — REST remains simpler for that  
- Not scaling correctly across multiple instances — SignalR needs a backplane (Redis) to sync connections across different servers  
- Forgetting to clean up groups/connections on disconnect, accumulating stale state  
- Sending sensitive data without authentication on the Hub — remember JWT (post 37), Hubs need authorization too  

---

# 📌 Conclusion

- SignalR lets the server push data to the client, without polling  
- Hubs keep connections open via WebSockets, with automatic fallback  
- Groups let you send updates only to clients interested in a specific resource  
- SignalR complements REST/GraphQL, it doesn't replace them — each solves a different communication pattern  

👉 With SignalR, your application gains a truly bidirectional communication channel, essential for real-time experiences

---

# 🔥 Next Step

Now that you've mastered real-time communication, the next level is:

👉 **C# Fundamentals: Blazor — Introduction**

Here you'll learn to build entire web interfaces using C# instead of JavaScript.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
