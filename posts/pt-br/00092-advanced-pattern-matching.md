# 🧠 Fundamentos do C#: Pattern Matching Avançado

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Generic Math e como unificar operações sobre qualquer tipo numérico  
- `switch` expressions e pattern matching básico, desde os primeiros posts sobre estruturas de controle  

Você já usa `is` e `switch` com padrões simples há muitos posts. C# foi expandindo pattern matching a cada versão, e hoje ele cobre casos muito mais ricos que "esse objeto é desse tipo?". Chegou a hora de ver o conjunto completo.

👉 **Vamos explorar pattern matching avançado**

---

# 💡 Revisão rápida: de onde viemos

```csharp
// Pattern matching básico, que você já usa
if (objeto is Produto produto && produto.Preco > 0)
{
    Console.WriteLine(produto.Nome);
}
```

👉 Isso combina um teste de tipo com uma captura de variável — o ponto de partida sobre o qual todos os padrões mais avançados se constroem

---

# 🔗 Padrões relacionais: comparações diretas no `switch`

```csharp
string ClassificarIdade(int idade) => idade switch
{
    < 0 => "Inválida",
    < 13 => "Criança",
    < 18 => "Adolescente",
    < 65 => "Adulto",
    _ => "Idoso"
};
```

👉 Padrões relacionais (`<`, `>`, `<=`, `>=`) dentro de um `switch` eliminam cadeias de `if/else if` — cada `case` já é uma comparação direta, avaliada em ordem

---

# 🔀 Combinando padrões com `and`, `or`, `not`

```csharp
string AvaliarNota(int nota) => nota switch
{
    >= 90 and <= 100 => "Excelente",
    >= 70 and < 90 => "Bom",
    >= 50 and < 70 => "Regular",
    not (>= 0 and <= 100) => "Nota inválida",
    _ => "Insuficiente"
};
```

👉 `and`, `or` e `not` combinam padrões da mesma forma que operadores lógicos combinam expressões booleanas — `>= 90 and <= 100` é muito mais direto que escrever `nota >= 90 && nota <= 100` fora de um `switch`

---

# 🧩 Padrões de propriedade: inspecionando campos de um objeto

```csharp
public record Pedido(decimal Valor, string Status, bool ClienteVip);

string CalcularDesconto(Pedido pedido) => pedido switch
{
    { ClienteVip: true, Valor: > 1000 } => "20% de desconto",
    { ClienteVip: true } => "10% de desconto",
    { Valor: > 500 } => "5% de desconto",
    _ => "Sem desconto"
};
```

👉 **Padrão de propriedade = testar valores de propriedades específicas de um objeto, diretamente dentro do padrão, sem precisar acessar `pedido.ClienteVip` e `pedido.Valor` manualmente**

Lembra dos `record`s (post 30)? Padrões de propriedade combinam perfeitamente com eles — records são frequentemente usados exatamente para modelar os dados que esses padrões inspecionam

---

# 📋 Padrões de lista: inspecionando arrays e coleções

```csharp
int[] numeros = { 1, 2, 3 };

string DescreverSequencia(int[] valores) => valores switch
{
    [] => "Vazio",
    [var unico] => $"Um elemento: {unico}",
    [var primeiro, var segundo] => $"Dois elementos: {primeiro} e {segundo}",
    [var primeiro, .., var ultimo] => $"Começa com {primeiro}, termina com {ultimo}",
};

Console.WriteLine(DescreverSequencia(numeros)); // Começa com 1, termina com 3
```

👉 **Padrões de lista = testar a forma (quantidade e posição de elementos) de um array ou lista diretamente no `switch`**

O `..` (slice pattern) captura "o resto" da sequência, permitindo testar só o início, só o fim, ou ambos, sem precisar escrever lógica manual de índices

---

# 🎯 Padrões de tupla: múltiplos valores de uma vez

```csharp
(int X, int Y) ponto = (0, 5);

string DescreverPosicao(int x, int y) => (x, y) switch
{
    (0, 0) => "Origem",
    (0, _) => "No eixo Y",
    (_, 0) => "No eixo X",
    (var px, var py) when px == py => "Na diagonal",
    _ => "Posição qualquer"
};
```

👉 Lembra das tuplas (post 51)? Combinadas com `switch`, elas permitem testar múltiplos valores relacionados em um único padrão — incluindo cláusulas `when` para condições que não se encaixam em um padrão estrutural simples

---

# 🌳 Padrões aninhados: combinando tudo

```csharp
public record Endereco(string Cidade, string Estado);
public record Cliente(string Nome, Endereco Endereco, bool Ativo);

string ClassificarCliente(Cliente cliente) => cliente switch
{
    { Ativo: true, Endereco: { Estado: "SP" } } => "Cliente ativo em São Paulo",
    { Ativo: true, Endereco: { Cidade: var cidade } } => $"Cliente ativo em {cidade}",
    { Ativo: false } => "Cliente inativo",
};
```

👉 Padrões de propriedade podem ser aninhados indefinidamente — inspecionando propriedades de propriedades, combinando com padrões relacionais, de lista e de tupla, tudo dentro de um único `switch` expressivo e legível

---

# ⚠️ Erros comuns

- Empilhar tantos padrões aninhados que o `switch` fica mais difícil de ler que a cadeia de `if/else` que ele deveria substituir  
- Esquecer o caso `_` (default) em um `switch expression`, causando `MatchFailureException` em tempo de execução para valores não cobertos  
- Usar `and`/`or` sem parênteses em combinações complexas, criando ambiguidade sobre a ordem de avaliação  
- Duplicar lógica de negócio significativa dentro de um padrão, quando ela deveria estar em um método nomeado e testável separadamente  

---

# 📌 Conclusão

- Padrões relacionais (`<`, `>`) eliminam cadeias de `if/else if` dentro de um `switch`  
- `and`, `or`, `not` combinam padrões com a mesma clareza de operadores lógicos  
- Padrões de propriedade inspecionam campos de objetos diretamente, sem acesso manual  
- Padrões de lista testam a forma de arrays e coleções, incluindo slices com `..`  
- Padrões podem ser aninhados livremente, combinando todos os tipos anteriores  

👉 Com pattern matching avançado dominado, o próximo passo é ver como C# garante, em tempo de compilação, que propriedades obrigatórias sejam sempre inicializadas

---

# 🔥 Próximo passo

Agora que você domina pattern matching em profundidade, o próximo nível é:

👉 **Fundamentos do C#: Required Members e Init-Only Properties**

Aqui você vai aprender a forçar, em tempo de compilação, que certas propriedades sejam sempre preenchidas na criação de um objeto.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
