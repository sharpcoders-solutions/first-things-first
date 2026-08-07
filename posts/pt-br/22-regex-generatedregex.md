# 🧠 Fundamentos do C#: Regex e GeneratedRegex

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Tipos Numéricos e Precisão (`float`, `double` e `decimal`)  
- Métodos e strings, usados desde os primeiros posts para processar texto  

Você provavelmente já usou `Regex` para validar um e-mail ou extrair um padrão de texto, sem pensar muito no custo por trás disso. Existe uma pegadinha de performance clássica em `Regex`, e desde C# 11 existe uma forma completamente diferente (e melhor) de lidar com ela.

👉 **Vamos aprender Regex a fundo, e o source generator `GeneratedRegex`**

---

# 💡 O básico: `Regex` para validar e extrair padrões

```csharp
using System.Text.RegularExpressions;

var regex = new Regex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$");

bool valido = regex.IsMatch("usuario@exemplo.com"); // true
```

```csharp
var regexTelefone = new Regex(@"\((\d{2})\)\s?(\d{4,5})-(\d{4})");
var match = regexTelefone.Match("(11) 98765-4321");

if (match.Success)
{
    var ddd = match.Groups[1].Value;    // "11"
    var prefixo = match.Groups[2].Value; // "98765"
    var sufixo = match.Groups[3].Value;  // "4321"
}
```

👉 Grupos de captura (os parênteses no padrão) permitem extrair partes específicas de um texto que combina com o padrão — não só saber que combina, mas extrair os pedaços relevantes

---

# 💸 O problema de performance: compilação de regex é cara

```csharp
public bool ValidarEmail(string email)
{
    var regex = new Regex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$"); // ❌ recompila o padrão TODA vez
    return regex.IsMatch(email);
}
```

👉 Criar um `new Regex(padrao)` envolve interpretar o padrão e construir uma máquina de estados internamente — um trabalho que **não** é gratuito. Fazer isso dentro de um método chamado com frequência (validação em cada request de uma API, por exemplo) desperdiça CPU repetindo o mesmo trabalho de compilação sem necessidade

---

# 🏗️ A solução clássica: `static readonly Regex`

```csharp
public class Validador
{
    private static readonly Regex RegexEmail =
        new Regex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$", RegexOptions.Compiled);

    public bool ValidarEmail(string email) => RegexEmail.IsMatch(email);
}
```

👉 Guardar a instância em um campo `static readonly` garante que a compilação do padrão aconteça **uma única vez**, na inicialização da classe. `RegexOptions.Compiled` vai além, gerando IL otimizado especificamente para aquele padrão — mais rápido em execução, mas com um custo de inicialização ainda maior

---

# ⚡ `GeneratedRegex`: compilação em tempo de build, não em runtime

```csharp
public partial class Validador
{
    [GeneratedRegex(@"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$")]
    private static partial Regex RegexEmail();

    public bool ValidarEmail(string email) => RegexEmail().IsMatch(email);
}
```

👉 **`[GeneratedRegex]` = um atributo que instrui um source generator a gerar, em tempo de **compilação**, todo o código C# necessário para aquele padrão — sem interpretar nada em runtime**

Diferente de `RegexOptions.Compiled` (que gera IL dinamicamente na primeira execução), o `GeneratedRegex` já entrega código totalmente pronto no assembly compilado — zero custo de inicialização, e o padrão é validado **em tempo de compilação**, pegando erros de sintaxe de regex antes mesmo do código rodar

---

# 🔍 Anatomia do `GeneratedRegex`

```csharp
public partial class Validador
{
    [GeneratedRegex(@"\((\d{2})\)\s?(\d{4,5})-(\d{4})", RegexOptions.IgnoreCase)]
    private static partial Regex RegexTelefone();
}
```

👉 A classe precisa ser `partial`, o método precisa ser `partial` e `static`, e o atributo `[GeneratedRegex]` recebe o padrão (e opcionalmente `RegexOptions`) como argumento — o source generator produz a implementação completa por baixo dos panos, visível como código gerado se você inspecionar a saída da compilação

---

# ⚖️ Quando usar cada abordagem

| | `new Regex(padrao)` | `static readonly` + `Compiled` | `[GeneratedRegex]` |
|---|---|---|---|
| Padrão usado uma vez, script simples | ✅ ok | Desnecessário | Desnecessário |
| Padrão usado com frequência em produção | ❌ desperdício | ✅ bom | ✅ melhor |
| .NET 7+ disponível | — | — | ✅ recomendado |
| Padrão só conhecido em runtime (variável) | ✅ única opção | ✅ única opção | ❌ não suporta |

👉 **Regra prática: se o padrão é um literal conhecido em tempo de compilação e o projeto roda em .NET 7 ou mais recente, use `[GeneratedRegex]` por padrão.** Só volte para `new Regex()` dinâmico quando o padrão realmente vem de uma fonte externa (configuração, banco de dados) e não pode ser conhecido em tempo de compilação

---

# ⚠️ Erros comuns

- Criar um `new Regex(padrao)` dentro de um método chamado com frequência, recompilando o padrão desnecessariamente a cada chamada  
- Usar `RegexOptions.Compiled` em padrões usados raramente, pagando um custo de inicialização maior sem ganho real  
- Escrever padrões de regex excessivamente complexos, quando uma validação simples com `string.Contains`/`StartsWith` resolveria com muito mais clareza  
- Ignorar `[GeneratedRegex]` em projetos já em .NET 7+, perdendo uma otimização praticamente gratuita  

---

# 📌 Conclusão

- `Regex` interpreta e compila o padrão em runtime, um custo real se repetido desnecessariamente  
- `static readonly Regex` com `RegexOptions.Compiled` compila uma vez, na inicialização  
- `[GeneratedRegex]` usa um source generator para compilar o padrão em tempo de **build**, com custo de runtime zero  
- Use `GeneratedRegex` como padrão em .NET 7+; reserve `new Regex()` dinâmico para padrões só conhecidos em runtime  

👉 Com expressões regulares otimizadas no seu repertório, o próximo passo é voltar para o mundo orientado a objetos e começar a modelar conceitos do mundo real em código

---

# 🔥 Próximo passo

Agora que você domina expressões regulares com performance máxima, o próximo nível é:

👉 **Fundamentos do C#: Classes e Objetos (Introdução à Programação Orientada a Objetos)**

Aqui você começa a modelar o mundo real dentro do seu código.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
