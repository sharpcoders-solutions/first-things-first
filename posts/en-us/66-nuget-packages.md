# 🧠 C# Fundamentals: Creating and Publishing NuGet Packages

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Roslyn Analyzers for distributing custom rules  
- You've already used dozens of NuGet packages throughout this track (Serilog, Hangfire, xUnit, Polly...)  

Every time you ran `dotnet add package`, someone packaged and published that code for the world to use. Now it's your turn to do the same with your own code.

👉 **Let's learn to create and publish NuGet packages**

---

# 💡 What is a NuGet package?

👉 **NuGet = .NET's package manager — a package is a compiled DLL, bundled with metadata, versioning, and dependencies**

Every package you've used throughout this track follows the same format — from `Microsoft.EntityFrameworkCore` to `Hangfire.AspNetCore`

---

# 🏗️ Creating a shareable library

```bash
dotnet new classlib -n MyCompany.Utils
cd MyCompany.Utils
```

```csharp
namespace MyCompany.Utils;

public static class StringExtensions
{
    public static string ToSlug(this string value)
    {
        return value.ToLowerInvariant()
            .Replace(" ", "-")
            .Normalize(NormalizationForm.FormD);
    }
}
```

👉 Nothing special so far — it's a regular class, written the same way you've been writing them since the methods post (post 17). What changes is how it's packaged and distributed

---

# 📦 Configuring `.csproj` for packaging

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <PackageId>MyCompany.Utils</PackageId>
    <Version>1.0.0</Version>
    <Authors>Vitor Santos</Authors>
    <Description>MyCompany's shared utility extensions</Description>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
  </PropertyGroup>
</Project>
```

```bash
dotnet pack -c Release
```

👉 This generates a `.nupkg` file — the same file format you automatically download every time you run `dotnet add package`

---

# 🚀 Publishing to NuGet.org

```bash
dotnet nuget push bin/Release/MyCompany.Utils.1.0.0.nupkg \
  --api-key YOUR_API_KEY \
  --source https://api.nuget.org/v3/index.json
```

👉 Once published, anyone in the world can install it with `dotnet add package MyCompany.Utils` — the same command you used for Serilog, Polly, and everything else

---

# 🏢 Private feeds for internal packages

```bash
dotnet nuget add source https://pkgs.dev.azure.com/mycompany/_packaging/internal/nuget/v3/index.json \
  --name "internal-feed"

dotnet nuget push MyCompany.Utils.1.0.0.nupkg --source "internal-feed"
```

👉 Not every package should go to public NuGet.org — internal company code goes to a private feed (Azure Artifacts, GitHub Packages), with the same mechanics, but access restricted to the team

---

# 🔢 Semantic versioning

```
1.0.0 → 1.0.1  (patch: bug fix, nothing breaks)
1.0.1 → 1.1.0  (minor: new feature, compatible)
1.1.0 → 2.0.0  (major: breaking change)
```

👉 Every dependency you added throughout this track follows this convention — following the same pattern prevents a "small" update from breaking the code of whoever consumes your package

---

# ⚠️ Common Mistakes

- Publishing a package without tests (remember post 33?), letting bugs go straight to consumers  
- Not following semantic versioning, breaking compatibility in a "patch" version  
- Forgetting to document the package (README, XML doc comments), leaving consumers without a clue how it works  
- Accidentally publishing secrets or API keys inside the package  

---

# 📌 Conclusion

- NuGet packages compiled C# code, with metadata and dependencies  
- `dotnet pack` generates the `.nupkg`; `dotnet nuget push` publishes it  
- Private feeds keep internal code out of public NuGet.org  
- Semantic versioning communicates the impact of every update  

👉 With NuGet packages, your code stops living isolated in one project and becomes reusable across the whole organization — or the entire world

---

# 🔥 Next Step

Now that you know how to package and share code, the next level is:

👉 **C# Fundamentals: Lock, Monitor, and Synchronization**

Here you'll learn to protect critical sections of code when multiple threads access the same state at the same time.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
