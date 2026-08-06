# 🧠 Fundamentos do C#: Expression Trees

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Covariância e contravariância em generics  
- LINQ (post 19) e Entity Framework Core (post 32)  

Já se perguntou como o Entity Framework transforma `.Where(p => p.Valor > 100)` em uma cláusula `WHERE Valor > 100` no SQL? Um lambda comum vira apenas código compilado — Expression Trees permitem que o código vire **dados** que podem ser inspecionados e traduzidos.

👉 **Vamos aprender Expression Trees**

---

# 💡 Func vs Expression: a diferença crucial

```csharp
Func<Pedido, bool> comoFuncao = p => p.Valor > 100;
bool resultado = comoFuncao(pedido); // executa o código imediatamente

Expression<Func<Pedido, bool>> comoExpressao = p => p.Valor > 100;
// isso NÃO executa nada — é a REPRESENTAÇÃO da lógica, como uma árvore de dados
```

👉 `Func<>` compila o lambda para IL executável. `Expression<Func<>>` compila o lambda para uma **árvore sintática** que descreve a lógica — o mesmo tipo de estrutura que você viu no post sobre Roslyn Analyzers (62), mas para expressões C# comuns

---

# 🏗️ Inspecionando uma Expression Tree

```csharp
Expression<Func<Pedido, bool>> expressao = p => p.Valor > 100;

var corpo = (BinaryExpression)expressao.Body;
var propriedade = (MemberExpression)corpo.Left;
var valor = (ConstantExpression)corpo.Right;

Console.WriteLine(corpo.NodeType);       // GreaterThan
Console.WriteLine(propriedade.Member.Name); // "Valor"
Console.WriteLine(valor.Value);           // 100
```

👉 Em vez de executar `p.Valor > 100`, você está **lendo a estrutura** dessa comparação — qual propriedade, qual operador, qual valor. É exatamente isso que o EF Core faz para gerar SQL

---

# 🎯 Como o Entity Framework usa isso de verdade

```csharp
var pedidosCaros = _contexto.Pedidos
    .Where(p => p.Valor > 100)  // isso é um Expression<Func<Pedido, bool>>
    .ToList();
```

```sql
-- O EF Core traduz a árvore de expressão para SQL real
SELECT * FROM Pedidos WHERE Valor > 100
```

👉 Lembra do post 32? O `DbSet<T>` do EF Core implementa `IQueryable<T>`, que usa `Expression<Func<>>` em vez de `Func<>` puro — por isso o EF consegue "ler" seu LINQ e traduzir para SQL, em vez de baixar todos os dados e filtrar em memória

---

# 🔨 Construindo Expression Trees dinamicamente

```csharp
ParameterExpression parametro = Expression.Parameter(typeof(Pedido), "p");
MemberExpression propriedade = Expression.Property(parametro, "Valor");
ConstantExpression constante = Expression.Constant(100m);
BinaryExpression comparacao = Expression.GreaterThan(propriedade, constante);

var lambda = Expression.Lambda<Func<Pedido, bool>>(comparacao, parametro);
var filtro = lambda.Compile(); // agora vira uma Func<> executável

bool resultado = filtro(pedido); // true, se pedido.Valor > 100
```

👉 Isso permite construir filtros dinamicamente em tempo de execução — útil para sistemas de busca avançada, onde o usuário escolhe quais campos e operadores filtrar, sem você precisar escrever um `if` para cada combinação possível

---

# ⚠️ Erros comuns

- Usar `Expression<Func<>>` quando só `Func<>` é necessário, adicionando complexidade sem motivo (Expression Trees só fazem sentido quando algo precisa **ler** a lógica, como o EF Core)  
- Chamar métodos C# arbitrários dentro de uma expressão que será traduzida para SQL, gerando erro em runtime porque o provider não sabe traduzir aquele método  
- Construir árvores de expressão manualmente quando um simples lambda resolveria o problema  
- Esquecer que compilar uma Expression Tree (`.Compile()`) tem custo — cachear o resultado quando usado repetidamente  

---

# 📌 Conclusão

- `Expression<Func<>>` representa código como uma árvore de dados inspecionável, em vez de código executável direto  
- Isso é a base de como o Entity Framework traduz LINQ para SQL  
- Expression Trees podem ser construídas dinamicamente para gerar lógica em tempo de execução  
- O custo de complexidade só vale a pena quando algo precisa **ler**, não só executar, sua lógica  

👉 Com Expression Trees, você entende a mágica por trás do LINQ to SQL — código C# que vira consultas de banco, sem uma linha de SQL escrita à mão

---

# 🔥 Próximo passo

Agora que você entende como código vira dados, o próximo nível é:

👉 **Fundamentos do C#: Nullable Reference Types em Profundidade**

Aqui você vai aprofundar no sistema de tipos que você já usa desde o início desta trilha, entendendo suas nuances mais avançadas.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
