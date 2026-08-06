# 🧠 C# Fundamentals: GraphQL with HotChocolate

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Event-driven architecture, connecting everything about messaging  
- REST APIs since post 31, with fixed endpoints per resource  

In REST, each endpoint returns a fixed data structure — if the client only needs the customer's name, but the endpoint returns the entire order, that's wasted bandwidth. GraphQL solves this by letting the client choose exactly what it wants.

👉 **Let's learn GraphQL with HotChocolate**

---

# 💡 REST vs GraphQL

## 🔹 REST (post 31)

```
GET /orders/123
```

```json
{
  "id": 123,
  "amount": 150.00,
  "status": "Confirmed",
  "customer": { "id": 1, "name": "Maria", "email": "maria@email.com", "address": {...} },
  "items": [...]
}
```

👉 The client receives **everything**, even if it only needs the `status`

## 🔹 GraphQL

```graphql
query {
  order(id: 123) {
    status
  }
}
```

```json
{ "order": { "status": "Confirmed" } }
```

👉 The client asks for exactly the fields it needs — nothing more, nothing less

---

# 🏗️ Setting up HotChocolate

```bash
dotnet add package HotChocolate.AspNetCore
```

```csharp
public class Query
{
    public async Task<Order?> GetOrder(int id, [Service] AppDbContext context) =>
        await context.Orders.Include(o => o.Customer).FirstOrDefaultAsync(o => o.Id == id);

    public async Task<List<Order>> ListOrders([Service] AppDbContext context) =>
        await context.Orders.ToListAsync();
}
```

```csharp
// Program.cs
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>();

// ...

app.MapGraphQL();
```

👉 The `Query` class defines the read entry points — similar to a controller (post 31), but HotChocolate automatically builds the GraphQL schema from it

---

# 🎯 Resolving relationships on demand

```csharp
public class Order
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public string Status { get; set; } = default!;

    [GraphQLIgnore]
    public int CustomerId { get; set; }

    public async Task<Customer> GetCustomer([Service] AppDbContext context) =>
        await context.Customers.FindAsync(CustomerId);
}
```

👉 The `Customer` field is only resolved (running the database query) if the API client actually asks for it in the GraphQL query — unlike REST, where EF Core's `Include` (post 32) always brings related data, whether it's used or not

---

# ✍️ Mutations: writing data

```csharp
public class Mutation
{
    public async Task<Order> CreateOrder(CreateOrderInput input, [Service] AppDbContext context)
    {
        var order = new Order { Amount = input.Amount, CustomerId = input.CustomerId };
        context.Orders.Add(order);
        await context.SaveChangesAsync();
        return order;
    }
}
```

```graphql
mutation {
  createOrder(input: { amount: 150.00, customerId: 1 }) {
    id
    status
  }
}
```

👉 Just like queries, a mutation also defines exactly which fields to return after the write — no need for a second `GET` to fetch the freshly created resource, as often happens in REST

---

# 🔍 A strongly-typed schema

```graphql
type Order {
  id: Int!
  amount: Decimal!
  status: String!
  customer: Customer!
}
```

👉 Remember the OpenAPI/Swagger post (49)? GraphQL requires a strongly-typed schema from the start — documentation and type validation aren't an add-on tacked on later, they're a structural part of GraphQL itself

---

# ⚠️ Common Mistakes

- Using GraphQL for simple APIs with few relationships, where REST would already solve it with less complexity  
- Not implementing DataLoader to avoid the N+1 problem in nested relationships (fetching each order's customer individually, instead of in a batch)  
- Exposing the entire internal database structure directly in the GraphQL schema, without an abstraction layer  
- Forgetting to apply query depth/complexity limits, allowing maliciously nested queries that overload the server  

---

# 📌 Conclusion

- GraphQL lets the client choose exactly which fields it needs, avoiding over-fetching  
- HotChocolate builds the schema from `Query` and `Mutation` classes, similar to REST controllers  
- Related fields are only resolved when actually requested  
- The strongly-typed schema is structurally part of GraphQL, not an add-on  

👉 With GraphQL, APIs gain query flexibility that traditional REST doesn't offer natively — the client decides, on every request, exactly what it needs

---

# 🔥 Next Step

Now that you know an alternative to REST, the next level is:

👉 **C# Fundamentals: SignalR**

Here you'll learn real-time communication between server and client, beyond the traditional request/response model.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
