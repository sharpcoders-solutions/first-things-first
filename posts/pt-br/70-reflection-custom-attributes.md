# 🧠 Fundamentos do C#: Reflection e Atributos Customizados

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Unsafe code, saindo da camada gerenciada do C#  
- `[HttpGet]`, `[Authorize]`, `[Fact]` — atributos que você já usa desde os primeiros posts sobre ASP.NET Core e xUnit  

Você já usou dezenas de atributos ao longo desta trilha, mas nunca perguntou: como o ASP.NET Core sabe que `[HttpGet]` marca um endpoint? A resposta é Reflection.

👉 **Vamos aprender Reflection e Atributos Customizados**

---

# 💡 O que é Reflection?

👉 **Reflection = a capacidade do C# de inspecionar tipos, métodos e propriedades em tempo de execução, não só em tempo de compilação**

```csharp
Type tipo = typeof(Pedido);

Console.WriteLine(tipo.Name); // "Pedido"

foreach (var propriedade in tipo.GetProperties())
{
    Console.WriteLine($"{propriedade.Name}: {propriedade.PropertyType}");
}
```

👉 Normalmente, você acessa `pedido.Id` diretamente no código. Com Reflection, você descobre que a propriedade `Id` existe, qual é o tipo dela, e pode até ler/escrever o valor — tudo isso sem saber de antemão qual é a classe

---

# 🏗️ Criando um atributo customizado

```csharp
[AttributeUsage(AttributeTargets.Property)]
public class ObrigatorioParaExportacaoAttribute : Attribute
{
    public string Motivo { get; }

    public ObrigatorioParaExportacaoAttribute(string motivo)
    {
        Motivo = motivo;
    }
}
```

```csharp
public class Pedido
{
    [ObrigatorioParaExportacao("Sistema fiscal exige este campo")]
    public string CpfCliente { get; set; } = default!;

    public string Observacoes { get; set; } = default!;
}
```

👉 Um atributo customizado nada mais é do que uma classe que herda de `Attribute` — o mesmo mecanismo por trás de `[Fact]` (post 33) e `[Authorize]` (post 37)

---

# 🎯 Lendo o atributo em tempo de execução

```csharp
public static void ValidarCamposObrigatorios<T>(T objeto)
{
    var propriedades = typeof(T).GetProperties();

    foreach (var propriedade in propriedades)
    {
        var atributo = propriedade.GetCustomAttribute<ObrigatorioParaExportacaoAttribute>();

        if (atributo != null)
        {
            var valor = propriedade.GetValue(objeto);

            if (valor is null || string.IsNullOrWhiteSpace(valor.ToString()))
            {
                throw new InvalidOperationException(
                    $"{propriedade.Name} é obrigatório: {atributo.Motivo}");
            }
        }
    }
}
```

👉 `GetCustomAttribute` procura o atributo na propriedade, e `GetValue`/`SetValue` leem e escrevem valores dinamicamente — isso é exatamente como frameworks de validação (FluentValidation) e serialização (System.Text.Json) funcionam por baixo dos panos

---

# 🔍 Como o ASP.NET Core usa isso de verdade

```csharp
[HttpGet("{id}")]
public IActionResult ObterPedido(int id) => Ok(_servico.Buscar(id));
```

👉 Quando a aplicação inicia, o ASP.NET Core usa Reflection para escanear todos os controllers, encontrar métodos marcados com `[HttpGet]`, `[HttpPost]` etc., e construir a tabela de rotas automaticamente — o mesmo mecanismo que você acabou de aprender, rodando em escala dentro do framework

---

# ⚠️ Erros comuns

- Usar Reflection em loops de alta performance sem cache — `GetProperties()` e `GetCustomAttribute()` são relativamente lentos comparados a acesso direto  
- Ignorar `BindingFlags` ao buscar membros privados, gerando resultados vazios inesperados  
- Usar Reflection para contornar encapsulamento (acessar campos `private` de fora), quebrando os princípios de OOP do post 23  
- Não considerar Source Generators (próximo post) como alternativa mais rápida quando a mesma tarefa é conhecida em tempo de compilação  

---

# 📌 Conclusão

- Reflection inspeciona e manipula tipos em tempo de execução  
- Atributos customizados são classes que herdam de `Attribute`, lidas via Reflection  
- Frameworks como ASP.NET Core usam exatamente esse mecanismo para descobrir rotas, validações e configurações  
- O custo de performance da Reflection deve ser considerado em código crítico  

👉 Com Reflection, você entende como o "mágico" comportamento dos atributos que usa desde o início da trilha realmente funciona por baixo dos panos

---

# 🔥 Próximo passo

Agora que você entende Reflection, o próximo nível é:

👉 **Fundamentos do C#: Source Generators**

Aqui você vai aprender uma alternativa moderna e mais rápida à Reflection, gerando código em tempo de compilação.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
