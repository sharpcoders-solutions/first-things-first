# 🧠 Fundamentos do C#: Static Abstract Interface Members

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Indexers para dar sintaxe de colchetes aos seus tipos  
- Operator overloading para `+`, `==` e comparações customizadas  

Interfaces sempre exigiram membros de **instância** — métodos e propriedades que só existem depois que você já tem um objeto criado. Desde C# 11, isso mudou: interfaces podem exigir membros **estáticos**, incluindo operadores. Vamos entender por que isso importa.

👉 **Vamos conhecer static abstract interface members**

---

# 💡 O problema que esse recurso resolve

```csharp
// Antes de C# 11: impossível expressar isso de forma genérica
public T Somar<T>(T a, T b)
{
    return a + b; // ❌ Erro: o compilador não sabe se T suporta "+"
}
```

👉 Genéricos (post sobre C# 2.0) sempre tiveram uma limitação: você não conseguia exigir que um tipo genérico `T` suportasse operadores como `+`, porque operadores são membros **estáticos**, e interfaces só podiam exigir membros de instância

---

# 🏗️ Static abstract members em interfaces

```csharp
public interface ICriavel<T>
{
    static abstract T Criar();
}

public class Produto : ICriavel<Produto>
{
    public string Nome { get; set; }

    public static Produto Criar() => new Produto { Nome = "Novo Produto" };
}

T CriarNovo<T>() where T : ICriavel<T> => T.Criar();

var produto = CriarNovo<Produto>();
```

👉 **`static abstract` = um membro que a interface exige que exista no **tipo**, não em uma instância dele**

Repare em `T.Criar()` — isso só é possível porque a constraint `where T : ICriavel<T>` garante ao compilador que qualquer `T` usado ali tem um método estático `Criar()`. Antes de C# 11, isso era simplesmente impossível de expressar

---

# ➕ O caso de uso real: operadores genéricos

```csharp
public interface INumero<T> where T : INumero<T>
{
    static abstract T operator +(T a, T b);
    static abstract T Zero { get; }
}

public struct Dinheiro : INumero<Dinheiro>
{
    public decimal Valor { get; }

    public Dinheiro(decimal valor) => Valor = valor;

    public static Dinheiro operator +(Dinheiro a, Dinheiro b) => new Dinheiro(a.Valor + b.Valor);
    public static Dinheiro Zero => new Dinheiro(0);
}

T Somar<T>(IEnumerable<T> valores) where T : INumero<T>
{
    T total = T.Zero;
    foreach (var valor in valores)
        total += valor; // usa o "operator +" do tipo T
    return total;
}
```

👉 Lembra do post sobre operator overloading? Agora esse `operator +` pode ser **exigido por uma interface**, e um método genérico consegue somar valores de qualquer tipo que implemente essa interface — `Dinheiro`, `int` (com generic math nativo), ou qualquer tipo customizado seu

---

# 🔢 Generic Math: o motivo real dessa mudança

```csharp
using System.Numerics;

T SomarTodos<T>(IEnumerable<T> numeros) where T : INumber<T>
{
    T total = T.Zero;
    foreach (var numero in numeros)
        total += numero;
    return total;
}

var somaInteiros = SomarTodos(new[] { 1, 2, 3 });        // funciona com int
var somaDecimais = SomarTodos(new[] { 1.5m, 2.5m });     // funciona com decimal
var somaDoubles = SomarTodos(new[] { 1.1, 2.2 });        // funciona com double
```

👉 A interface `INumber<T>`, do próprio .NET, já usa static abstract members para unificar `int`, `double`, `decimal` e outros tipos numéricos sob uma única abstração — pela primeira vez, dá para escrever **um único método genérico** que funciona com qualquer tipo numérico, sem duplicar código para cada um

---

# 🎯 Quando você realmente precisa disso

👉 **Na prática do dia a dia, você raramente vai *declarar* uma interface com static abstract members — mas vai se beneficiar delas o tempo todo, usando `INumber<T>` e interfaces relacionadas da BCL**

Casos onde faz sentido criar sua própria interface com static abstract members:

- Bibliotecas genéricas que precisam operar sobre múltiplos tipos numéricos ou "criáveis"  
- Frameworks que definem um contrato de fábrica (`static abstract T Criar()`) para tipos plugáveis  
- Código de infraestrutura compartilhado entre múltiplos tipos que devem seguir o mesmo "protocolo estático"  

---

# ⚠️ Erros comuns

- Tentar usar static abstract members em versões do C# anteriores à 11, onde o recurso simplesmente não existe  
- Criar interfaces com static abstract members para problemas simples, quando um método estático comum resolveria com menos complexidade  
- Esquecer a constraint `where T : IMinhaInterface<T>` (o padrão "curiously recurring"), sem a qual `T.MetodoEstatico()` não compila  
- Confundir `static abstract` (exigido pela interface, implementado pela classe) com `static virtual` (tem uma implementação padrão, mas pode ser sobrescrita)  

---

# 📌 Conclusão

- `static abstract` permite que interfaces exijam membros estáticos, incluindo operadores  
- Isso resolve uma limitação histórica de genéricos: não dava pra exigir que `T` suportasse `+`, `-`, etc  
- `INumber<T>` da BCL usa esse recurso para unificar todos os tipos numéricos sob uma única abstração genérica  
- Na prática, você usa mais essas interfaces do .NET do que cria as suas próprias  

👉 Com iteradores, operadores customizados, indexers e static abstract members, você completa o quadro dos recursos mais avançados de tipos do C# — a base perfeita para voltar ao mundo prático de testes e infraestrutura

---

# 🔥 Próximo passo

Agora que você domina os recursos mais avançados de tipos, o próximo nível é:

👉 **Fundamentos do C#: Testes de Integração com WebApplicationFactory**

Aqui você vai aprender a testar sua API inteira, do HTTP ao banco de dados, com testes automatizados de verdade.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
