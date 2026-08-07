# 🧠 C# Fundamentals: CQRS and MediatR

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Messaging to decouple systems in time  
- The Repository pattern and dependency injection, organizing the application into layers  

As the application grows, use cases multiply, and service classes start accumulating a mix of read and write methods. There's a pattern that separates this even more explicitly.

👉 **Let's get to know CQRS and the MediatR library**

---

# 💡 What is CQRS?

👉 **CQRS = Command Query Responsibility Segregation — separating operations that change state (commands) from those that only read data (queries)**

- **Command** → "create order," "update product," "cancel subscription" — changes the system, usually returns nothing beyond a confirmation  
- **Query** → "get order by id," "list products" — only reads, never modifies anything  

👉 This separation exists because reads and writes usually have very different needs: writes need validation and business rules; reads need performance and formats optimized for display

---

# 📬 The Mediator pattern: a step before MediatR

👉 **Mediator = a central object that receives a request and forwards it to the right handler, without the sender needing to know who processes it**

```
Controller → Mediator → Specific handler
```

This is, in practice, an application of the **Dependency Inversion Principle**: the controller only depends on the mediator (an abstraction), never directly on each individual handler.

---

# 🏗️ Installing MediatR

```bash
dotnet add package MediatR
```

```csharp
// Program.cs
builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));
```

---

# ✍️ Defining a Command

```csharp
public record CreateProductCommand(string Name, decimal Price) : IRequest<int>;

public class CreateProductHandler : IRequestHandler<CreateProductCommand, int>
{
    private readonly IProductRepository _repository;

    public CreateProductHandler(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<int> Handle(CreateProductCommand command, CancellationToken cancellationToken)
    {
        var product = new Product(command.Name, command.Price);
        await _repository.AddAsync(product);
        return product.Id;
    }
}
```

👉 Notice the `record` in the command — exactly the feature from the modern C# post, perfect for representing an immutable intent to change state

## 🔹 Using the Command in the controller

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateProductCommand command)
{
    var id = await _mediator.Send(command);
    return CreatedAtAction(nameof(GetById), new { id }, null);
}
```

👉 The controller doesn't know `CreateProductHandler` exists — it just sends the command through `IMediator` and gets back the result. The same decoupling you already saw in the SOLID post (the D in Dependency Inversion), with an even more explicit syntax

---

# 🔍 Defining a Query

```csharp
public record GetProductQuery(int Id) : IRequest<ProductDto>;

public class GetProductHandler : IRequestHandler<GetProductQuery, ProductDto>
{
    private readonly AppDbContext _context;

    public GetProductHandler(AppDbContext context)
    {
        _context = context;
    }

    public async Task<ProductDto> Handle(GetProductQuery query, CancellationToken cancellationToken)
    {
        return await _context.Products
            .Where(p => p.Id == query.Id)
            .Select(p => new ProductDto(p.Id, p.Name, p.Price))
            .FirstOrDefaultAsync(cancellationToken);
    }
}
```

👉 The query can go straight to the `DbContext` and use LINQ to project exactly the fields it needs (`ProductDto`), without going through the entity's business rules — reads get a freedom that writes don't

---

# 🔌 Pipeline Behaviors: intercepting every request

One of MediatR's biggest advantages is being able to intercept **every** command/query that flows through it:

```csharp
public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : notnull
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;

    public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger)
    {
        _logger = logger;
    }

    public async Task<TResponse> Handle(
        TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Processing {Name}", typeof(TRequest).Name);
        var response = await next();
        _logger.LogInformation("Completed {Name}", typeof(TRequest).Name);
        return response;
    }
}
```

👉 This applies logging (from the Serilog post) automatically to **every** command and query, without repeating code in each handler — the same principle works for validation, caching, and database transactions

---

# 🔀 CQRS doesn't require two databases

👉 A common mistake is thinking CQRS always means separate databases for reads and writes. In most projects, **commands and queries use the same database** — CQRS here is just a separation of responsibility in the code, not in infrastructure. Splitting databases is an optional evolution, useful only in systems with extreme scaling needs.

---

# ⚠️ Common Mistakes

- Thinking CQRS requires separate databases from day one  
- Creating a giant handler that reads and writes at the same time, losing the point of the separation  
- Using MediatR for everything, even trivial operations that don't need the indirection  
- Putting heavy business logic inside a query handler, when queries should stay simple and fast  

---

# 📌 Conclusion

- CQRS separates write operations (commands) from read operations (queries)  
- MediatR decouples whoever sends the request from whoever processes it, via `IMediator`  
- Pipeline behaviors apply cross-cutting behavior (logging, validation) to every handler  
- CQRS doesn't require separate infrastructure — it's usually just code organization  

👉 With CQRS and MediatR, your application gains an even clearer separation between what changes the system and what just queries it

---

# 🔥 Next Step

Now that you know how to separate commands from queries, the next level is:

👉 **C# Fundamentals: Value Types vs Reference Types in Depth**

Here you'll understand the fundamental difference between struct and class, and how it affects performance and behavior in every handler you just wrote.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
