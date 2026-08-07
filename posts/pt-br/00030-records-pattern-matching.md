# 🧠 Fundamentos do C#: Records e Pattern Matching (C# Moderno)

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Classes, objetos e os pilares da OOP  
- Async/await na prática  

Agora vamos conhecer dois recursos que mudaram a forma como se escreve C# moderno: menos código repetitivo, mais expressividade.

👉 **Records e Pattern Matching**

---

# 💡 O que é um record?

👉 **Record = um tipo pensado para representar dados, com igualdade por valor**

```csharp
record Produto(string Nome, decimal Preco);
```

Essa única linha já gera automaticamente:

- Propriedades `Nome` e `Preco`  
- Construtor que recebe os dois valores  
- `ToString()` legível  
- Igualdade por **valor**, não por referência  

```csharp
var p1 = new Produto("Notebook", 3500);
var p2 = new Produto("Notebook", 3500);

Console.WriteLine(p1 == p2); // true — mesmo sendo objetos diferentes na memória
```

---

# 🔀 Record vs Classe: a diferença que importa

```csharp
class ProdutoClasse
{
    public string Nome { get; set; }
    public decimal Preco { get; set; }
}

var c1 = new ProdutoClasse { Nome = "Notebook", Preco = 3500 };
var c2 = new ProdutoClasse { Nome = "Notebook", Preco = 3500 };

Console.WriteLine(c1 == c2); // false — compara referência, não valor
```

👉 Com `class`, `==` compara se são **o mesmo objeto** na memória. Com `record`, `==` compara se os **valores** são iguais

**Regra prática:** use `record` quando o objeto representa **dados** (um DTO, uma resposta de API, um valor imutável). Use `class` quando o objeto representa uma **entidade com identidade e comportamento**, como você viu nos posts sobre OOP.

---

# 🧊 Imutabilidade e o `with`

Por padrão, propriedades de um record são `init` — só podem ser definidas na criação:

```csharp
record Produto(string Nome, decimal Preco);

var original = new Produto("Notebook", 3500);
// original.Preco = 4000; // ❌ erro de compilação
```

Para "alterar" um record, você cria uma **cópia** com valores diferentes:

```csharp
var comDesconto = original with { Preco = 2999 };

Console.WriteLine(original.Preco);     // 3500 — original intacto
Console.WriteLine(comDesconto.Preco);  // 2999 — nova instância
```

👉 O `with` cria uma nova instância copiando os valores existentes, mudando só o que você especificar — isso é chamado de **mutação não destrutiva**

---

# 🧱 `record class` vs `record struct`

```csharp
record class Produto(string Nome, decimal Preco);   // referência (padrão)
record struct Ponto(int X, int Y);                   // valor
```

👉 Por padrão, `record` é um tipo referência. Desde o C# 10, `record struct` permite ter os mesmos benefícios (igualdade por valor, `with`) em um tipo valor, mais leve para casos como coordenadas ou pequenas estruturas de dados

---

# 📦 Desconstrução

```csharp
var produto = new Produto("Notebook", 3500);
var (nome, preco) = produto;

Console.WriteLine(nome);  // Notebook
Console.WriteLine(preco); // 3500
```

👉 Records geram automaticamente um método de desconstrução, permitindo "desempacotar" os valores direto em variáveis

---

# 🔍 Pattern Matching: além do `switch` básico

Você já viu `switch expressions` no post sobre estruturas de controle. Pattern matching vai além, permitindo verificar **tipo, propriedades e valores** ao mesmo tempo.

## 🔹 Type pattern

```csharp
object valor = 42;

if (valor is int numero)
{
    Console.WriteLine($"É um inteiro: {numero}");
}
```

## 🔹 Property pattern

```csharp
Produto produto = new Produto("Notebook", 3500);

string categoria = produto switch
{
    { Preco: > 3000 } => "Premium",
    { Preco: > 1000 } => "Intermediário",
    _ => "Econômico"
};
```

👉 O `switch` está avaliando diretamente a **propriedade** `Preco` do objeto, sem precisar de `produto.Preco` explícito em cada `case`

## 🔹 Relational e logical patterns

```csharp
string ClassificarIdade(int idade) => idade switch
{
    < 12 => "Criança",
    >= 12 and < 18 => "Adolescente",
    >= 18 => "Adulto"
};
```

👉 `and`, `or` e `not` combinam condições diretamente dentro do pattern, sem precisar de `if`s aninhados

---

# 🏗️ Exemplo real: combinando records e pattern matching

```csharp
record Pedido(string Cliente, decimal Total, string Status);

string Descrever(Pedido pedido) => pedido switch
{
    { Status: "Cancelado" } => $"Pedido de {pedido.Cliente} foi cancelado",
    { Total: > 1000, Status: "Pendente" } => $"Pedido grande de {pedido.Cliente} aguardando pagamento",
    { Status: "Entregue" } => $"Pedido de {pedido.Cliente} já foi entregue",
    _ => $"Pedido de {pedido.Cliente} em andamento"
};

var pedido = new Pedido("Maria", 1500, "Pendente");
Console.WriteLine(Descrever(pedido));
// Pedido grande de Maria aguardando pagamento
```

👉 Records deixam o modelo de dados enxuto, e pattern matching deixa a lógica de decisão declarativa e fácil de ler

---

# ⚠️ Erros comuns

- Usar `record` para entidades que precisam de identidade e comportamento mutável (isso é papel de `class`)  
- Esquecer que `with` cria uma **cópia**, não altera o original  
- Empilhar `if`s aninhados quando um `switch` com property pattern resolveria de forma mais limpa  
- Achar que `record` substitui `class` completamente — eles resolvem problemas diferentes  

---

# 📌 Conclusão

- `record` é ideal para representar dados, com igualdade por valor e imutabilidade por padrão  
- `with` permite criar cópias modificadas sem alterar o objeto original  
- `record struct` traz os benefícios de record para tipos valor  
- Pattern matching (type, property, relational, logical) torna decisões complexas mais declarativas  

👉 Com records e pattern matching, seu C# fica mais moderno, conciso e expressivo

---

# 🔥 Próximo passo

Agora que você conhece os recursos modernos da linguagem, o próximo nível é:

👉 **Fundamentos do C#: Princípios SOLID (Introdução ao Design de Software)**

Aqui você começa a sair da sintaxe da linguagem e entra no mundo de arquitetura e design de software profissional.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
