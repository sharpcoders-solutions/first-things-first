# 🧠 C# Fundamentals: API Versioning

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Advanced security following the OWASP Top 10  
- How to structure API contracts with DTOs and records  

Your API is live, being consumed by other systems. One day you'll need to change a contract — and breaking whoever already depends on it isn't an option.

👉 **Let's learn how to evolve an API without breaking its consumers**

---

# 💡 Why version?

👉 **Versioning = allowing multiple versions of the same API to coexist, so clients can migrate at their own pace**

```csharp
public record ProductResponseV1(int Id, string Name, decimal Price);

public record ProductResponseV2(int Id, string Name, decimal Price, string Category, bool InStock);
```

👉 Without versioning, adding `Category` as a required field can break a client that wasn't expecting it — or worse, removing a field another system still uses takes down entire integrations without warning

---

# 🏗️ Versioning strategies

```bash
dotnet add package Asp.Versioning.Mvc
```

## 🔹 URL-based versioning (the most common and explicit)

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetAllV1() => Ok(/* ... */);

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetAllV2() => Ok(/* ... */);
}
```

```
GET /api/v1/products
GET /api/v2/products
```

👉 Easy to understand and test directly in the browser — the version is visible right in the URL

## 🔹 Header-based versioning

```csharp
[HttpGet]
public IActionResult GetAll([FromHeader(Name = "X-Api-Version")] string version)
{
    return version switch
    {
        "2.0" => Ok(_service.GetAllV2()),
        _ => Ok(_service.GetAllV1())
    };
}
```

```
GET /api/products
X-Api-Version: 2.0
```

👉 Keeps the URL "clean," but requires the client to know to add the header — less discoverable, more common in internal APIs between microservices

## 🔹 Query string versioning

```
GET /api/products?api-version=2.0
```

👉 Easy to test, but clutters the URL with a parameter that isn't really about the resource

---

# 📦 Registering versioning in ASP.NET Core

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});
```

👉 `AssumeDefaultVersionWhenUnspecified` guarantees old clients, who don't even know versioning exists, keep working on v1 without needing to change anything

---

# 🔀 Changes that need a new version vs safe changes

## 🔹 Safe, no new version needed

- Adding a **new endpoint**  
- Adding an **optional** field to the response (most clients ignore unknown fields)  

## 🔹 Requires a new version

- Removing or renaming an existing field  
- Changing a field's type (`string` to `int`, for example)  
- Changing an existing endpoint's behavior in an incompatible way  

👉 The general rule: if a client already integrated with you **would break** without changing anything on their end, it's a change that needs a new version

---

# 🗑️ Deprecating old versions responsibly

```csharp
[ApiVersion("1.0", Deprecated = true)]
public class ProductsController : ControllerBase
{
    // ...
}
```

```
Response Header: api-supported-versions: 1.0, 2.0
Response Header: api-deprecated-versions: 1.0
```

👉 Marking something as `Deprecated` doesn't remove the version — it just signals to consumers, through response headers, that they should migrate. This gives time for a planned transition, instead of a sudden break

---

# ⚠️ Common Mistakes

- Making incompatible changes to an existing version instead of creating a new one  
- Never deprecating or removing old versions, piling up infinite maintenance  
- Versioning too early, creating unnecessary complexity for an API that doesn't have external consumers yet  
- Poorly documenting the differences between versions, leaving consumers to guess what changed  

---

# 📌 Conclusion

- Versioning lets you evolve the API without breaking whoever already depends on it  
- URL, header, and query string are the three most common strategies — URL is the most explicit  
- Adding optional fields is generally safe; removing or renaming fields requires a new version  
- Deprecating (not abruptly removing) gives consumers time to migrate  

👉 An API that versions well can keep evolving continuously, without ever becoming hostage to the clients that already depend on it

---

# 🔥 Next Step

Now that you know how to evolve your API safely, the next level is:

👉 **C# Fundamentals: Documenting APIs with Advanced OpenAPI/Swagger**

Here you'll learn to document every version and every detail of your API in a way that genuinely helps whoever consumes it.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
