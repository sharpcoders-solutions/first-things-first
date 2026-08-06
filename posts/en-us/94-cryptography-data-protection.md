# 🧠 C# Fundamentals: Cryptography and Data Protection

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- OAuth2 and OpenID Connect for federated authentication  
- OWASP Top 10 (post 47), including sensitive data exposure as one of the most critical vulnerabilities  

Authentication guarantees you know **who** is accessing. But the data itself — passwords, tax IDs, card numbers — needs its own protection, both stored in the database and traveling across the network.

👉 **Let's learn Cryptography and Data Protection**

---

# 💡 Hashing vs Encryption: the distinction that avoids serious bugs

```csharp
// ❌ Never "encrypt" passwords — that implies they can be decrypted
var encryptedPassword = Encrypt(password);

// ✅ Passwords are "hashed" — a one-way, irreversible process
var passwordHash = HashPassword(password);
```

👉 Encryption is reversible (with the right key). Hashing isn't — if someone accesses your database, a correctly hashed password is useless to them, even with full database access

---

# 🏗️ Hashing passwords the right way

```csharp
public class PasswordService
{
    public string GenerateHash(string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password, workFactor: 12);
    }

    public bool Verify(string password, string storedHash)
    {
        return BCrypt.Net.BCrypt.Verify(password, storedHash);
    }
}
```

```csharp
// ❌ NEVER do this
var hash = SHA256.HashData(Encoding.UTF8.GetBytes(password)); // too fast, vulnerable to brute force
```

👉 BCrypt (or Argon2) is **deliberately slow** — that's a feature, not a bug, because it makes brute-force attacks harder. SHA256 is too fast and shouldn't be used for passwords, even though it's a "secure" hash algorithm for other purposes

---

# 🔐 Encrypting sensitive data at rest

```csharp
public class EncryptionService
{
    private readonly byte[] _key;

    public string Encrypt(string plainText)
    {
        using var aes = Aes.Create();
        aes.Key = _key;
        aes.GenerateIV();

        using var encryptor = aes.CreateEncryptor();
        var textBytes = Encoding.UTF8.GetBytes(plainText);
        var encryptedBytes = encryptor.TransformFinalBlock(textBytes, 0, textBytes.Length);

        return Convert.ToBase64String(aes.IV.Concat(encryptedBytes).ToArray());
    }
}
```

👉 Data like a card number or tax ID (when you need it back, unlike a password) uses AES — symmetric, reversible with the right key, and the key should never live in the same place as the encrypted data

---

# 🔑 Where to store keys: never in code

```csharp
// ❌ Never do this
private readonly byte[] _key = Convert.FromBase64String("MySecretKey123==");

// ✅ Use a secrets vault
var key = await _keyVaultClient.GetSecretAsync("orders-encryption-key");
```

👉 Remember the configuration post (76)? Encryption keys follow the same secrets principle we discussed there, but with even more rigor — Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault manage rotation and access, something an `appsettings.json` should never do

---

# 🌐 Data in transit: HTTPS isn't optional

```csharp
// Program.cs
app.UseHttpsRedirection();
app.UseHsts(); // forces HTTPS on future browser requests
```

👉 Even with data encrypted at rest, an HTTP request without TLS exposes everything in plain text as it travels across the network — HTTPS protects "in transit" the same way AES protects "at rest"

---

# 🛡️ ASP.NET Core Data Protection API

```csharp
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"/shared-keys"))
    .SetApplicationName("MyCompany.Orders");
```

```csharp
public class SecureTokenService
{
    private readonly IDataProtector _protector;

    public SecureTokenService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("PasswordResetTokens");
    }

    public string Protect(string data) => _protector.Protect(data);
    public string Unprotect(string protectedData) => _protector.Unprotect(protectedData);
}
```

👉 ASP.NET Core already ships with a managed encryption API — for common cases (temporary tokens, cookies), it handles key rotation and implementation details you wouldn't need to reinvent manually

---

# ⚠️ Common Mistakes

- Using the same algorithm (hash) for passwords and for data that needs to be recovered — these are different problems, with different solutions  
- Storing encryption keys in version control, even "temporarily"  
- Relying only on HTTPS and forgetting to also encrypt sensitive data at rest, in the database  
- Implementing your own encryption algorithm instead of using tested, reviewed libraries (`System.Security.Cryptography`)  

---

# 📌 Conclusion

- Hashing (passwords) and encryption (recoverable data) solve different problems, with different tools  
- BCrypt/Argon2 are deliberately slow to make brute force harder; never use a fast hash for passwords  
- Encryption keys live in secrets vaults, never in code or version control  
- HTTPS protects data in transit; encryption protects data at rest — both are necessary  

👉 With cryptography applied correctly, even a database leak doesn't directly expose your users' most sensitive information

---

# 🔥 Next Step

Now that you protect sensitive data correctly, the next level is:

👉 **C# Fundamentals: Clean Code**

Here you'll consolidate clean code principles that have run through this entire track, since the very first post.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
