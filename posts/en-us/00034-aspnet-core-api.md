# 🧠 C# Fundamentals: Building Your First API with ASP.NET Core

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- The entire foundation of the C# language  
- SOLID, design patterns, and automated testing  

All of that has prepared you for the most anticipated moment in this track:

👉 **Moving beyond isolated code and building a real web application**

Today you'll create your first API with **ASP.NET Core**, the .NET web framework used by the vast majority of companies that hire C# developers.

---

# 💡 What is a Web API?

👉 **API = an application that exposes functionality through HTTP endpoints, so other systems (front-ends, mobile apps, other services) can consume it**

ASP.NET Core is the framework that does the heavy lifting: it receives HTTP requests, routes them to the right code, and returns responses — all built on top of the CLR and .NET you already know from the early posts in this track.

---

# 🏗️ Creating the project

```bash
dotnet new webapi -o MyApi
cd MyApi
dotnet run
```

👉 The `webapi` template already ships with a working example endpoint and Swagger set up so you can test the API from the browser

---

# 🔀 Minimal APIs vs Controllers

Since .NET 6, there are two main ways to define endpoints.

## 🔹 Minimal API (directly in `Program.cs`)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", () => new[] { "Laptop", "Mouse", "Keyboard" });

app.Run();
```

👉 Ideal for small APIs or microservices — follows the same spirit as the **top-level statements** you saw in the first program post: less ceremony, straight to the point

## 🔹 Controllers (the traditional approach, great for larger APIs)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll()
    {
        var products = new[] { "Laptop", "Mouse", "Keyboard" };
        return Ok(products);
    }
}
```

👉 Controllers organize larger APIs better, grouping many related endpoints in the same class — this is the most common approach in corporate projects

---

# 📦 Receiving data: model binding

```csharp
public record CreateProductRequest(string Name, decimal Price);

[HttpPost]
public IActionResult Create([FromBody] CreateProductRequest request)
{
    // request.Name and request.Price are already populated from the request body
    return Ok(request);
}

[HttpGet("{id}")]
public IActionResult GetById([FromRoute] int id)
{
    return Ok($"Product {id}");
}

[HttpGet("search")]
public IActionResult Search([FromQuery] string term)
{
    return Ok($"Searching: {term}");
}
```

## 🔹 Where the data comes from

- `[FromBody]` → the request body (JSON), used in POST/PUT  
- `[FromRoute]` → part of the URL (`/products/5`)  
- `[FromQuery]` → the query string (`/products/search?term=laptop`)  

👉 Notice that `CreateProductRequest` is a `record` — exactly the feature you learned in the modern C# post, perfect for representing immutable input data

---

# 🔌 Built-in dependency injection

ASP.NET Core already ships with a ready-to-use dependency injection container — it's the direct practical application of what you saw in the SOLID and Repository posts:

```csharp
// Program.cs
builder.Services.AddSingleton<IRepository<Product>, InMemoryRepository<Product>>();
```

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IRepository<Product> _repository;

    public ProductsController(IRepository<Product> repository) // injected automatically
    {
        _repository = repository;
    }

    [HttpGet]
    public IActionResult GetAll() => Ok(_repository.ListAll());
}
```

👉 ASP.NET Core creates and hands over the `IRepository<Product>` instance automatically — you never write `new InMemoryRepository<Product>()` inside the controller. This is the Dependency Inversion Principle working in practice, with no manual wiring

---

# 📊 Proper HTTP responses

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repository.GetById(id);

    if (product is null)
        return NotFound(); // 404

    return Ok(product); // 200
}

[HttpPost]
public IActionResult Create(CreateProductRequest request)
{
    var product = new Product { Name = request.Name };
    _repository.Add(product);

    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product); // 201
}
```

## 🔹 The most commonly used codes

- `200 OK` → success, with a response body  
- `201 Created` → resource created, with its location in the header  
- `404 Not Found` → resource not found  
- `400 Bad Request` → invalid request (usually a validation failure)  

👉 Returning the correct HTTP status code isn't just formality — it's the contract that whoever consumes your API relies on to handle errors correctly

---

# 🧪 Testing with Swagger

The template already spins up an interactive interface at `/swagger`, where you can:

- See every available endpoint  
- Test requests directly from the browser  
- See the expected format of each request/response  

👉 It's the fastest way to validate your API without writing a front-end or using external tools while you develop

---

# ⚠️ Common Mistakes

- Exposing the domain class (`Product`) directly instead of a request DTO/record, mixing the data model with the API contract  
- Using synchronous methods for I/O operations (database, external calls) instead of `async`/`await`, wasting what you learned about asynchronous programming  
- Returning `200 OK` for everything, even error or "not found" cases  
- Putting business logic directly in the controller instead of delegating to service classes — controllers should stay thin  

---

# 📌 Conclusion

- ASP.NET Core exposes functionality through HTTP endpoints  
- Minimal APIs are lean; Controllers organize larger APIs better  
- `[FromBody]`, `[FromRoute]`, and `[FromQuery]` control where data comes from  
- ASP.NET Core's dependency injection applies DIP automatically, with no manual wiring  
- Correct HTTP status codes (`200`, `201`, `404`, `400`) are part of your API's contract  

👉 You just took the step that turns language knowledge into the ability to build real systems

---

# 🔥 Next Step

Now that you have a working API, the next level is:

👉 **C# Fundamentals: Entity Framework Core — Persisting Real Data**

Here you'll swap the in-memory repository for a real database, without breaking anything you've already built.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
