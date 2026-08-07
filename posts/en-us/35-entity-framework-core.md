# 🧠 C# Fundamentals: Entity Framework Core — Persisting Real Data

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to build an API with ASP.NET Core  
- The Repository pattern and dependency injection  

Your API already works, but the data disappears every time you restart the application — because it only lives in a list in memory. Time to fix that for real.

👉 **Let's swap memory for a real database, using Entity Framework Core**

---

# 💡 What is Entity Framework Core?

👉 **EF Core = an ORM (Object-Relational Mapper) that translates your C# classes into database tables, and back**

Instead of writing SQL by hand, you work with objects and collections — EF Core handles the translation:

```csharp
var products = context.Products.Where(p => p.Price > 100).ToList();
```

👉 That line is pure LINQ syntax, from the post you already saw — except now, under the hood, EF Core turns it into a real SQL query

---

# 🏗️ Installing the packages

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
```

👉 The first package is the **provider** (here, SQL Server — it could be PostgreSQL, SQLite, MySQL...). The second enables migration support from the command line

---

# 🧱 Defining the `DbContext`

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products { get; set; }
}
```

## 🔹 The two main pieces

- `DbContext` → represents the session with the database  
- `DbSet<T>` → represents a table, with all the query operations you already know from LINQ  

👉 Notice the constructor taking `DbContextOptions` — that's dependency injection again, exactly like you saw in the ASP.NET Core post

---

# 🔌 Registering the `DbContext` in ASP.NET Core

```csharp
// Program.cs
var connectionString = builder.Configuration.GetConnectionString("Default");

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

```json
// appsettings.json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyApiDb;Trusted_Connection=True;"
  }
}
```

👉 `AddDbContext` registers the context in the DI container with a `Scoped` lifetime by default — a new instance per HTTP request, preventing one user's data from leaking into another's

---

# 🚀 Migrations: versioning the database schema

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## 🔹 What each command does

- `migrations add` → generates a C# code file describing the schema changes  
- `database update` → applies those changes to the real database  

👉 Migrations work like "Git for the database" — every model change becomes a versioned, reproducible record that can be applied in any environment

---

# ✍️ CRUD operations with EF Core

```csharp
// Create
context.Products.Add(new Product { Name = "Laptop", Price = 3500 });
await context.SaveChangesAsync();

// Read
var product = await context.Products.FirstOrDefaultAsync(p => p.Id == 1);
var all = await context.Products.ToListAsync();

// Update
product.Price = 2999;
await context.SaveChangesAsync();

// Delete
context.Products.Remove(product);
await context.SaveChangesAsync();
```

👉 Notice the `async`/`await` on every database operation — exactly what you learned in the asynchronous programming post. Database access is I/O, and blocking the thread while waiting for SQL Server to respond wastes resources

---

# 🗄️ Implementing the Repository with EF Core

Remember `IRepository<T>` from the design patterns post? Now it gets a real implementation:

```csharp
public class EfCoreRepository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;

    public EfCoreRepository(AppDbContext context)
    {
        _context = context;
    }

    public void Add(T item) => _context.Set<T>().Add(item);
    public T GetById(int id) => _context.Set<T>().Find(id);
    public IEnumerable<T> ListAll() => _context.Set<T>().ToList();
}
```

```csharp
// Program.cs — a one-line swap, nothing else changes
builder.Services.AddScoped<IRepository<Product>, EfCoreRepository<Product>>();
// used to be: AddSingleton<IRepository<Product>, InMemoryRepository<Product>>();
```

👉 The `ProductsController` from the previous post **doesn't change a single line** — it depends on the `IRepository<T>` interface, never on the concrete implementation. This is the definitive proof that the Dependency Inversion Principle isn't just theory: it's what lets you swap "memory" for "real database" without touching the rest of the system

---

# ⚠️ Common Mistakes

- Registering the `DbContext` as `Singleton` instead of `Scoped`, causing concurrency problems across requests  
- Using synchronous methods (`ToList()`, `SaveChanges()`) instead of the async versions in API code  
- Forgetting to run `dotnet ef database update` after creating a migration, then wondering why the table doesn't exist  
- Falling into the N+1 query problem: fetching a list and then running a separate query for each item, instead of using `Include()` to load related data at once  

---

# 📌 Conclusion

- EF Core translates C# classes into database tables, using LINQ for queries  
- `DbContext` represents the session; `DbSet<T>` represents each table  
- Migrations version the database schema in a reproducible way  
- Every database operation should be `async`, following what you already learned about I/O  
- Swapping the in-memory Repository for the EF Core-backed one doesn't require changing the controller — DIP in practice  

👉 With EF Core, your API stops losing data on every restart and starts persisting real information, with all the design you've built throughout this track staying intact

---

# 🔥 Next Step

Now that your application persists real data, the next level is:

👉 **C# Fundamentals: Extension Methods and Custom LINQ**

Here you'll understand the mechanism behind the LINQ you've used since post 19, and learn to create your own chainable operators.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
