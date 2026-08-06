# 🧠 Fundamentos do C#: Roslyn Analyzers

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Mutation Testing para medir a qualidade real dos testes  
- Regras de código consistentes ao longo de toda a trilha (nullable, async, SOLID)  

Como garantir que toda a equipe siga as mesmas regras, sem depender de lembrar em cada Pull Request? O compilador do C# (Roslyn) permite escrever suas próprias regras, que rodam automaticamente a cada build.

👉 **Vamos aprender a criar Roslyn Analyzers**

---

# 💡 O que é o Roslyn?

👉 **Roslyn = o compilador open-source do C#, que expõe sua própria árvore sintática (AST) para você inspecionar e até gerar código**

Você já usa analyzers todos os dias sem perceber:

```csharp
public async void ProcessarPedido() // ⚠️ CS-alguma-coisa: evite async void
```

Esse aviso não vem de regras escritas à mão pela Microsoft para o seu projeto — vem de um analyzer padrão do SDK, examinando a árvore sintática do seu código

---

# 🏗️ Criando um Analyzer simples

```csharp
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public class ProibirConsoleWriteLineAnalyzer : DiagnosticAnalyzer
{
    private static readonly DiagnosticDescriptor Regra = new(
        id: "MEU001",
        title: "Não use Console.WriteLine em código de produção",
        messageFormat: "Use ILogger em vez de Console.WriteLine",
        category: "Design",
        DiagnosticSeverity.Warning,
        isEnabledByDefault: true);

    public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics
        => ImmutableArray.Create(Regra);

    public override void Initialize(AnalysisContext contexto)
    {
        contexto.RegisterSyntaxNodeAction(Analisar, SyntaxKind.InvocationExpression);
    }

    private void Analisar(SyntaxNodeAnalysisContext contexto)
    {
        var invocacao = (InvocationExpressionSyntax)contexto.Node;

        if (invocacao.Expression.ToString() == "Console.WriteLine")
        {
            var diagnostico = Diagnostic.Create(Regra, invocacao.GetLocation());
            contexto.ReportDiagnostic(diagnostico);
        }
    }
}
```

👉 `RegisterSyntaxNodeAction` registra um callback que roda toda vez que o compilador encontra aquele tipo de nó na árvore sintática — nesse caso, qualquer chamada de método

---

# 🎯 O resultado no editor

```csharp
public void ProcessarPedido()
{
    Console.WriteLine("Processando..."); // ⚠️ MEU001: Use ILogger em vez de Console.WriteLine
}
```

👉 Combinado com o Serilog (post 37), esse analyzer garante que ninguém volte a usar `Console.WriteLine` sem querer — o aviso aparece direto no editor, antes mesmo do code review

---

# 🔧 Code Fix: corrigindo automaticamente

```csharp
[ExportCodeFixProvider(LanguageNames.CSharp)]
public class UsarLoggerCodeFix : CodeFixProvider
{
    public override async Task RegisterCodeFixesAsync(CodeFixContext contexto)
    {
        var acao = CodeAction.Create(
            title: "Substituir por _logger.LogInformation",
            createChangedDocument: c => SubstituirPorLogger(contexto.Document, c));

        contexto.RegisterCodeFix(acao, contexto.Diagnostics.First());
    }
}
```

👉 Além de avisar, você pode oferecer a correção automática com um clique (`Ctrl + .` no Visual Studio) — o mesmo mecanismo que já sugere "adicionar using" ou "tornar método async"

---

# 📦 Distribuindo o Analyzer para o time

```xml
<!-- Diretorio.Build.props na raiz da solução -->
<Project>
  <ItemGroup>
    <PackageReference Include="MinhaEmpresa.Analyzers" Version="1.0.0" PrivateAssets="all" />
  </ItemGroup>
</Project>
```

👉 Empacotado como NuGet (mais sobre isso no próximo post) e referenciado no `Directory.Build.props`, o analyzer roda automaticamente em toda a solução, para todos os desenvolvedores — sem depender de configuração manual em cada máquina

---

# ⚠️ Erros comuns

- Criar analyzers para tudo, tornando o build lento e a experiência de desenvolvimento irritante  
- Definir severidade `Error` para regras estilísticas, bloqueando builds por preferências, não bugs reais  
- Não escrever testes para o próprio analyzer, correndo o risco de falsos positivos em massa  
- Ignorar analyzers já existentes na comunidade antes de escrever um do zero — muita coisa já existe pronta  

---

# 📌 Conclusão

- Roslyn expõe a árvore sintática do C#, permitindo escrever regras de análise customizadas  
- `DiagnosticAnalyzer` detecta padrões problemáticos e gera avisos direto no editor  
- `CodeFixProvider` oferece correções automáticas de um clique  
- Empacotado como NuGet, o analyzer garante consistência para todo o time, sem depender de lembrança individual  

👉 Com Roslyn Analyzers, as regras da sua equipe deixam de viver só na documentação e passam a ser aplicadas automaticamente pelo compilador

---

# 🔥 Próximo passo

Agora que você sabe criar e distribuir regras customizadas, o próximo nível é:

👉 **Fundamentos do C#: Criando e Publicando Pacotes NuGet**

Aqui você vai aprender a empacotar e compartilhar seu próprio código como uma biblioteca reutilizável.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
