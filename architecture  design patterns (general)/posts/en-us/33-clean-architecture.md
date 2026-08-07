# 🧠 C# Fundamentals: Clean Architecture in Practice

⏱️ Reading time: 9 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- SOLID and design patterns  
- How to build an API with ASP.NET Core  
- How to persist real data with EF Core  

You already have all the pieces. This post is about how to **organize** them in a way that survives the project's growth — the kind of structure you'll find in practically every serious C# project in the industry.

👉 **Let's bring it all together with Clean Architecture**

---

# 💡 What is Clean Architecture?

👉 **Clean Architecture = a way of organizing code into layers, where business rules never depend on technical details like a database or a web framework**

The idea was popularized by Robert C. Martin — the same "Uncle Bob" from the SOLID post. That's no coincidence: Clean Architecture is, in large part, the **Dependency Inversion Principle** applied to the entire project structure, not just a single class.

## 🔹 The Dependency Rule

👉 **Dependencies always point inward, toward the business rules — never the other way around**

```
API  →  Application  →  Domain
             ↑
      Infrastructure
```

The `Domain` (the center) doesn't know a database exists, an API exists, or a framework exists. Everything **technical** (EF Core, ASP.NET Core, email providers) lives at the edges, and depends on the center — never the reverse.

---

# 🧱 The four layers

## 🔹 1. Domain — the heart of the application

```csharp
// MyApi.Domain
public class Product
{
    public int Id { get; set; }
    public string Name { get; private set; }
    public decimal Price { get; private set; }

    public Product(string name, decimal price)
    {
        if (price <= 0)
            throw new ArgumentException("Price must be greater than zero");

        Name = name;
        Price = price;
    }

    public void ApplyDiscount(decimal percentage)
    {
        Price -= Price * percentage;
    }
}
```

👉 `Domain` contains the entities and business rules (you recognize this from the classes and objects post: encapsulation protecting the state). **No reference to EF Core, ASP.NET, or anything external**

## 🔹 2. Application — the use cases

```csharp
// MyApi.Application
public interface IProductRepository
{
    void Add(Product product);
    Product GetById(int id);
    IEnumerable<Product> ListAll();
}

public class CreateProductUseCase
{
    private readonly IProductRepository _repository;

    public CreateProductUseCase(IProductRepository repository)
    {
        _repository = repository;
    }

    public Product Execute(string name, decimal price)
    {
        var product = new Product(name, price);
        _repository.Add(product);
        return product;
    }
}
```

👉 `Application` orchestrates the system's use cases ("create product," "apply discount") and defines **interfaces** for everything external — like `IProductRepository`. Notice: the interface lives here, not in infrastructure. That's the Dependency Rule in practice

## 🔹 3. Infrastructure — the technical details

```csharp
// MyApi.Infrastructure
public class ProductRepositoryEfCore : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepositoryEfCore(AppDbContext context)
    {
        _context = context;
    }

    public void Add(Product product) => _context.Products.Add(product);
    public Product GetById(int id) => _context.Products.Find(id);
    public IEnumerable<Product> ListAll() => _context.Products.ToList();
}
```

👉 `Infrastructure` implements the interfaces defined in `Application`, using EF Core, external APIs, file systems — everything that's a "detail." This layer **depends on** `Application`, never the other way around

## 🔹 4. API (Presentation) — the entry point

```csharp
// MyApi.Api
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly CreateProductUseCase _createProduct;

    public ProductsController(CreateProductUseCase createProduct)
    {
        _createProduct = createProduct;
    }

    [HttpPost]
    public IActionResult Create(CreateProductRequest request)
    {
        var product = _createProduct.Execute(request.Name, request.Price);
        return CreatedAtAction(nameof(Create), new { id = product.Id }, product);
    }
}
```

👉 The controller doesn't know about EF Core, and doesn't know how the product gets saved — it just calls the use case. Compared to the ASP.NET Core post, where the controller talked directly to the repository, there's now a business orchestration layer between the two

---

# 🔌 Wiring it all together: dependency injection

```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options => options.UseSqlServer(connectionString));

builder.Services.AddScoped<IProductRepository, ProductRepositoryEfCore>();
builder.Services.AddScoped<CreateProductUseCase>();
```

👉 ASP.NET Core assembles the whole chain: when the controller asks for a `CreateProductUseCase`, the container hands it one already wired with `IProductRepository` — which, in turn, already comes with `AppDbContext` ready to go. It's all the dependency injection you already learned, just orchestrating four layers instead of two loose classes

---

# 🗂️ Folder / project structure

```
MyApi.sln
├── MyApi.Domain          (no dependencies)
├── MyApi.Application     (depends on Domain)
├── MyApi.Infrastructure  (depends on Application and Domain)
└── MyApi.Api             (depends on Application, Infrastructure, and Domain)
```

👉 In real projects, each layer is usually a **separate .NET project**, and the compiler itself prevents references in the wrong direction — if `Domain` tries to reference `Infrastructure`, the build fails. The architecture becomes a rule enforced by the tooling, not just a team convention

---

# 🔗 How it all connects to what you already learned

| Layer | Concepts you've already seen |
|---|---|
| **Domain** | Classes, encapsulation, constructors validating state |
| **Application** | Interfaces, Dependency Inversion, use cases as Strategy |
| **Infrastructure** | EF Core, concrete implementation of the Repository pattern |
| **API** | ASP.NET Core, controllers, model binding, HTTP responses |

👉 Clean Architecture doesn't introduce new concepts — it's the **organization** of everything you already master, each piece in its right place

---

# ⚠️ Common Mistakes

- Applying Clean Architecture to a project too small for it, creating four projects for an application that would fit in a single file  
- Letting `Domain` reference EF Core (e.g., using Entity Framework attributes directly on domain entities)  
- Putting business logic in the controller "because it's faster," emptying out the purpose of the `Application` layer  
- Thinking Clean Architecture is a fixed folder structure — what matters is the **direction of dependencies**, not the exact layer names  

---

# 📌 Conclusion

- The Dependency Rule: everything points inward, toward the business rules  
- `Domain` depends on nothing; `Infrastructure` implements what `Application` defines  
- Interfaces live close to the center; concrete implementations live at the edges  
- Nothing here is a new concept — it's everything you've already learned, organized with intent  

👉 With Clean Architecture, your project stops being "just an API that saves to a database" and becomes a system where swapping the database, the web framework, or even how users get notified never threatens the core business rule

---

# 🔥 Next Step

Now that you know how to structure a professional application end to end, the next level is:

👉 **C# Fundamentals: Authentication and Authorization with JWT**

Here you'll learn to secure your API, making sure only authenticated (and authorized) users can access each endpoint.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
