# 🧠 C# Fundamentals: Identity Server and OAuth2

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Full observability, consolidating metrics, traces, and logs  
- JWT (post 34) — you already issue and validate tokens, but always within your own API  

In post 34, your own API created and validated the JWT. But what about when you need users to log in with Google, or multiple applications to share a single identity provider? That's OAuth2 and OpenID Connect.

👉 **Let's go deeper into Identity Server and OAuth2**

---

# 💡 Revisiting the JWT from post 34, with more context

```csharp
// Post 34: your own API issued the token
var token = new JwtSecurityToken(
    issuer: "myapi.com",
    claims: claims,
    expires: DateTime.UtcNow.AddHours(1));
```

👉 This works well when only one API exists. But if you have 5 different applications, each reimplementing login, or want to allow "Sign in with Google" — you need a centralized identity provider

---

# 🏗️ OAuth2: roles, not implementation

👉 **OAuth2 is an authorization protocol — it defines roles and flows, not a specific library**

```
Resource Owner: the user
Client: the application that wants to access data on the user's behalf
Authorization Server: issues tokens (e.g., Identity Server, Auth0, Azure AD)
Resource Server: your API, which validates the token (what you've already done since post 34)
```

---

# 🎯 The Authorization Code flow (the most common)

```
1. User clicks "Sign in" on your app
2. App redirects to the Authorization Server
3. User logs in there (not on your application)
4. Authorization Server redirects back with a "code"
5. Your application exchanges that code for an Access Token + Refresh Token
6. Your API validates the Access Token — the same mechanism from post 34
```

```csharp
// Program.cs — configuring your API as a Resource Server
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://my-identity-server.com";
        options.Audience = "my-api";
    });
```

👉 The key difference: your API no longer issues the token, it just **trusts** an external issuer — the `[Authorize]` you've used since post 34 keeps working exactly the same

---

# 🔐 OpenID Connect: identity on top of OAuth2

```
OAuth2 alone: "this client is allowed to access this resource"
OpenID Connect (OIDC): OAuth2 + "and this is the user: name, email, photo"
```

```csharp
var claims = User.Claims;
var name = claims.First(c => c.Type == "name").Value;
var email = claims.First(c => c.Type == "email").Value;
```

👉 Plain OAuth2 solves authorization; OIDC adds identity — the `id_token` (different from the `access_token`) carries these claims about who the user is, in the same JWT format you already know

---

# 🏢 Two .NET options: Identity Server vs managed providers

## 🔹 Duende IdentityServer (self-hosted)

```csharp
builder.Services.AddIdentityServer()
    .AddInMemoryClients(Config.Clients)
    .AddInMemoryApiScopes(Config.ApiScopes)
    .AddDeveloperSigningCredential();
```

👉 You host and control the entire Authorization Server — maximum control, maximum operational responsibility

## 🔹 Managed providers (Azure AD, Auth0, Okta)

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://mycompany.b2clogin.com/...";
    });
```

👉 Outsources the complexity of identity security — usually the right choice, unless you have very specific customization requirements

---

# ⚠️ Common Mistakes

- Implementing your own Authorization Server from scratch without a real need, when a managed provider would solve it with much less risk  
- Confusing `access_token` with `id_token` — one authorizes resource access, the other carries identity  
- Not validating the token's `Audience`, accepting tokens issued for a different application  
- Storing tokens insecurely on the client (localStorage is vulnerable to XSS) instead of HttpOnly cookies  

---

# 📌 Conclusion

- OAuth2 defines authorization roles and flows; OpenID Connect adds identity on top of it  
- The Authorization Code flow is the most common pattern for web applications  
- Your API keeps validating tokens with `[Authorize]`, just now trusting an external issuer  
- Managed providers are usually the right choice over hosting your own Identity Server  

👉 With OAuth2 and OpenID Connect, authentication stops being reinvented in every application and becomes a centralized, trustworthy service

---

# 🔥 Next Step

Now that you've mastered federated authentication, the next level is:

👉 **C# Fundamentals: Cryptography and Data Protection**

Here you'll learn to protect sensitive data at rest and in transit, beyond authentication.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
