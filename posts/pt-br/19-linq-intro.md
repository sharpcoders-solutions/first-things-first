# 🧠 Fundamentos do C#: Introdução ao LINQ

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Métodos, parâmetros e retorno  
- Coleções: arrays, listas e dicionários  

Você já sabe guardar grupos de dados. Mas filtrar, ordenar e transformar essas coleções com loops manuais fica repetitivo rápido.

👉 **É aí que entra o LINQ**

---

# 💡 O que é LINQ?

👉 **LINQ = Language Integrated Query**

É um conjunto de recursos do C# que permite **consultar coleções** de forma declarativa, parecida com SQL — mas dentro do próprio código.

```csharp
List<int> numeros = new List<int> { 1, 2, 3, 4, 5, 6 };

var pares = numeros.Where(n => n % 2 == 0);
```

👉 Em vez de escrever um `foreach` com `if` dentro, você descreve **o que** quer, não **como** obter

---

# 🔎 `Where`: filtrando dados

```csharp
var maiores = numeros.Where(n => n > 3);
// resultado: 4, 5, 6
```

👉 `Where` recebe uma condição (lambda) e retorna apenas os elementos que passam nela

---

# 🔄 `Select`: transformando dados

```csharp
var dobrados = numeros.Select(n => n * 2);
// resultado: 2, 4, 6, 8, 10, 12
```

👉 `Select` transforma cada elemento em outra coisa — igual ao `map` de outras linguagens

## 🔹 Combinando `Where` e `Select`

```csharp
var resultado = numeros
    .Where(n => n % 2 == 0)
    .Select(n => n * 10);
// resultado: 20, 40, 60
```

👉 Métodos LINQ podem ser **encadeados**, formando um pipeline de transformações

---

# 📊 Ordenando e buscando

```csharp
var ordenado = numeros.OrderBy(n => n);          // crescente
var decrescente = numeros.OrderByDescending(n => n);

int primeiro = numeros.First();                   // lança exceção se vazio
int? primeiroOuNulo = numeros.FirstOrDefault();    // retorna 0 (ou null) se vazio

bool existePar = numeros.Any(n => n % 2 == 0);
int totalPares = numeros.Count(n => n % 2 == 0);
int soma = numeros.Sum();
```

## 🔹 Métodos mais usados

- `OrderBy` / `OrderByDescending` → ordena  
- `First` / `FirstOrDefault` → primeiro elemento (com ou sem exceção)  
- `Any` → existe pelo menos um que satisfaz a condição?  
- `Count` → quantidade de elementos (com ou sem filtro)  
- `Sum`, `Average`, `Max`, `Min` → agregações  

👉 Prefira `FirstOrDefault` quando não tiver certeza de que existe um resultado

---

# ✍️ Method syntax vs Query syntax

O LINQ tem duas formas de escrever a mesma consulta:

## 🔹 Method syntax (mais usada no dia a dia)

```csharp
var pares = numeros.Where(n => n % 2 == 0).OrderBy(n => n);
```

## 🔹 Query syntax (parecida com SQL)

```csharp
var pares = from n in numeros
            where n % 2 == 0
            orderby n
            select n;
```

👉 As duas geram o mesmo resultado — method syntax é mais comum no mercado e mais fácil de encadear

---

# ⏳ Execução adiada (deferred execution)

Um detalhe importante: consultas LINQ **não executam na hora em que são escritas**.

```csharp
var consulta = numeros.Where(n => n > 2); // ainda não executou

numeros.Add(10);

foreach (var n in consulta) // executa agora, já considerando o 10
{
    Console.WriteLine(n);
}
```

👉 A consulta só roda quando você **itera** sobre o resultado (`foreach`, `ToList()`, etc.)

Para forçar a execução imediata:

```csharp
var listaFixa = numeros.Where(n => n > 2).ToList();
```

---

# ⚠️ Erros comuns

- Usar `First()` sem garantir que existe pelo menos um elemento  
- Achar que a consulta LINQ já executou no momento em que foi declarada  
- Encadear métodos demais em uma única linha, prejudicando a leitura  
- Rodar `Count()` múltiplas vezes em vez de guardar o resultado em uma variável  

---

# 📌 Conclusão

- LINQ permite consultar coleções de forma declarativa  
- `Where`, `Select`, `OrderBy` e agregações cobrem a maioria dos casos do dia a dia  
- Method syntax é a forma mais usada no mercado  
- A execução é adiada até a coleção ser iterada  

👉 Com LINQ, manipular coleções fica mais expressivo e muito mais legível

---

# 🔥 Próximo passo

Agora que você sabe consultar e transformar coleções, o próximo nível é:

👉 **Fundamentos do C#: DateOnly e TimeOnly**

Aqui você vai conhecer dois tipos que resolvem um problema clássico do `DateTime`: representar só a data, ou só o horário, sem carregar informação que não faz sentido no seu domínio.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
