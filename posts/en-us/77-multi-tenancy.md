# 🧠 C# Fundamentals: Multi-tenancy

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Options Pattern for strongly-typed configuration  
- Entity Framework Core and JWT authentication  

Imagine a SaaS where each client company needs to see only its own data, same application, same deployment. That's multi-tenancy — a problem that shows up as soon as your product moves from "one company uses it" to "multiple companies use the same instance."

👉 **Let's learn Multi-tenancy**

---

# 💡 What is a tenant?

👉 **Tenant = a customer logically isolated within the same application**

If you build an order management system sold to multiple stores, each store is a tenant — store A should never see store B's orders, even while sharing the same application and (possibly) the same database

---

# 🏗️ Data isolation strategies

## 🔹 Database per tenant (full isolation)

```csharp
public string GetConnectionString(string tenantId) =>
    $"Server=myserver;Database=Company_{tenantId};...";
```

👉 Maximum isolation — each tenant has its own database. Safer, but operationally more expensive: migrations (post 32) need to run against N databases

## 🔹 Schema per tenant

```sql
CREATE SCHEMA Company_123;
CREATE TABLE Company_123.Orders (...);
```

👉 A middle ground: same database server, separate schemas per tenant

## 🔹 Discriminator column (shared database)

```csharp
public class Order
{
    public int Id { get; set; }
    public Guid TenantId { get; set; } // every table carries this column
    public decimal Amount { get; set; }
}
```

👉 Operationally cheaper, but requires total discipline: **every** query needs to filter by `TenantId`, no exceptions

---

# 🎯 Applying the filter automatically with EF Core

```csharp
public class AppDbContext : DbContext
{
    private readonly string _currentTenantId;

    public AppDbContext(DbContextOptions options, ITenantProvider tenantProvider)
        : base(options)
    {
        _currentTenantId = tenantProvider.GetTenantId();
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.Entity<Order>().HasQueryFilter(o => o.TenantId == _currentTenantId);
    }
}
```

👉 `HasQueryFilter` applies the tenant filter **automatically** to every LINQ query (post 19) — no one on the team needs to remember to manually write `.Where(o => o.TenantId == currentTenant)` on every query, eliminating the chance of a cross-tenant data leak due to an oversight

---

# 🔍 Identifying the tenant from the request

```csharp
public class TenantMiddleware
{
    private readonly RequestDelegate _next;

    public async Task InvokeAsync(HttpContext context, ITenantProvider tenantProvider)
    {
        var subdomain = context.Request.Host.Host.Split('.')[0]; // e.g.: company123.myapp.com

        tenantProvider.SetTenantId(subdomain);

        await _next(context);
    }
}
```

👉 Remember the middleware pipeline from the ASP.NET Core post? The tenant is usually identified by subdomain, a custom header, or a claim in the JWT (post 34) — resolved once at the start of the request, available for the rest of the pipeline

---

# ⚠️ Common Mistakes

- Forgetting `HasQueryFilter` on a new entity, creating a silent data leak between tenants  
- Scattering tenant identification logic throughout the code instead of centralizing it in a middleware  
- Choosing full isolation (database per tenant) without a real need, needlessly increasing operational complexity  
- Not specifically testing tenant isolation scenarios (remember the integration tests from post 59?) — this is the kind of bug that only shows up in production, with real data from two different customers  

---

# 📌 Conclusion

- Multi-tenancy logically isolates customers within the same application  
- Strategies range from full isolation (database per tenant) to shared (discriminator column)  
- EF Core's `HasQueryFilter` applies the tenant filter automatically, reducing leak risk  
- The tenant is identified once, early in the pipeline, and propagated for the rest of the request  

👉 With well-implemented multi-tenancy, a single application serves multiple customers with the same isolation confidence separate instances would give

---

# 🔥 Next Step

Now that you can isolate data between customers, the next level is:

👉 **C# Fundamentals: Kafka**

Here you'll learn stream-oriented messaging, an alternative to RabbitMQ for extremely high-volume scenarios.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
