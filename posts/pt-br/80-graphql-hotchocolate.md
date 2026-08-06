# 🧠 Fundamentos do C#: GraphQL com HotChocolate

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Arquitetura orientada a eventos, conectando tudo sobre mensageria  
- APIs REST desde o post 31, com endpoints fixos por recurso  

Em REST, cada endpoint retorna uma estrutura fixa de dados — se o cliente precisa só do nome do cliente, mas o endpoint retorna o pedido inteiro, isso é desperdício de rede. GraphQL resolve isso deixando o cliente escolher exatamente o que quer.

👉 **Vamos aprender GraphQL com HotChocolate**

---

# 💡 REST vs GraphQL

## 🔹 REST (post 31)

```
GET /pedidos/123
```

```json
{
  "id": 123,
  "valor": 150.00,
  "status": "Confirmado",
  "cliente": { "id": 1, "nome": "Maria", "email": "maria@email.com", "endereco": {...} },
  "itens": [...]
}
```

👉 O cliente recebe **tudo**, mesmo que só precise do `status`

## 🔹 GraphQL

```graphql
query {
  pedido(id: 123) {
    status
  }
}
```

```json
{ "pedido": { "status": "Confirmado" } }
```

👉 O cliente pede exatamente os campos que precisa — nada a mais, nada a menos

---

# 🏗️ Configurando o HotChocolate

```bash
dotnet add package HotChocolate.AspNetCore
```

```csharp
public class Query
{
    public async Task<Pedido?> ObterPedido(int id, [Service] AppDbContext contexto) =>
        await contexto.Pedidos.Include(p => p.Cliente).FirstOrDefaultAsync(p => p.Id == id);

    public async Task<List<Pedido>> ListarPedidos([Service] AppDbContext contexto) =>
        await contexto.Pedidos.ToListAsync();
}
```

```csharp
// Program.cs
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>();

// ...

app.MapGraphQL();
```

👉 A classe `Query` define os pontos de entrada de leitura — parecido com um controller (post 31), mas o HotChocolate constrói o schema GraphQL automaticamente a partir dela

---

# 🎯 Resolvendo relacionamentos sob demanda

```csharp
public class Pedido
{
    public int Id { get; set; }
    public decimal Valor { get; set; }
    public string Status { get; set; } = default!;

    [GraphQLIgnore]
    public int ClienteId { get; set; }

    public async Task<Cliente> ObterCliente([Service] AppDbContext contexto) =>
        await contexto.Clientes.FindAsync(ClienteId);
}
```

👉 O campo `Cliente` só é resolvido (executando a query no banco) se o cliente da API realmente pedir por ele na consulta GraphQL — diferente do REST, onde o `Include` do EF Core (post 32) sempre traz os dados relacionados, seja usado ou não

---

# ✍️ Mutations: escrevendo dados

```csharp
public class Mutation
{
    public async Task<Pedido> CriarPedido(CriarPedidoInput input, [Service] AppDbContext contexto)
    {
        var pedido = new Pedido { Valor = input.Valor, ClienteId = input.ClienteId };
        contexto.Pedidos.Add(pedido);
        await contexto.SaveChangesAsync();
        return pedido;
    }
}
```

```graphql
mutation {
  criarPedido(input: { valor: 150.00, clienteId: 1 }) {
    id
    status
  }
}
```

👉 Igual às queries, uma mutation também define exatamente quais campos retornar depois da escrita — sem precisar de um segundo `GET` para buscar o recurso recém-criado, como costuma acontecer em REST

---

# 🔍 Um schema fortemente tipado

```graphql
type Pedido {
  id: Int!
  valor: Decimal!
  status: String!
  cliente: Cliente!
}
```

👉 Lembra do post sobre OpenAPI/Swagger (49)? GraphQL exige um schema fortemente tipado desde o início — a documentação e a validação de tipos não são um complemento adicionado depois, são parte estrutural do próprio GraphQL

---

# ⚠️ Erros comuns

- Usar GraphQL para APIs simples com poucos relacionamentos, onde REST já resolveria com menos complexidade  
- Não implementar DataLoader para evitar o problema N+1 em relacionamentos aninhados (buscar o cliente de cada pedido individualmente, em vez de em lote)  
- Expor toda a estrutura interna do banco diretamente no schema GraphQL, sem uma camada de abstração  
- Esquecer de aplicar limites de profundidade/complexidade de query, permitindo consultas maliciosamente aninhadas que sobrecarregam o servidor  

---

# 📌 Conclusão

- GraphQL deixa o cliente escolher exatamente quais campos precisa, evitando over-fetching  
- HotChocolate constrói o schema a partir de classes `Query` e `Mutation`, parecido com controllers REST  
- Campos relacionados só são resolvidos quando realmente solicitados  
- O schema fortemente tipado é parte estrutural do GraphQL, não um complemento  

👉 Com GraphQL, APIs ganham flexibilidade de consulta que o REST tradicional não oferece nativamente — o cliente decide, a cada requisição, exatamente o que precisa

---

# 🔥 Próximo passo

Agora que você conhece uma alternativa ao REST, o próximo nível é:

👉 **Fundamentos do C#: SignalR**

Aqui você vai aprender comunicação em tempo real entre servidor e cliente, para além do modelo request/response tradicional.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
