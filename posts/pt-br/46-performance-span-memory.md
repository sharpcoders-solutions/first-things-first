# 🧠 Fundamentos do C#: Performance em C# (Span, Memory e Benchmarking)

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- gRPC para comunicação eficiente entre serviços  
- Cache, resiliência e mensageria para escalar sua aplicação  

Você já otimizou a arquitetura. Agora vamos descer um nível: como escrever código C# que gasta menos memória e menos tempo de CPU, no nível mais baixo da linguagem.

👉 **Vamos falar sobre performance de verdade, com medição, não achismo**

---

# 💡 A regra de ouro: meça antes de otimizar

👉 **"Premature optimization is the root of all evil" — otimizar sem medir é chute**

Antes de qualquer mudança de performance, você precisa de dados. É aí que entra o **BenchmarkDotNet**:

```bash
dotnet add package BenchmarkDotNet
```

```csharp
[MemoryDiagnoser]
public class ConcatenacaoBenchmark
{
    [Benchmark]
    public string ComStringBuilder()
    {
        var sb = new StringBuilder();
        for (int i = 0; i < 1000; i++)
            sb.Append(i);
        return sb.ToString();
    }

    [Benchmark]
    public string ComConcatenacao()
    {
        string resultado = "";
        for (int i = 0; i < 1000; i++)
            resultado += i;
        return resultado;
    }
}
```

```csharp
BenchmarkRunner.Run<ConcatenacaoBenchmark>();
```

👉 O resultado mostra tempo de execução **e** memória alocada por método — muitas vezes a diferença de alocação de memória importa mais que a diferença de velocidade bruta, porque menos alocação significa menos trabalho para o Garbage Collector (lembra dele, do post sobre arquitetura .NET?)

---

# 🧵 O problema das alocações desnecessárias

```csharp
string texto = "João,25,São Paulo";
string[] partes = texto.Split(','); // aloca um novo array + três novas strings
```

👉 Toda vez que você fatia uma string com `Split` ou `Substring`, o CLR aloca **memória nova** — em código que roda milhões de vezes (um parser, um processamento de log), isso se acumula rápido e gera pressão no Garbage Collector

---

# 🔪 `Span<T>`: fatiando sem alocar

```csharp
ReadOnlySpan<char> texto = "João,25,São Paulo".AsSpan();

int primeiraVirgula = texto.IndexOf(',');
ReadOnlySpan<char> nome = texto.Slice(0, primeiraVirgula);

Console.WriteLine(nome.ToString()); // João
```

👉 **`Span<T>` = uma "janela" para um trecho de memória já existente, sem copiar dados**

`Slice` não cria uma nova string — ele aponta para a mesma memória, só que delimitando um pedaço menor. Isso é diferente de `Substring`, que sempre aloca uma cópia nova

## 🔹 Onde `Span<T>` brilha

- Parsing de texto (arquivos, protocolos, logs)  
- Manipulação de arrays sem criar cópias intermediárias  
- Código de alta performance, como bibliotecas de serialização  

👉 Para o C# do dia a dia (controllers, services de negócio), essa otimização raramente importa — o ganho real aparece em código executado em loops apertados, com alto volume de dados

---

# 🧠 `Memory<T>`: `Span<T>` para cenários assíncronos

```csharp
public async Task ProcessarAsync(Memory<byte> dados)
{
    await Task.Delay(100); // Span<T> não pode atravessar um await; Memory<T> pode
    Processar(dados.Span);
}
```

👉 `Span<T>` só pode viver na stack (é um `ref struct`), o que significa que **não pode** ser usado dentro de métodos `async` (lembra do post sobre async/await? O estado de um método assíncrono pode viver na heap entre awaits). `Memory<T>` resolve isso, funcionando como uma versão de `Span<T>` que pode atravessar operações assíncronas

---

# 📊 `struct` vs `class`: o impacto na alocação

```csharp
public struct PontoStruct // tipo por valor — vive na stack quando possível
{
    public int X;
    public int Y;
}

public class PontoClasse // tipo por referência — sempre alocado na heap
{
    public int X;
    public int Y;
}
```

👉 Lembra da diferença entre tipos por valor e por referência do post sobre variáveis? Isso tem impacto direto em performance: `struct`s pequenas, usadas em grande volume (ex: milhões de coordenadas), evitam pressão no Garbage Collector por não precisarem de alocação na heap

**Regra prática:** use `struct` para dados pequenos, imutáveis, usados em grande quantidade. Para a maioria das entidades de negócio, `class` continua sendo a escolha certa.

---

# ⚙️ `StringBuilder`: evitando concatenação repetida

```csharp
// ❌ Cada += cria uma nova string, descartando a anterior
string resultado = "";
foreach (var item in itens)
    resultado += item + ", ";

// ✅ StringBuilder reutiliza o mesmo buffer internamente
var sb = new StringBuilder();
foreach (var item in itens)
    sb.Append(item).Append(", ");
string resultado = sb.ToString();
```

👉 Strings em C# são **imutáveis** (você viu isso desde o post sobre variáveis) — toda concatenação com `+=` cria uma string totalmente nova. Em loops com muitas iterações, isso vira `O(n²)` de trabalho desperdiçado

---

# ⚠️ Erros comuns

- Otimizar código sem medir antes com um benchmark, baseado só em intuição  
- Usar `Span<T>` em código simples, comum, onde a complexidade extra não traz ganho perceptível  
- Concatenar strings com `+=` dentro de loops grandes em vez de `StringBuilder`  
- Transformar toda entidade de negócio em `struct` "para performance", sem entender que isso muda a semântica de cópia de valor vs referência  

---

# 📌 Conclusão

- Meça com BenchmarkDotNet antes de otimizar — dados vencem intuição  
- `Span<T>` evita alocações desnecessárias ao fatiar memória existente  
- `Memory<T>` é a versão de `Span<T>` compatível com código `async`  
- `struct` reduz pressão no Garbage Collector para dados pequenos e numerosos  
- `StringBuilder` evita o custo de concatenações repetidas em strings imutáveis  

👉 Performance em C# não é sobre truques obscuros — é sobre entender como a linguagem aloca memória, e medir antes de agir

---

# 🔥 Próximo passo

Agora que você sabe otimizar no nível mais baixo, o próximo nível é:

👉 **Fundamentos do C#: Segurança Avançada em APIs (OWASP Top 10 na Prática)**

Aqui você vai aprender a proteger sua aplicação contra as vulnerabilidades mais comuns e mais exploradas do mundo real.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
