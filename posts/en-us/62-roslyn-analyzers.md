# 🧠 C# Fundamentals: Roslyn Analyzers

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Mutation Testing for measuring real test quality  
- Consistent code rules throughout this track (nullable, async, SOLID)  

How do you guarantee the whole team follows the same rules, without relying on everyone remembering them in every Pull Request? The C# compiler (Roslyn) lets you write your own rules, which run automatically on every build.

👉 **Let's learn to create Roslyn Analyzers**

---

# 💡 What is Roslyn?

👉 **Roslyn = the open-source C# compiler, which exposes its own syntax tree (AST) for you to inspect and even generate code from**

You already use analyzers every day without noticing:

```csharp
public async void ProcessOrder() // ⚠️ CS-something: avoid async void
```

That warning doesn't come from rules Microsoft hand-wrote for your project — it comes from a built-in SDK analyzer, examining your code's syntax tree

---

# 🏗️ Creating a simple Analyzer

```csharp
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public class ForbidConsoleWriteLineAnalyzer : DiagnosticAnalyzer
{
    private static readonly DiagnosticDescriptor Rule = new(
        id: "MY001",
        title: "Don't use Console.WriteLine in production code",
        messageFormat: "Use ILogger instead of Console.WriteLine",
        category: "Design",
        DiagnosticSeverity.Warning,
        isEnabledByDefault: true);

    public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics
        => ImmutableArray.Create(Rule);

    public override void Initialize(AnalysisContext context)
    {
        context.RegisterSyntaxNodeAction(Analyze, SyntaxKind.InvocationExpression);
    }

    private void Analyze(SyntaxNodeAnalysisContext context)
    {
        var invocation = (InvocationExpressionSyntax)context.Node;

        if (invocation.Expression.ToString() == "Console.WriteLine")
        {
            var diagnostic = Diagnostic.Create(Rule, invocation.GetLocation());
            context.ReportDiagnostic(diagnostic);
        }
    }
}
```

👉 `RegisterSyntaxNodeAction` registers a callback that runs every time the compiler finds that kind of node in the syntax tree — in this case, any method invocation

---

# 🎯 The result in the editor

```csharp
public void ProcessOrder()
{
    Console.WriteLine("Processing..."); // ⚠️ MY001: Use ILogger instead of Console.WriteLine
}
```

👉 Combined with Serilog (post 37), this analyzer ensures no one accidentally goes back to `Console.WriteLine` — the warning shows up right in the editor, before code review even happens

---

# 🔧 Code Fix: fixing it automatically

```csharp
[ExportCodeFixProvider(LanguageNames.CSharp)]
public class UseLoggerCodeFix : CodeFixProvider
{
    public override async Task RegisterCodeFixesAsync(CodeFixContext context)
    {
        var action = CodeAction.Create(
            title: "Replace with _logger.LogInformation",
            createChangedDocument: c => ReplaceWithLogger(context.Document, c));

        context.RegisterCodeFix(action, context.Diagnostics.First());
    }
}
```

👉 Besides warning, you can offer a one-click automatic fix (`Ctrl + .` in Visual Studio) — the same mechanism that already suggests "add using" or "make method async"

---

# 📦 Distributing the Analyzer to the team

```xml
<!-- Directory.Build.props at the solution root -->
<Project>
  <ItemGroup>
    <PackageReference Include="MyCompany.Analyzers" Version="1.0.0" PrivateAssets="all" />
  </ItemGroup>
</Project>
```

👉 Packaged as NuGet (more on that in the next post) and referenced in `Directory.Build.props`, the analyzer runs automatically across the whole solution, for every developer — with no reliance on manual setup on each machine

---

# ⚠️ Common Mistakes

- Creating analyzers for everything, making the build slow and the development experience annoying  
- Setting `Error` severity for stylistic rules, blocking builds over preferences instead of real bugs  
- Not writing tests for the analyzer itself, risking mass false positives  
- Ignoring analyzers that already exist in the community before writing one from scratch — a lot already exists ready-made  

---

# 📌 Conclusion

- Roslyn exposes C#'s syntax tree, letting you write custom analysis rules  
- `DiagnosticAnalyzer` detects problematic patterns and generates warnings right in the editor  
- `CodeFixProvider` offers one-click automatic fixes  
- Packaged as NuGet, the analyzer guarantees consistency for the whole team, without relying on individual memory  

👉 With Roslyn Analyzers, your team's rules stop living only in documentation and start being enforced automatically by the compiler

---

# 🔥 Next Step

Now that you can create and distribute custom rules, the next level is:

👉 **C# Fundamentals: Creating and Publishing NuGet Packages**

Here you'll learn to package and share your own code as a reusable library.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
