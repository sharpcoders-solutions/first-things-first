# 🧠 C# Fundamentals: Advanced API Security (OWASP Top 10 in Practice)

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Authentication and authorization with JWT  
- Performance and code optimization  

You already know **who** can access each endpoint. But authentication doesn't protect against everything — there are specific vulnerabilities that attack APIs even when authentication is working perfectly.

👉 **Let's walk through the OWASP Top 10 as it applies to a real C# API**

---

# 💡 What is the OWASP Top 10?

👉 **A list, maintained by the OWASP community, of the most critical and most common vulnerabilities in web applications**

It's not a theoretical list — it's based on real security incident data. Let's go through the ones that most affect .NET APIs day to day.

---

# 💉 1. Injection (SQL Injection)

```csharp
// ❌ Vulnerable: concatenating user input directly into the query
var sql = $"SELECT * FROM Products WHERE Name = '{userTypedName}'";
```

```csharp
// ✅ Safe: EF Core parameterizes automatically
var products = await _context.Products
    .Where(p => p.Name == userTypedName)
    .ToListAsync();
```

👉 Remember EF Core? Using LINQ instead of manually concatenated SQL **already eliminates** most of the SQL Injection risk — EF Core always parameterizes queries under the hood

---

# 🔓 2. Broken Authentication

```csharp
// ❌ Session never expires, no minimum password policy
services.AddAuthentication().AddCookie(options =>
{
    options.ExpireTimeSpan = TimeSpan.FromDays(365); // way too long
});
```

```csharp
// ✅ Short expiration + renewal, strong password policy
builder.Services.Configure<IdentityOptions>(options =>
{
    options.Password.RequiredLength = 12;
    options.Password.RequireNonAlphanumeric = true;
    options.Lockout.MaxFailedAccessAttempts = 5;
});
```

👉 Remember the JWT post? Short-lived tokens, with `ValidateLifetime = true`, shrink the exposure window if a token leaks

---

# 📤 3. Sensitive Data Exposure

```csharp
// ❌ Returning the entire domain entity, including the password hash
return Ok(user);
```

```csharp
// ✅ An explicit DTO controls exactly what goes out
public record UserResponse(int Id, string Name, string Email);

return Ok(new UserResponse(user.Id, user.Name, user.Email));
```

👉 This is exactly the common mistake you saw in the ASP.NET Core post: exposing the domain entity directly in the response. A response `record` guarantees sensitive fields never leak by accident

---

# 🌐 4. Misconfigured CORS

```csharp
// ❌ Allows any origin, with credentials
app.UseCors(policy => policy.AllowAnyOrigin().AllowCredentials());
```

```csharp
// ✅ Explicit list of trusted origins
app.UseCors(policy => policy
    .WithOrigins("https://myapp.com")
    .AllowCredentials()
    .WithMethods("GET", "POST"));
```

👉 `AllowAnyOrigin` combined with `AllowCredentials` is a dangerous combination — in practice, the browser doesn't even allow this combination, but poorly thought-out CORS configurations remain a common cause of vulnerabilities

---

# ⚠️ 5. Lack of Input Validation

```csharp
public record CreateProductRequest(string Name, decimal Price);

public class CreateProductValidator : AbstractValidator<CreateProductRequest>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(100);
        RuleFor(x => x.Price).GreaterThan(0);
    }
}
```

👉 Using the **FluentValidation** library, you guarantee invalid or malicious data never reaches your entity's business rules — the same protection you saw in the classes and objects post, now applied at the API's edge, before data even enters the domain

---

# 🔑 6. Broken Access Control

```csharp
// ❌ Only checks if authenticated, not if they own the resource
[HttpGet("{orderId}")]
[Authorize]
public IActionResult GetOrder(int orderId)
{
    return Ok(_repository.GetById(orderId)); // any logged-in user can see anyone else's order
}
```

```csharp
// ✅ Checks whether the order belongs to the authenticated user
[HttpGet("{orderId}")]
[Authorize]
public IActionResult GetOrder(int orderId)
{
    var order = _repository.GetById(orderId);
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

    if (order.CustomerId.ToString() != userId)
        return Forbid();

    return Ok(order);
}
```

👉 `[Authorize]` alone only checks **authentication**. Checking whether the user has permission over **that specific resource** is an additional responsibility the code needs to implement explicitly — this is probably the most common and most exploited mistake in real APIs

---

# 🧾 7. Insufficient Logging and Monitoring

```csharp
_logger.LogWarning("Unauthorized access attempt to order {OrderId} by {UserId}", orderId, userId);
```

👉 Remember the Serilog post? Logging denied access attempts, authentication failures, and sensitive actions is what lets you **detect** an attack in progress — without it, a breach can go unnoticed for months

---

# ⚠️ Common Mistakes

- Blindly trusting data coming from the client, even after authentication  
- Returning overly detailed error messages (full stack traces) in production, revealing implementation details  
- Forgetting HTTPS in any environment, even internal ones  
- Validating only on the front-end, without repeating validation on the back-end (the front-end is optional for an attacker)  

---

# 📌 Conclusion

- LINQ/EF Core already protects against most SQL Injection  
- Response DTOs prevent sensitive data from leaking by accident  
- CORS should list explicit origins, never allow everything with credentials  
- Input validation (FluentValidation) blocks invalid data before it reaches the domain  
- `[Authorize]` checks authentication; checking resource ownership is the code's explicit responsibility  

👉 Security isn't a final step — it's a layer that runs through every decision you've made throughout this entire track, from validation to logging

---

# 🔥 Next Step

Now that your API is protected against the most common vulnerabilities, the next level is:

👉 **C# Fundamentals: API Versioning**

Here you'll learn to evolve your API without breaking the clients that already depend on it.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
