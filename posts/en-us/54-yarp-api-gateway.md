# 🧠 C# Fundamentals: API Gateway with YARP

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Rate Limiting to protect a single API  
- Microservices — architecture with multiple independent services  

When your architecture grows to several services (remember the microservices post?), the client shouldn't need to know each one's address. That's where an API Gateway comes in.

👉 **Let's learn to build a Gateway with YARP (Yet Another Reverse Proxy)**

---

# 💡 What is an API Gateway?

👉 **API Gateway = a single entry point that routes requests to the correct services behind it**

Without a gateway, the client needs to know:

```
GET http://orders-service:5001/orders/123
GET http://inventory-service:5002/inventory/456
GET http://payment-service:5003/payments/789
```

With a gateway:

```
GET http://api.mycompany.com/orders/123
GET http://api.mycompany.com/inventory/456
GET http://api.mycompany.com/payments/789
```

👉 The gateway knows the internal topology; the client only knows **one** address

---

# 🏗️ Setting up YARP

```bash
dotnet add package Yarp.ReverseProxy
```

```csharp
// Program.cs
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

// ...

app.MapReverseProxy();
```

```json
// appsettings.json
{
  "ReverseProxy": {
    "Routes": {
      "orders-route": {
        "ClusterId": "orders-cluster",
        "Match": { "Path": "/orders/{**catch-all}" }
      },
      "inventory-route": {
        "ClusterId": "inventory-cluster",
        "Match": { "Path": "/inventory/{**catch-all}" }
      }
    },
    "Clusters": {
      "orders-cluster": {
        "Destinations": {
          "destination1": { "Address": "http://orders-service:5001" }
        }
      },
      "inventory-cluster": {
        "Destinations": {
          "destination1": { "Address": "http://inventory-service:5002" }
        }
      }
    }
  }
}
```

👉 The whole routing configuration lives declaratively in `appsettings.json` — no manual routing code

---

# 🎯 Centralizing cross-cutting concerns

## 🔹 Rate Limiting at the Gateway, not in each service

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("global", opts =>
    {
        opts.PermitLimit = 1000;
        opts.Window = TimeSpan.FromMinutes(1);
    });
});
```

👉 Remember the previous post? Instead of configuring rate limiting in every microservice, you centralize it in the gateway — one single place protects all the services behind it

## 🔹 Centralized authentication

```csharp
app.MapReverseProxy().RequireAuthorization();
```

👉 JWT (post 34) is validated once at the gateway. The internal services can trust that every incoming request has already been authenticated

## 🔹 Load balancing across multiple instances

```json
"orders-cluster": {
  "LoadBalancingPolicy": "RoundRobin",
  "Destinations": {
    "destination1": { "Address": "http://orders-service-1:5001" },
    "destination2": { "Address": "http://orders-service-2:5001" }
  }
}
```

👉 If you scaled the orders service horizontally (multiple instances running, remember the Docker post?), the gateway automatically distributes load between them

---

# 🔄 Request transformations

```json
"orders-route": {
  "ClusterId": "orders-cluster",
  "Match": { "Path": "/api/v2/orders/{**catch-all}" },
  "Transforms": [
    { "PathPattern": "/orders/{**catch-all}" }
  ]
}
```

👉 The external client calls `/api/v2/orders`, but the gateway rewrites it to `/orders` internally — API versioning (remember post 48?) can live entirely at the gateway layer, without forcing changes on internal services

---

# ⚠️ Common Mistakes

- Putting business logic in the gateway, which should be a purely routing and cross-cutting-concerns layer  
- Not configuring timeouts, letting a slow service hang requests at the gateway indefinitely  
- Forgetting that the gateway is a single point of failure — without high availability there, the whole architecture goes down with it  
- Duplicating authentication in each internal service after already centralizing it at the gateway, creating unnecessary complexity  

---

# 📌 Conclusion

- An API Gateway centralizes routing to multiple services behind a single address  
- YARP configures routes and clusters declaratively, with no manual routing code  
- Rate limiting, authentication, and load balancing are centralized at the gateway, not duplicated in each service  
- Path transformations decouple the public API from the internal structure of the services  

👉 With a Gateway, your microservices architecture gets a single, consistent entry point that's easier to secure

---

# 🔥 Next Step

Now that you have a centralized entry point, the next level is:

👉 **C# Fundamentals: Observability with OpenTelemetry**

Here you'll learn to trace a request across multiple services, from the gateway all the way to the database.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
