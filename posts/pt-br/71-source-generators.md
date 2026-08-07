# 🧠 Fundamentos do C#: Source Generators

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Reflection para inspecionar tipos em tempo de execução  
- Roslyn Analyzers para criar regras de análise customizadas (post 65)  

Reflection é poderosa, mas tem um custo: roda em tempo de execução, toda vez. Source Generators usam a mesma infraestrutura do Roslyn, mas geram código real em tempo de **compilação** — sem custo nenhum quando a aplicação está rodando.

👉 **Vamos aprender Source Generators**

---

# 💡 O problema que Source Generators resolvem

```csharp
// Com Reflection (post 70): lento, roda a cada chamada
public static void SerializarComReflection(object objeto)
{
    var propriedades = objeto.GetType().GetProperties(); // custo em runtime
    // ...
}
```

```csharp
// Com Source Generator: o código já existe, compilado, sem Reflection nenhuma
public static void SerializarPedido(Pedido pedido)
{
    // código gerado automaticamente durante o build,
    // sem nenhuma chamada de Reflection em runtime
}
```

👉 O `System.Text.Json` (que você usa desde o post sobre APIs) tem um modo de Source Generator justamente por isso — serialização sem Reflection é significativamente mais rápida

---

# 🏗️ Como um Source Generator funciona

```csharp
[Generator]
public class GeradorToString : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext contexto)
    {
        var classes = contexto.SyntaxProvider
            .CreateSyntaxProvider(
                predicate: (no, _) => no is ClassDeclarationSyntax,
                transform: (ctx, _) => (ClassDeclarationSyntax)ctx.Node)
            .Where(classe => TemAtributo(classe, "GerarToString"));

        contexto.RegisterSourceOutput(classes, (spc, classe) =>
        {
            var codigo = GerarCodigoToString(classe);
            spc.AddSource($"{classe.Identifier}_ToString.g.cs", codigo);
        });
    }
}
```

👉 Isso lembra o Roslyn Analyzer do post 65 — mas em vez de só reportar um diagnóstico, o generator **adiciona arquivos de código C# novos** à compilação, antes mesmo do build terminar

---

# 🎯 Usando na prática

```csharp
[GerarToString]
public partial class Pedido
{
    public int Id { get; set; }
    public decimal Valor { get; set; }
}
```

```csharp
// Arquivo gerado automaticamente: Pedido_ToString.g.cs
public partial class Pedido
{
    public override string ToString() => $"Pedido {{ Id = {Id}, Valor = {Valor} }}";
}
```

👉 Você escreve `partial class Pedido` com o atributo, e o compilador gera o resto — parece mágica, mas é só código C# normal, escrito por outro código C#, durante a compilação

---

# ⚡ Onde isso já é usado no mundo real

- **System.Text.Json** — serialização sem Reflection via `[JsonSerializable]`  
- **Regex** — `[GeneratedRegex]` compila expressões regulares em tempo de build, em vez de interpretá-las em runtime  
- **ASP.NET Core Minimal APIs** — geração de código para roteamento de alta performance  

```csharp
public partial class MeuRegex
{
    [GeneratedRegex(@"^\d{3}-\d{4}$")]
    public static partial Regex TelefoneRegex();
}
```

👉 Lembra do post sobre LINQ e performance? `[GeneratedRegex]` é literalmente mais rápido que `new Regex(...)` porque o padrão é compilado durante o build, não interpretado a cada execução

---

# ⚠️ Erros comuns

- Escrever Source Generators para problemas simples que Reflection já resolveria sem problema de performance real  
- Não testar o generator isoladamente, tornando difícil depurar código gerado incorretamente  
- Gerar código que conflita com código escrito manualmente na mesma `partial class`  
- Subestimar a curva de aprendizado — Source Generators exigem entender a árvore sintática do Roslyn, como no post 65  

---

# 📌 Conclusão

- Source Generators criam código C# real durante a compilação, não em runtime  
- Isso elimina o overhead de Reflection em cenários de alta performance  
- `System.Text.Json` e `[GeneratedRegex]` são exemplos reais já usados nesta trilha  
- A base técnica é a mesma dos Roslyn Analyzers do post 65, mas gerando código em vez de só diagnósticos  

👉 Com Source Generators, você troca o custo de runtime da Reflection pelo custo (único) de compilação — o melhor dos dois mundos em cenários de performance crítica

---

# 🔥 Próximo passo

Agora que você sabe gerar código em tempo de compilação, o próximo nível é:

👉 **Fundamentos do C#: Native AOT**

Aqui você vai aprender a compilar sua aplicação C# direto para código nativo, sem depender do runtime .NET instalado na máquina de destino.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
