# 🧠 Fundamentos do C#: Required Members e Init-Only Properties

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Pattern matching avançado com padrões de propriedade, lista e tupla  
- `record`s e propriedades imutáveis, desde o post sobre C# moderno  

Você já usa `record` e sabe que ele favorece imutabilidade. Mas o que garante que um objeto seja criado **corretamente**, com todas as propriedades essenciais preenchidas, sem depender de um construtor gigante ou de disciplina manual do time? É isso que `required` e `init` resolvem juntos.

👉 **Vamos entender `required` members e `init`-only properties**

---

# 💡 O problema: propriedades opcionais que na verdade são obrigatórias

```csharp
public class Produto
{
    public string Nome { get; set; } = default!;
    public decimal Preco { get; set; }
}

var produto = new Produto(); // ✅ compila, mas Nome está vazio e Preco é zero
```

👉 Usar `object initializer` (`new Produto { Nome = "...", Preco = ... }`) é conveniente, mas nada **obriga** quem cria o objeto a preencher `Nome` — o compilador aceita `new Produto()` silenciosamente, mesmo que um produto sem nome não faça sentido no seu domínio

---

# ✅ `required`: obrigando o preenchimento em tempo de compilação

```csharp
public class Produto
{
    public required string Nome { get; set; }
    public required decimal Preco { get; set; }
    public string? Descricao { get; set; } // continua opcional
}

var produto1 = new Produto(); // ❌ erro de compilação: Nome e Preco são obrigatórios
var produto2 = new Produto { Nome = "Notebook", Preco = 3500m }; // ✅ ok
```

👉 **`required` = uma propriedade que o compilador exige que seja preenchida na inicialização, através de object initializer**

Diferente de validação em runtime (lembra do FluentValidation, do post sobre segurança de APIs?), `required` pega o erro **antes mesmo do código rodar** — a classe de erro "esqueci de preencher um campo obrigatório" desaparece completamente em tempo de desenvolvimento

---

# 🔒 `init`: propriedades que só podem ser definidas na criação

```csharp
public class Produto
{
    public required string Nome { get; init; }
    public required decimal Preco { get; init; }
}

var produto = new Produto { Nome = "Notebook", Preco = 3500m };
produto.Nome = "Outro nome"; // ❌ erro de compilação: Nome só pode ser definido na inicialização
```

👉 **`init` = como `set`, mas só pode ser chamado durante a criação do objeto (object initializer ou construtor) — depois disso, a propriedade se torna somente leitura**

Lembra da diferença entre `record` e `class` mutável comum? `init` é exatamente o mecanismo que dá aos `record`s (e a qualquer `class` que você queira) esse comportamento de imutabilidade após a construção

---

# 🎯 `required` + `init`: a combinação mais comum

```csharp
public class ConfiguracaoApi
{
    public required string UrlBase { get; init; }
    public required int TempoLimiteSegundos { get; init; }
    public int TentativasMaximas { get; init; } = 3; // opcional, com valor padrão
}

var config = new ConfiguracaoApi
{
    UrlBase = "https://api.exemplo.com",
    TempoLimiteSegundos = 30
    // TentativasMaximas usa o padrão (3)
};
```

👉 Essa combinação é o padrão recomendado para DTOs, opções de configuração (lembra do Options Pattern, post 79?) e qualquer classe onde algumas propriedades são essenciais e outras são realmente opcionais — o compilador garante os campos obrigatórios, `init` garante que ninguém muda esses valores depois

---

# 🏗️ `required` com construtores

```csharp
public class Produto
{
    public required string Nome { get; init; }

    [SetsRequiredMembers]
    public Produto(string nome)
    {
        Nome = nome;
    }
}

var produto1 = new Produto("Notebook");           // ✅ construtor cobre o required
var produto2 = new Produto { Nome = "Notebook" };  // ✅ object initializer também funciona
```

👉 Se você prefere expor um construtor tradicional em vez de forçar object initializer, o atributo `[SetsRequiredMembers]` avisa o compilador que aquele construtor específico já garante que todos os membros `required` foram preenchidos

---

# ⚖️ `required`/`init` vs construtor posicional de `record`

```csharp
// record: obrigatoriedade via posição no construtor primário
public record ProdutoRecord(string Nome, decimal Preco);

// class com required/init: obrigatoriedade via nome, mais flexível para muitas propriedades opcionais
public class ProdutoClasse
{
    public required string Nome { get; init; }
    public required decimal Preco { get; init; }
    public string? Descricao { get; init; }
    public string? Categoria { get; init; }
}
```

👉 **Regra prática: use o construtor posicional de `record` quando todas (ou quase todas) as propriedades são obrigatórias e a ordem é natural. Use `required`/`init` em uma `class` quando você tem uma mistura de muitas propriedades obrigatórias e opcionais** — nomear cada uma explicitamente na criação fica mais legível que um construtor com dez parâmetros posicionais

---

# ⚠️ Erros comuns

- Usar `= default!` para "enganar" o compilador em vez de `required`, perdendo a garantia real de preenchimento  
- Esquecer `init` em propriedades que deveriam ser imutáveis após a criação, permitindo mutação acidental mais tarde no código  
- Aplicar `required` em toda propriedade de uma classe, mesmo as genuinamente opcionais, forçando quem cria o objeto a preencher campos desnecessários  
- Confundir `init` com `readonly` — `readonly` só pode ser definido no construtor; `init` pode ser definido em qualquer inicializador, incluindo object initializers  

---

# 📌 Conclusão

- `required` obriga, em tempo de compilação, que uma propriedade seja preenchida na criação do objeto  
- `init` permite definir uma propriedade só durante a criação, tornando-a somente leitura depois  
- A combinação `required` + `init` é o padrão recomendado para DTOs e classes de configuração  
- `[SetsRequiredMembers]` permite usar construtores tradicionais mesmo com propriedades `required`  

👉 Com objetos que se auto-validam em tempo de compilação, o próximo passo é olhar para outro recurso que evoluiu bastante: expressões regulares, e como compilá-las para performance máxima

---

# 🔥 Próximo passo

Agora que você garante objetos corretamente inicializados em tempo de compilação, o próximo nível é:

👉 **Fundamentos do C#: Reflection.Emit e Geração de Código em Runtime (IL Emit)**

Aqui você vai aprender a gerar métodos e tipos inteiros dinamicamente, em memória, indo além do que reflection tradicional permite.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
