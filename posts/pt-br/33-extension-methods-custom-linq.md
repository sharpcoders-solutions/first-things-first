# 🧠 Fundamentos do C#: Extension Methods e LINQ Customizado

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Introdução ao LINQ (post 19) — `Where`, `Select`, `OrderBy`  
- Como persistir dados de verdade com EF Core  

Toda vez que você escreveu `pedidos.Where(p => p.Valor > 100)`, usou um extension method sem parar para pensar como isso é possível — `Where` não é um método da classe `List<T>`. Chegou a hora de entender o mecanismo por trás disso, e criar os seus próprios.

👉 **Vamos aprender Extension Methods e LINQ Customizado**

---

# 💡 O que é um Extension Method?

👉 **Extension Method = um método estático que parece um método de instância, "adicionado" a um tipo que você não pode (ou não deveria) modificar**

```csharp
public static class ExtensoesString
{
    public static bool EhCpfValido(this string valor)
    {
        var digitos = valor.Where(char.IsDigit).ToArray();
        return digitos.Length == 11;
    }
}
```

```csharp
string cpf = "123.456.789-00";
bool valido = cpf.EhCpfValido(); // parece um método de string, mas não é
```

👉 O `this` antes do primeiro parâmetro é o que transforma um método estático comum em extension method — o compilador permite chamá-lo como se fosse parte da própria classe `string`, mesmo sem poder editar o BCL (Base Class Library)

---

# 🏗️ Como o LINQ é construído inteiramente com isso

```csharp
// Simplificação de como o próprio Where do LINQ funciona
public static class MeuLinq
{
    public static IEnumerable<T> MeuWhere<T>(this IEnumerable<T> origem, Func<T, bool> predicado)
    {
        foreach (var item in origem)
        {
            if (predicado(item))
                yield return item;
        }
    }
}
```

```csharp
var pedidosCaros = pedidos.MeuWhere(p => p.Valor > 100);
```

👉 `Where`, `Select`, `OrderBy` — tudo que você usa desde o post 19 é só uma coleção de extension methods sobre `IEnumerable<T>`, definidos em `System.Linq`. Não existe mágica de linguagem separada; é o mesmo mecanismo que você acabou de criar

---

# 🎯 Escrevendo seu próprio operador LINQ

```csharp
public static class ExtensoesPedido
{
    public static IEnumerable<Pedido> Vencidos(this IEnumerable<Pedido> pedidos)
    {
        return pedidos.Where(p => p.Status == "Pendente" && p.DataVencimento < DateTime.Now);
    }

    public static decimal ValorTotal(this IEnumerable<Pedido> pedidos)
    {
        return pedidos.Sum(p => p.Valor);
    }
}
```

```csharp
var total = pedidos.Vencidos().ValorTotal();
```

👉 Encadear `.Vencidos().ValorTotal()` funciona exatamente como `.Where().Select()` — porque é a mesma técnica. Extension methods sobre `IEnumerable<T>` que devolvem `IEnumerable<T>` continuam encadeáveis, formando consultas expressivas específicas do seu domínio

---

# 🔍 Extension methods vs herança

```csharp
// ❌ Não é possível: string é sealed, você não pode herdar dela
public class MinhaString : string { } 

// ✅ Extension method resolve isso sem herança
public static string ParaTitulo(this string valor) =>
    CultureInfo.CurrentCulture.TextInfo.ToTitleCase(valor.ToLower());
```

👉 Lembra do post sobre herança e polimorfismo (21)? Muitos tipos do .NET são `sealed` justamente para evitar hierarquias frágeis — extension methods dão flexibilidade sem exigir herança, e sem alterar o tipo original

---

# ⚠️ Erros comuns

- Criar extension methods para tipos que você controla, quando um método de instância normal seria mais direto e descobrível  
- Sobrecarregar namespaces com extension methods genéricos demais, poluindo o autocomplete de tipos básicos como `string` e `int`  
- Esquecer que extension methods não podem acessar membros `private` do tipo estendido — eles só enxergam a API pública  
- Nomear um extension method igual a um método de instância existente, criando confusão sobre qual será chamado  

---

# 📌 Conclusão

- Extension methods são métodos estáticos com `this` no primeiro parâmetro, chamados como se fossem de instância  
- Todo o LINQ (`Where`, `Select`, `OrderBy`) é construído inteiramente com essa técnica  
- Extension methods próprios sobre `IEnumerable<T>` continuam encadeáveis, como qualquer LINQ nativo  
- Eles resolvem o problema de "adicionar comportamento" sem herança, especialmente útil em tipos `sealed`  

👉 Com Extension Methods, você entende que o LINQ nunca foi mágica de linguagem — é uma API que você pode estender com a mesma técnica, criando seu próprio vocabulário fluente sobre qualquer coleção

---

# 🔥 Próximo passo

Agora que você sabe estender tipos sem herança, o próximo nível é:

👉 **Fundamentos do C#: Autenticação e Autorização com JWT**

Aqui você vai aprender a proteger sua API, garantindo que só usuários autenticados e autorizados acessem cada endpoint.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
