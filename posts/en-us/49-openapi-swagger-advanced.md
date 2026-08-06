# 🧠 C# Fundamentals: Documenting APIs with Advanced OpenAPI/Swagger

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- API versioning  
- Security following the OWASP Top 10  

Since the ASP.NET Core post, you've been using the basic, automatically generated Swagger. But for an API with multiple versions and protected endpoints, the default documentation isn't enough anymore.

👉 **Let's get your documentation truly ready for whoever consumes your API from the outside**

---

# 💡 What is OpenAPI?

👉 **OpenAPI = the specification that formally describes a REST API. Swagger = the set of tools that reads that specification and generates the interactive interface**

All the configuration you've already seen (`[HttpGet]`, `[FromBody]`, `[ApiVersion]`) already feeds this specification automatically — now let's enrich it

---

# 🏗️ Enriching the basic configuration

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "API for managing products and orders",
        Contact = new OpenApiContact { Name = "Vitor Santos", Email = "contact@company.com" }
    });

    options.SwaggerDoc("v2", new OpenApiInfo { Title = "My API", Version = "v2" });
});
```

👉 Each version registered here generates **separate** documentation, connecting directly with the versioning post — whoever consumes v1 only sees the v1 endpoints

---

# 📝 Documenting endpoints with XML comments

```bash
dotnet add package Swashbuckle.AspNetCore.Annotations
```

```csharp
/// <summary>
/// Creates a new product in the catalog.
/// </summary>
/// <param name="request">The name and price of the product to create.</param>
/// <returns>The newly created product, including its Id.</returns>
/// <response code="201">Product created successfully.</response>
/// <response code="400">Invalid input data.</response>
[HttpPost]
[ProducesResponseType(typeof(ProductResponse), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public IActionResult Create(CreateProductRequest request)
{
    // ...
}
```

```xml
<!-- In the .csproj -->
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

👉 `[ProducesResponseType]` explicitly documents **every** possible HTTP status code — remember the proper HTTP responses post? Now Swagger shows this to whoever consumes the API, without needing to read the source code

---

# 🔒 Documenting authentication in Swagger

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        Description = "Enter the JWT token in the format: Bearer {your token}"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" }
            },
            Array.Empty<string>()
        }
    });
});
```

👉 This adds an **"Authorize"** button to the Swagger interface — whoever is testing the API can paste the JWT token (from the authentication post) once and test every protected endpoint without repeating the header manually

---

# 📋 Request/response examples

```csharp
public record CreateProductRequest(string Name, decimal Price)
{
    /// <example>Dell Laptop</example>
    public string Name { get; init; } = Name;

    /// <example>3500.00</example>
    public decimal Price { get; init; } = Price;
}
```

👉 Concrete examples in the request body remove ambiguity — instead of guessing the expected format, whoever consumes the API sees exactly how to fill in each field

---

# 🗂️ Organizing with Tags

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[Tags("Products")]
public class ProductsController : ControllerBase { }

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[Tags("Orders")]
public class OrdersController : ControllerBase { }
```

👉 In APIs with many controllers, tags group endpoints in the Swagger interface logically — the difference between browsable documentation and a confusing list of fifty loose endpoints

---

# ⚠️ Common Mistakes

- Leaving the generic auto-generated documentation as is, with no real descriptions of what each endpoint does  
- Forgetting to document the possible error codes, leaving consumers to find out the hard way  
- Not keeping documentation in sync across versions, confusing whoever still uses v1  
- Publicly exposing Swagger in production with no protection, revealing the API's entire structure to anyone  

---

# 📌 Conclusion

- OpenAPI is the specification; Swagger is the tool that makes it interactive  
- `[ProducesResponseType]` explicitly documents every possible HTTP response  
- Configuring authentication in Swagger lets you test protected endpoints right in the interface  
- Tags organize large APIs into browsable groups  

👉 A well-documented API is just as important as a well-built one — it's what lets other teams (and future you) use it without needing to read the entire source code

---

# 🔥 Next Step

You've reached the end of this track's technical journey — from your first `Console.WriteLine` to a complete, secure, documented, production-ready API. The next (and last) step is about you:

👉 **C# Fundamentals: Career — Preparing for C#/.NET Interviews**

Here you'll learn to turn everything you've built throughout this track into a real career.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
