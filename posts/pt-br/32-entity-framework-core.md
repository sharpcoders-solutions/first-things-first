# 🧠 Fundamentos do C#: Entity Framework Core — Persistindo Dados de Verdade

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como criar uma API com ASP.NET Core  
- Repository pattern e injeção de dependência  

Sua API já funciona, mas os dados somem toda vez que você reinicia a aplicação — porque estão só em uma lista na memória. Chegou a hora de resolver isso de verdade.

👉 **Vamos trocar a memória por um banco de dados real, usando Entity Framework Core**

---

# 💡 O que é o Entity Framework Core?

👉 **EF Core = um ORM (Object-Relational Mapper) que traduz suas classes C# em tabelas de banco de dados, e vice-versa**

Em vez de escrever SQL manualmente, você trabalha com objetos e coleções — o EF Core cuida da tradução:

```csharp
var produtos = contexto.Produtos.Where(p => p.Preco > 100).ToList();
```

👉 Essa linha é pura sintaxe LINQ, do post que você já viu — só que agora, por baixo dos panos, o EF Core a transforma em uma consulta SQL de verdade

---

# 🏗️ Instalando os pacotes

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
```

👉 O primeiro pacote é o **provider** (aqui, SQL Server — poderia ser PostgreSQL, SQLite, MySQL...). O segundo habilita o suporte a migrations pela linha de comando

---

# 🧱 Definindo o `DbContext`

```csharp
public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public decimal Preco { get; set; }
}

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Produto> Produtos { get; set; }
}
```

## 🔹 As duas peças principais

- `DbContext` → representa a sessão com o banco de dados  
- `DbSet<T>` → representa uma tabela, com todas as operações de consulta que você já conhece do LINQ  

👉 Note o construtor recebendo `DbContextOptions` — isso é injeção de dependência de novo, exatamente como você viu no post sobre ASP.NET Core

---

# 🔌 Registrando o `DbContext` no ASP.NET Core

```csharp
// Program.cs
var connectionString = builder.Configuration.GetConnectionString("Padrao");

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

```json
// appsettings.json
{
  "ConnectionStrings": {
    "Padrao": "Server=localhost;Database=MinhaApiDb;Trusted_Connection=True;"
  }
}
```

👉 `AddDbContext` registra o contexto no container de DI com tempo de vida `Scoped` por padrão — uma instância nova por requisição HTTP, evitando que dados de um usuário vazem para outro

---

# 🚀 Migrations: versionando o esquema do banco

```bash
dotnet ef migrations add CriacaoInicial
dotnet ef database update
```

## 🔹 O que cada comando faz

- `migrations add` → gera um arquivo de código C# descrevendo as mudanças no esquema  
- `database update` → aplica essas mudanças no banco de dados de verdade  

👉 Migrations funcionam como um "Git para o banco de dados" — cada mudança no modelo vira um registro versionado e reproduzível, aplicável em qualquer ambiente

---

# ✍️ Operações CRUD com EF Core

```csharp
// Create
contexto.Produtos.Add(new Produto { Nome = "Notebook", Preco = 3500 });
await contexto.SaveChangesAsync();

// Read
var produto = await contexto.Produtos.FirstOrDefaultAsync(p => p.Id == 1);
var todos = await contexto.Produtos.ToListAsync();

// Update
produto.Preco = 2999;
await contexto.SaveChangesAsync();

// Delete
contexto.Produtos.Remove(produto);
await contexto.SaveChangesAsync();
```

👉 Repare o `async`/`await` em toda operação de banco — exatamente o que você aprendeu no post sobre programação assíncrona. Acesso a banco de dados é I/O, e bloquear a thread esperando o SQL Server responder é desperdício de recursos

---

# 🗄️ Implementando o Repository com EF Core

Lembra do `IRepositorio<T>` do post sobre design patterns? Agora ele ganha uma implementação de verdade:

```csharp
public class RepositorioEfCore<T> : IRepositorio<T> where T : class
{
    private readonly AppDbContext _contexto;

    public RepositorioEfCore(AppDbContext contexto)
    {
        _contexto = contexto;
    }

    public void Adicionar(T item) => _contexto.Set<T>().Add(item);
    public T ObterPorId(int id) => _contexto.Set<T>().Find(id);
    public IEnumerable<T> ListarTodos() => _contexto.Set<T>().ToList();
}
```

```csharp
// Program.cs — troca de uma linha, nada mais muda
builder.Services.AddScoped<IRepositorio<Produto>, RepositorioEfCore<Produto>>();
// antes era: AddSingleton<IRepositorio<Produto>, RepositorioEmMemoria<Produto>>();
```

👉 O `ProdutosController` do post anterior **não muda uma linha sequer** — ele depende da interface `IRepositorio<T>`, nunca da implementação concreta. Essa é a prova definitiva de que o Dependency Inversion Principle não é teoria: é o que te permite trocar "memória" por "banco de dados real" sem tocar no resto do sistema

---

# ⚠️ Erros comuns

- Registrar o `DbContext` como `Singleton` em vez de `Scoped`, causando problemas de concorrência entre requisições  
- Usar métodos síncronos (`ToList()`, `SaveChanges()`) em vez das versões assíncronas em código de API  
- Esquecer de rodar `dotnet ef database update` depois de criar uma migration, e se perguntar por que a tabela não existe  
- Cair no problema de N+1 queries: buscar uma lista e depois fazer uma consulta separada para cada item, em vez de usar `Include()` para carregar os dados relacionados de uma vez  

---

# 📌 Conclusão

- EF Core traduz classes C# em tabelas de banco de dados, usando LINQ para consultas  
- `DbContext` representa a sessão; `DbSet<T>` representa cada tabela  
- Migrations versionam o esquema do banco de forma reproduzível  
- Toda operação de banco deveria ser `async`, seguindo o que você já aprendeu sobre I/O  
- Trocar o Repository em memória pelo baseado em EF Core não exige mudar o controller — DIP na prática  

👉 Com EF Core, sua API deixa de perder dados a cada reinício e passa a persistir informação de verdade, com todo o design que você construiu ao longo da trilha continuando intacto

---

# 🔥 Próximo passo

Agora que sua aplicação persiste dados de verdade, o próximo nível é:

👉 **Fundamentos do C#: Extension Methods e LINQ Customizado**

Aqui você vai entender o mecanismo por trás do LINQ que já usa desde o post 19, e aprender a criar seus próprios operadores encadeáveis.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
