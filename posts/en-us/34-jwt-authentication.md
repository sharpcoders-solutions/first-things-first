# 🧠 C# Fundamentals: Authentication and Authorization with JWT

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to build and organize an API with Clean Architecture  
- How to persist real data with EF Core  

Your API is functional and well-structured, but right now it accepts **any request from anyone**. Time to close that door.

👉 **Let's secure the API with authentication and authorization using JWT**

---

# 💡 Authentication vs Authorization: the distinction everyone mixes up

👉 **Authentication = who you are. Authorization = what you're allowed to do**

- **Authentication**: the user proves their identity (logging in with a username and password)  
- **Authorization**: once identified, the system decides whether they have permission for that specific action  

👉 A user can be **authenticated** (logged in) and still **not authorized** to access an admin endpoint

---

# 🔑 What is JWT?

👉 **JWT (JSON Web Token) = a compact, self-contained token that carries verifiable information about the user**

A JWT has three parts, separated by dots:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMiLCJyb2xlIjoiQWRtaW4ifQ.4f8a...
   Header              Payload                              Signature
```

- **Header** → the signing algorithm used  
- **Payload** → the **claims** (data about the user: id, name, role)  
- **Signature** → guarantees the token wasn't tampered with after it was issued  

👉 The server doesn't need to query the database on every request to know who the user is — the information already lives inside the token itself, signed in a way that any tampering is detectable

---

# 🏗️ Setting up JWT authentication

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

```csharp
// Program.cs
var key = builder.Configuration["Jwt:Key"];

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(key))
        };
    });

builder.Services.AddAuthorization();

// ...

app.UseAuthentication(); // always before UseAuthorization
app.UseAuthorization();
```

👉 `UseAuthentication` identifies **who** is making the request; `UseAuthorization` decides **whether** that person can access the resource — the order between the two matters

---

# ✍️ Generating a token on login

```csharp
[HttpPost("login")]
public IActionResult Login(LoginRequest request)
{
    // username and password validation (simplified)
    if (request.Username != "admin" || request.Password != "123456")
        return Unauthorized();

    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, request.Username),
        new Claim(ClaimTypes.Role, "Admin")
    };

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]));
    var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

    var token = new JwtSecurityToken(
        issuer: _configuration["Jwt:Issuer"],
        audience: _configuration["Jwt:Audience"],
        claims: claims,
        expires: DateTime.UtcNow.AddHours(2),
        signingCredentials: credentials
    );

    return Ok(new { token = new JwtSecurityTokenHandler().WriteToken(token) });
}
```

👉 `Claim` is the basic unit of information inside the token — here we carry the user's name and their role (`Role`), which will later be read for authorization decisions

---

# 🔒 Protecting endpoints with `[Authorize]`

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [Authorize] // requires any authenticated user
    public IActionResult GetAll() => Ok(/* ... */);

    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")] // requires an authenticated user AND the Admin role
    public IActionResult Delete(int id) => Ok(/* ... */);
}
```

👉 Without a valid token in the `Authorization: Bearer {token}` header, the request gets a `401 Unauthorized`. With a valid token but missing the required role, it gets a `403 Forbidden` — two different HTTP codes for two different problems

---

# 👤 Reading the authenticated user's claims

```csharp
[HttpGet("my-profile")]
[Authorize]
public IActionResult MyProfile()
{
    string name = User.Identity.Name;
    string role = User.FindFirst(ClaimTypes.Role)?.Value;

    return Ok(new { name, role });
}
```

👉 Once the authentication middleware validates the token, ASP.NET Core automatically populates `User` with the claims — you never need to decode the token manually

---

# ⚠️ Common Mistakes

- Storing the signing key (`Jwt:Key`) directly in source code instead of secure configuration (environment variables, a secrets manager)  
- Forgetting `ValidateLifetime`, allowing expired tokens to keep being accepted  
- Confusing `401` (not authenticated) with `403` (authenticated, but not permitted) while debugging access problems  
- Putting sensitive data in the JWT payload — it's **signed**, not **encrypted**, so anyone can read the content, they just can't alter it without invalidating the signature  

---

# 📌 Conclusion

- Authentication proves identity; authorization decides permissions  
- JWT carries verifiable claims, without requiring a database lookup on every request  
- `[Authorize]` protects endpoints; `[Authorize(Roles = "...")]` adds an extra layer of control  
- `401` means "who are you?"; `403` means "I know who you are, but you can't"  

👉 With authentication and authorization, your API stops trusting whoever's on the other end of the request and starts **verifying**

---

# 🔥 Next Step

Now that your API is properly secured, the next level is:

👉 **C# Fundamentals: Docker and Deploying a .NET API**

Here you'll learn to package and publish your application so it runs consistently in any environment.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
