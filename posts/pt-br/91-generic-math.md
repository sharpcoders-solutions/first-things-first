# 🧠 Fundamentos do C#: Generic Math

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- SIMD e Hardware Intrinsics, processando vários valores numéricos em paralelo  
- Static abstract interface members (post 61), incluindo o exemplo de `INumber<T>`  

No post sobre static abstract interface members, você viu de relance a interface `INumber<T>`. Agora é hora de explorar Generic Math a fundo — o recurso que resolve um problema que existia desde a primeira versão de genéricos do C#.

👉 **Vamos dominar Generic Math**

---

# 💡 O problema histórico: genéricos não suportavam operadores

```csharp
// Antes de C# 11, isso era impossível de escrever
public T Somar<T>(T a, T b)
{
    return a + b; // ❌ o compilador não sabe se T suporta "+"
}
```

👉 Lembra do post sobre static abstract interface members? Antes de C# 11, você precisava escrever uma sobrecarga separada de cada método matemático genérico para `int`, `double`, `decimal`, `float`... — duplicação de código que existia desde sempre na linguagem, sem solução elegante

---

# 🔢 `INumber<T>`: a interface que unifica todos os tipos numéricos

```csharp
using System.Numerics;

public T Somar<T>(T a, T b) where T : INumber<T> => a + b;

int somaInt = Somar(3, 4);           // 7
double somaDouble = Somar(3.5, 4.2); // 7.7
decimal somaDecimal = Somar(3.5m, 4.2m); // 7.7
```

👉 `INumber<T>` é implementada por **todos** os tipos numéricos primitivos do .NET (`int`, `long`, `float`, `double`, `decimal`, e até tipos como `BigInteger`) — um único método genérico agora funciona com qualquer um deles, sem nenhuma duplicação

---

# 🎯 Operações matemáticas genéricas de verdade

```csharp
public T Media<T>(IEnumerable<T> valores) where T : INumber<T>
{
    T soma = T.Zero;
    int quantidade = 0;

    foreach (var valor in valores)
    {
        soma += valor;
        quantidade++;
    }

    return soma / T.CreateChecked(quantidade);
}

var mediaPrecos = Media(new decimal[] { 10.5m, 20.3m, 15.8m });
var mediaNotas = Media(new double[] { 8.5, 9.0, 7.5 });
```

👉 `T.Zero` e `T.CreateChecked` também vêm de `INumber<T>` — o valor zero "genérico" e uma forma segura de converter outro tipo numérico (como `int quantidade`) para o tipo `T` sendo usado, sem boxing (lembra do post 47?) e sem casts manuais arriscados

---

# 🔍 Outras interfaces do namespace `System.Numerics`

```csharp
// IComparisonOperators: <, >, <=, >=
public T Maximo<T>(T a, T b) where T : IComparisonOperators<T, T, bool> => a > b ? a : b;

// IAdditionOperators: só a operação de soma, sem exigir todo o resto de INumber<T>
public T SomarApenas<T>(T a, T b) where T : IAdditionOperators<T, T, T> => a + b;

// IMinMaxValue: valores mínimo e máximo do tipo
public T ValorMaximoDoTipo<T>() where T : IMinMaxValue<T> => T.MaxValue;
```

👉 `INumber<T>` é a interface "completa", mas o .NET oferece interfaces mais granulares — se seu método só precisa de comparação, ou só de adição, você pode restringir a constraint exatamente ao que é necessário, seguindo o mesmo princípio de segregação de interfaces que você já viu em SOLID

---

# 📊 Um caso de uso real: uma biblioteca de estatística genérica

```csharp
public static class EstatisticasGenericas
{
    public static T Somar<T>(this IEnumerable<T> valores) where T : INumber<T>
    {
        T total = T.Zero;
        foreach (var valor in valores) total += valor;
        return total;
    }

    public static T Maximo<T>(this IEnumerable<T> valores) where T : INumber<T>, IMinMaxValue<T>
    {
        T maximo = T.MinValue;
        foreach (var valor in valores)
            if (valor > maximo) maximo = valor;
        return maximo;
    }
}

var totalVendas = vendasDoMes.Somar();      // funciona com decimal
var maiorTemperatura = temperaturas.Maximo(); // funciona com double
```

👉 Lembra do post sobre extension methods e LINQ? Isso é exatamente o mesmo padrão — só que agora, combinado com Generic Math, seus extension methods funcionam com **qualquer** tipo numérico, ao invés de precisar de uma versão para `int`, outra para `decimal`, outra para `double`

---

# ⚠️ Erros comuns

- Usar Generic Math para problemas simples que já seriam resolvidos com uma única sobrecarga concreta, adicionando complexidade desnecessária  
- Esquecer que `T.CreateChecked` pode lançar exceção em conversões que perderiam dados — use `T.CreateSaturating` ou `T.CreateTruncating` quando esse comportamento não é desejado  
- Misturar tipos numéricos diferentes em uma mesma coleção genérica sem conversão, esperando que o generic math resolva incompatibilidades de tipo sozinho  
- Não restringir a constraint à interface mais específica necessária (usando `INumber<T>` completo quando `IAdditionOperators<T,T,T>` já resolveria)  

---

# 📌 Conclusão

- `INumber<T>` unifica todos os tipos numéricos do .NET sob uma única abstração genérica  
- Generic Math elimina a duplicação histórica de métodos matemáticos por tipo numérico  
- `T.Zero`, `T.CreateChecked` e `T.MinValue`/`MaxValue` vêm das interfaces de `System.Numerics`  
- Interfaces mais granulares (`IAdditionOperators`, `IComparisonOperators`) permitem constraints mais precisas  

👉 Com Generic Math dominado, você conclui o bloco sobre tipos numéricos — o próximo passo é olhar para um recurso completamente diferente: pattern matching avançado, que torna decisões condicionais complexas muito mais expressivas

---

# 🔥 Próximo passo

Agora que você escreve matemática genérica de verdade, o próximo nível é:

👉 **Fundamentos do C#: Pattern Matching Avançado**

Aqui você vai aprender padrões de comparação (`and`, `or`, `not`), pattern matching em listas, e outras evoluções do recurso que você já usa desde os primeiros posts sobre `switch`.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
