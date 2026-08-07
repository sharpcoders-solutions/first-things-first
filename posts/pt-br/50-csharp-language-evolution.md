# 🧠 Fundamentos do C#: A Evolução da Linguagem C#

⏱️ Tempo de leitura: 9 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Anonymous types e por que `dynamic` deve ser evitado na maioria dos casos  
- Toda a base da linguagem, do `Console.WriteLine` a `Span<T>`, passando por enums, tuplas e tipos anônimos  

Você usou dezenas de recursos ao longo desta trilha — `record`, pattern matching, `async`/`await`, nullable reference types — sem parar para pensar de onde cada um veio. C# não nasceu assim. Esses recursos foram adicionados, versão a versão, ao longo de mais de vinte anos.

👉 **Vamos fazer um tour pela evolução do C#, e entender por que a linguagem chegou onde chegou**

---

# 💡 Por que isso importa na prática

👉 **Saber a origem de um recurso ajuda a entender seu propósito — e a reconhecer código legado que ainda não adotou a forma moderna de resolver o mesmo problema**

Em entrevistas técnicas e em code review, é comum encontrar código escrito no estilo de uma versão antiga do C#, mesmo em projetos que já rodam em uma versão recente do .NET. Entender essa linha do tempo ajuda você a modernizar código legado com confiança.

---

# 🏗️ C# 1.0 a 2.0: os fundamentos e os genéricos

```csharp
// C# 1.0 (2002): sem genéricos — ArrayList guarda "object", exige boxing
ArrayList lista = new ArrayList();
lista.Add(42); // boxing

// C# 2.0 (2005): genéricos — o marco que mudou tudo
List<int> listaGenerica = new List<int>();
listaGenerica.Add(42); // sem boxing
```

👉 Lembra do post sobre boxing e unboxing? A dor que motivou genéricos em C# 2.0 é exatamente essa: antes deles, toda coleção guardava `object`, forçando boxing em cada valor. C# 2.0 também trouxe `nullable<T>` (`int?`), iteradores com `yield return`, e métodos anônimos — a base para tudo que viria depois

---

# 🔧 C# 3.0: LINQ muda a forma de escrever código

```csharp
// Antes do LINQ: loops explícitos para filtrar e transformar
var caros = new List<Produto>();
foreach (var produto in produtos)
{
    if (produto.Preco > 100)
        caros.Add(produto);
}

// C# 3.0 (2007): LINQ, expressões lambda, tipos anônimos, var
var caros = produtos.Where(p => p.Preco > 100).ToList();
```

👉 C# 3.0 é provavelmente a versão que mais mudou o **estilo** de escrever C#. LINQ, lambdas, `var`, expression trees e anonymous types (do post anterior) chegaram todos juntos — e a maior parte do código C# "moderno" que você já viu nesta trilha usa recursos desta versão

---

# ⚡ C# 4.0 a 5.0: dynamic e async/await

```csharp
// C# 4.0 (2010): dynamic, parâmetros opcionais e nomeados
void Registrar(string mensagem, string nivel = "Info") { }
Registrar(mensagem: "Erro crítico", nivel: "Error");

// C# 5.0 (2012): async/await — o maior salto em programação assíncrona
public async Task<Produto> BuscarProdutoAsync(int id)
{
    return await _repositorio.ObterPorIdAsync(id);
}
```

👉 `async`/`await` (C# 5.0) merece destaque especial: antes dele, código assíncrono em .NET significava callbacks aninhados, difíceis de ler e de depurar. Essa única mudança de sintaxe é a razão pela qual todo o código assíncrono que você escreveu nesta trilha parece código síncrono comum

---

# 🎯 C# 6.0 a 7.0: qualidade de vida e pattern matching inicial

```csharp
// C# 6.0 (2015): string interpolation, expression-bodied members
public string NomeCompleto => $"{Nome} {Sobrenome}";

// C# 7.0 (2017): tuplas (do post anterior!), pattern matching básico, out inline
if (obj is Produto produto && produto.Preco > 0)
{
    Console.WriteLine(produto.Nome);
}
```

👉 A tupla que você aprendeu no post anterior nasceu exatamente em C# 7.0 — junto com `is` pattern matching, que evoluiu bastante nas versões seguintes até virar o pattern matching robusto que você já usa hoje

---

# 🛡️ C# 8.0: nullable reference types e mais pattern matching

```csharp
#nullable enable

public string? Apelido { get; set; } // pode ser null, e o compilador sabe disso
public string Nome { get; set; } = ""; // não pode ser null

// switch expressions
string categoria = preco switch
{
    < 50 => "Barato",
    < 200 => "Médio",
    _ => "Caro"
};
```

👉 Nullable reference types (`string?` vs `string`) foi uma das mudanças mais impactantes para reduzir `NullReferenceException` em produção — o compilador passou a avisar quando você tenta usar um valor que pode ser `null` sem verificar antes

---

# 🚀 C# 9.0 a 12.0: `record`, tipos abstratos estáticos, e a linguagem que você usa hoje

```csharp
// C# 9.0 (2020): record — a base de tudo que você já usou em DTOs
public record Produto(int Id, string Nome, decimal Preco);

// C# 10-11: required members, generic math, raw string literals
public class Pedido
{
    public required string Cliente { get; init; }
}

// C# 12.0 (2023): primary constructors em classes comuns
public class ServicoProdutos(IProdutoRepositorio repositorio)
{
    public Produto ObterPorId(int id) => repositorio.ObterPorId(id);
}
```

👉 Cada versão recente adiciona menos "recursos revolucionários" e mais **refinamentos** de sintaxe — `record`, `required`, primary constructors — todos reduzindo a quantidade de código repetitivo (boilerplate) necessária para expressar a mesma intenção

---

# 📅 Como o C# evolui hoje: ciclo anual, aberto no GitHub

👉 Desde 2020, uma nova versão principal do C# é lançada **todo ano**, junto com a versão correspondente do .NET, e todo o processo de design acontece publicamente no repositório `dotnet/csharplang` no GitHub — qualquer desenvolvedor pode acompanhar (ou até propor) discussões sobre o futuro da linguagem

---

# ⚠️ Erros comuns

- Escrever código em estilo pré-C# 3.0 (loops manuais em vez de LINQ) em um projeto que já roda em uma versão recente do .NET  
- Ignorar nullable reference types em projetos novos, perdendo os avisos de compilador que previnem `NullReferenceException`  
- Achar que "a versão mais nova da linguagem" e "a versão mais nova do .NET" são a mesma coisa — são conceitos relacionados, mas diferentes  
- Adotar recursos muito novos (preview features) em código de produção sem avaliar a estabilidade  

---

# 📌 Conclusão

- Genéricos (C# 2.0) eliminaram o boxing forçado das coleções antigas  
- LINQ e lambdas (C# 3.0) mudaram o estilo predominante de escrever C#  
- `async`/`await` (C# 5.0) tornou código assíncrono legível como código síncrono  
- Nullable reference types (C# 8.0) e `record` (C# 9.0) reduziram bugs e boilerplate diretamente  
- O C# evolui em ciclo anual, com design aberto no GitHub  

👉 Toda a linguagem que você dominou nesta trilha é resultado de mais de vinte anos de evolução incremental — e essa evolução continua a cada nova versão

---

# 🔥 Próximo passo

Agora que você entende a jornada da linguagem, o próximo nível é:

👉 **Fundamentos do C#: Feature Flags e Configuração Dinâmica**

Aqui você vai aprender a lançar funcionalidades novas com segurança, sem depender de um novo deploy para cada mudança.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
