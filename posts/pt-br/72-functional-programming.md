# 🧠 Fundamentos do C#: Programação Funcional em C#

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Channels para coordenação assíncrona  
- Records e imutabilidade (post 27), LINQ (post 19), delegates e lambdas (post 25)  

O C# é uma linguagem orientada a objetos por natureza, mas ao longo desta trilha você já usou vários conceitos funcionais sem nomear formalmente. Hora de conectar os pontos.

👉 **Vamos aprender Programação Funcional em C#**

---

# 💡 Os pilares da programação funcional

## 🔹 Imutabilidade

```csharp
// Já usamos isso desde o post 27
public record Pedido(int Id, decimal Valor, string Status);

var pedido = new Pedido(1, 100m, "Novo");
var pedidoAtualizado = pedido with { Status = "Confirmado" }; // cria um novo objeto
```

👉 Em vez de modificar o objeto original, você cria uma nova versão — o mesmo `with` que você já usa desde o post sobre records

## 🔹 Funções puras

```csharp
// ✅ Pura: mesma entrada, sempre a mesma saída, sem efeitos colaterais
public static decimal CalcularDesconto(decimal valor, decimal percentual)
    => valor * (1 - percentual);

// ❌ Impura: depende de estado externo e tem efeito colateral
private decimal _totalAcumulado;
public decimal CalcularDescontoImpuro(decimal valor)
{
    _totalAcumulado += valor; // efeito colateral
    return valor * 0.9m;
}
```

👉 Funções puras são mais fáceis de testar (lembra do post 30?) — sem mocks, sem setup, só entrada e saída

---

# 🏗️ Composição de funções

```csharp
Func<decimal, decimal> aplicarDesconto = valor => valor * 0.9m;
Func<decimal, decimal> aplicarFrete = valor => valor + 15m;
Func<decimal, decimal> arredondar = valor => Math.Round(valor, 2);

Func<decimal, decimal> calcularTotal = valor =>
    arredondar(aplicarFrete(aplicarDesconto(valor)));

var total = calcularTotal(100m); // 100 → 90 → 105 → 105.00
```

👉 Lembra do post sobre delegates (25)? `Func<T, TResult>` já é a base disso — composição é simplesmente encadear pequenas funções puras para formar um comportamento maior

---

# 🎯 LINQ é programação funcional

```csharp
var pedidosCaros = pedidos
    .Where(p => p.Valor > 100)        // filtra, sem modificar a lista original
    .Select(p => p with { Status = "Prioritario" }) // transforma, sem mutação
    .OrderByDescending(p => p.Valor)
    .ToList();
```

👉 Você usa LINQ desde o post 19 — e cada método (`Where`, `Select`, `OrderBy`) é uma função pura que recebe uma sequência e devolve outra, sem modificar a original. Isso é programação funcional em ação, mesmo sem ter esse nome naquele momento

---

# 🚫 Evitando exceções como controle de fluxo: Option/Result

```csharp
public record Resultado<T>(bool Sucesso, T? Valor, string? Erro);

public Resultado<Pedido> BuscarPedido(int id)
{
    var pedido = _repositorio.Buscar(id);

    return pedido is not null
        ? new Resultado<Pedido>(true, pedido, null)
        : new Resultado<Pedido>(false, null, "Pedido não encontrado");
}
```

```csharp
var resultado = BuscarPedido(123);

if (resultado.Sucesso)
    Console.WriteLine(resultado.Valor);
else
    Console.WriteLine(resultado.Erro);
```

👉 Em vez de lançar exceções para fluxo de controle esperado (lembra do post 23, sobre quando usar exceções?), o padrão `Result` torna o caminho de erro explícito no tipo de retorno — o chamador é forçado a lidar com ambos os casos

---

# ⚠️ Erros comuns

- Tentar transformar C# em uma linguagem 100% funcional, ignorando que OOP ainda é o paradigma central da linguagem  
- Abusar de composição de funções a ponto de tornar o código difícil de debugar  
- Usar `Result<T>` para todo erro, mesmo situações genuinamente excepcionais que deveriam lançar exceção  
- Esquecer que imutabilidade tem custo de alocação — usar com critério em loops de alta performance  

---

# 📌 Conclusão

- Programação funcional não é uma linguagem separada, é um conjunto de princípios que o C# já incorpora  
- Imutabilidade, funções puras e composição já apareceram nesta trilha desde records e LINQ  
- O padrão `Result` torna erros esperados explícitos, em vez de usar exceções como controle de fluxo  
- C# é multi-paradigma — o objetivo é usar o melhor conceito para cada problema, não escolher um "lado"  

👉 Com programação funcional, você enxerga que boa parte do C# moderno já é funcional por natureza, mesmo escrevendo classes o dia inteiro

---

# 🔥 Próximo passo

Agora que você conecta os conceitos funcionais do C#, o próximo nível é:

👉 **Fundamentos do C#: Covariância e Contravariância**

Aqui você vai aprender como generics podem ser flexíveis com hierarquias de tipos, e quando isso é seguro.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
