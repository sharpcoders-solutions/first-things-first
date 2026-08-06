# 🧠 C# Fundamentals: gRPC — Efficient Communication Between Services

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- When (and when not) to split an application into microservices  
- How services communicate via HTTP and messaging  

You've already built REST APIs with JSON — it works great for front-ends and public integrations. But for communication **between your own microservices**, there's a faster, more type-safe alternative.

👉 **Let's get to know gRPC**

---

# 💡 What is gRPC?

👉 **gRPC = a communication framework that uses Protocol Buffers (binary) instead of JSON, and HTTP/2 instead of HTTP/1.1**

| | REST + JSON | gRPC |
|---|---|---|
| Format | Text (JSON) | Binary (Protocol Buffers) |
| Protocol | HTTP/1.1 | HTTP/2 |
| Contract | Usually informal (Swagger helps) | Formal, defined in `.proto` |
| Performance | Good | Better (smaller payload, multiplexing) |

👉 The performance gain comes from two places: binary payloads are smaller than JSON, and HTTP/2 allows multiple calls over the same connection, without the overhead of opening a new one for every request

---

# 📝 Defining the contract: `.proto` files

```protobuf
// products.proto
syntax = "proto3";

service ProductService {
  rpc GetById (GetProductRequest) returns (ProductResponse);
  rpc ListAll (Empty) returns (ListProductsResponse);
}

message GetProductRequest {
  int32 id = 1;
}

message ProductResponse {
  int32 id = 1;
  string name = 2;
  double price = 3;
}

message Empty {}

message ListProductsResponse {
  repeated ProductResponse products = 1;
}
```

👉 This replaces the role that `record`s and DTOs play in a REST API — except here the **contract comes first**, and the C# is generated automatically from it, guaranteeing client and server never fall out of sync about the data format

---

# 🏗️ Implementing the service

```bash
dotnet new grpc -o MyGrpcService
```

```csharp
public class ProductServiceImpl : ProductService.ProductServiceBase
{
    private readonly IProductRepository _repository;

    public ProductServiceImpl(IProductRepository repository)
    {
        _repository = repository;
    }

    public override Task<ProductResponse> GetById(GetProductRequest request, ServerCallContext context)
    {
        var product = _repository.GetById(request.Id);

        return Task.FromResult(new ProductResponse
        {
            Id = product.Id,
            Name = product.Name,
            Price = (double)product.Price
        });
    }
}
```

```csharp
// Program.cs
app.MapGrpcService<ProductServiceImpl>();
```

👉 Notice `ProductServiceImpl` receives `IProductRepository` through its constructor — the same dependency injection mechanism you already master, applied to a gRPC service instead of a REST controller

---

# 📞 Consuming the service from another microservice

```csharp
// Consumer service's Program.cs
builder.Services.AddGrpcClient<ProductService.ProductServiceClient>(options =>
{
    options.Address = new Uri("https://product-service:5001");
});
```

```csharp
public class OrdersService
{
    private readonly ProductService.ProductServiceClient _productClient;

    public OrdersService(ProductService.ProductServiceClient productClient)
    {
        _productClient = productClient;
    }

    public async Task<ProductResponse> GetProduct(int id)
    {
        return await _productClient.GetByIdAsync(new GetProductRequest { Id = id });
    }
}
```

👉 The call looks like a local method — `_productClient.GetByIdAsync(...)` — but underneath it's a real network call, typed end to end by the `.proto` contract. Type errors that would only show up at runtime in a JSON API are caught at **compile time** here

---

# 🌊 Streaming: beyond request/response

```protobuf
service OrderService {
  rpc TrackStatus (TrackRequest) returns (stream StatusResponse);
}
```

```csharp
public override async Task TrackStatus(
    TrackRequest request, IServerStreamWriter<StatusResponse> responseStream, ServerCallContext context)
{
    while (!context.CancellationToken.IsCancellationRequested)
    {
        var status = await GetCurrentStatus(request.OrderId);
        await responseStream.WriteAsync(new StatusResponse { Status = status });
        await Task.Delay(2000);
    }
}
```

👉 Traditional REST is always one request followed by one response. gRPC supports **streaming**: the server can send multiple responses over time on the same connection — useful for status updates, real-time notifications, or large volumes of data sent in chunks

---

# 🔀 When to use gRPC vs REST

## 🔹 Use gRPC when:
- The communication is **between your own microservices**  
- Performance and low latency are critical  
- You want a strongly typed contract, checked at compile time  

## 🔹 Use REST when:
- The API needs to be consumed directly by **browsers** (limited gRPC support in the browser)  
- It's a **public** API, consumed by third parties expecting JSON and conventional HTTP  
- REST's simplicity and familiarity matter more than the performance gain  

---

# ⚠️ Common Mistakes

- Using gRPC for a public API consumed by web front-ends, without considering browser support limitations  
- Not versioning the `.proto` file, breaking old clients when changing an existing contract  
- Ignoring that gRPC requires HTTP/2, which may need extra configuration in some proxy/load balancer environments  
- Thinking gRPC replaces REST in every scenario — the choice depends on who's consuming the API  

---

# 📌 Conclusion

- gRPC uses Protocol Buffers (binary) and HTTP/2, more efficient than REST + JSON  
- The `.proto` contract generates C# code automatically, guaranteeing end-to-end typing  
- Streaming allows multiple responses over a single connection  
- gRPC shines for communication between microservices; REST remains the right choice for public APIs  

👉 With gRPC, communication between the services you learned to split in the previous post becomes faster and safer against contract mistakes

---

# 🔥 Next Step

Now that you know how to connect services efficiently, the next level is:

👉 **C# Fundamentals: Performance in C# (Span, Memory, and Benchmarking)**

Here you'll learn to measure and optimize your code's performance at the lowest level.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
