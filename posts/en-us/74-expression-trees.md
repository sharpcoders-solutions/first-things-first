# 🧠 C# Fundamentals: Expression Trees

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Covariance and contravariance in generics  
- LINQ (post 19) and Entity Framework Core (post 32)  

Ever wondered how Entity Framework turns `.Where(o => o.Amount > 100)` into a `WHERE Amount > 100` clause in SQL? A regular lambda just becomes compiled code — Expression Trees let code become **data** that can be inspected and translated.

👉 **Let's learn Expression Trees**

---

# 💡 Func vs Expression: the crucial difference

```csharp
Func<Order, bool> asFunction = o => o.Amount > 100;
bool result = asFunction(order); // executes the code immediately

Expression<Func<Order, bool>> asExpression = o => o.Amount > 100;
// this does NOT execute anything — it's the REPRESENTATION of the logic, like a data tree
```

👉 `Func<>` compiles the lambda into executable IL. `Expression<Func<>>` compiles the lambda into a **syntax tree** that describes the logic — the same kind of structure you saw in the Roslyn Analyzers post (62), but for regular C# expressions

---

# 🏗️ Inspecting an Expression Tree

```csharp
Expression<Func<Order, bool>> expression = o => o.Amount > 100;

var body = (BinaryExpression)expression.Body;
var property = (MemberExpression)body.Left;
var value = (ConstantExpression)body.Right;

Console.WriteLine(body.NodeType);       // GreaterThan
Console.WriteLine(property.Member.Name); // "Amount"
Console.WriteLine(value.Value);           // 100
```

👉 Instead of executing `o.Amount > 100`, you're **reading the structure** of that comparison — which property, which operator, which value. That's exactly what EF Core does to generate SQL

---

# 🎯 How Entity Framework actually uses this

```csharp
var expensiveOrders = _context.Orders
    .Where(o => o.Amount > 100)  // this is an Expression<Func<Order, bool>>
    .ToList();
```

```sql
-- EF Core translates the expression tree into real SQL
SELECT * FROM Orders WHERE Amount > 100
```

👉 Remember post 32? EF Core's `DbSet<T>` implements `IQueryable<T>`, which uses `Expression<Func<>>` instead of plain `Func<>` — that's why EF can "read" your LINQ and translate it to SQL, instead of downloading all the data and filtering in memory

---

# 🔨 Building Expression Trees dynamically

```csharp
ParameterExpression parameter = Expression.Parameter(typeof(Order), "o");
MemberExpression property = Expression.Property(parameter, "Amount");
ConstantExpression constant = Expression.Constant(100m);
BinaryExpression comparison = Expression.GreaterThan(property, constant);

var lambda = Expression.Lambda<Func<Order, bool>>(comparison, parameter);
var filter = lambda.Compile(); // now becomes an executable Func<>

bool result = filter(order); // true, if order.Amount > 100
```

👉 This lets you build filters dynamically at runtime — useful for advanced search systems, where the user picks which fields and operators to filter by, without you writing an `if` for every possible combination

---

# ⚠️ Common Mistakes

- Using `Expression<Func<>>` when only `Func<>` is needed, adding complexity for no reason (Expression Trees only make sense when something needs to **read** the logic, like EF Core)  
- Calling arbitrary C# methods inside an expression that will be translated to SQL, causing a runtime error because the provider doesn't know how to translate that method  
- Manually building expression trees when a simple lambda would solve the problem  
- Forgetting that compiling an Expression Tree (`.Compile()`) has a cost — cache the result when used repeatedly  

---

# 📌 Conclusion

- `Expression<Func<>>` represents code as an inspectable data tree, instead of directly executable code  
- This is the foundation of how Entity Framework translates LINQ into SQL  
- Expression Trees can be built dynamically to generate logic at runtime  
- The complexity cost is only worth it when something needs to **read**, not just execute, your logic  

👉 With Expression Trees, you understand the magic behind LINQ to SQL — C# code that becomes database queries, without a single line of SQL written by hand

---

# 🔥 Next Step

Now that you understand how code becomes data, the next level is:

👉 **C# Fundamentals: Nullable Reference Types in Depth**

Here you'll go deeper into the type system you've used since the start of this track, understanding its more advanced nuances.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
