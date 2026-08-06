# 🧠 Fundamentos do C#: Criando sua Primeira API com ASP.NET Core

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Toda a base da linguagem C#  
- SOLID, design patterns e testes automatizados  

Tudo isso te preparou para o momento mais aguardado da trilha:

👉 **Sair do código isolado e construir uma aplicação web real**

Hoje você cria sua primeira API com **ASP.NET Core**, o framework web do .NET usado pela imensa maioria das empresas que contratam desenvolvedores C#.

---

# 💡 O que é uma Web API?

👉 **API = uma aplicação que expõe funcionalidades através de endpoints HTTP, para que outros sistemas (front-ends, apps mobile, outros serviços) consumam**

O ASP.NET Core é o framework que faz o trabalho pesado: recebe requisições HTTP, roteia para o código certo, e devolve respostas — tudo isso construído sobre o CLR e o .NET que você já conhece dos primeiros posts da trilha.

---

# 🏗️ Criando o projeto

```bash
dotnet new webapi -o MinhaApi
cd MinhaApi
dotnet run
```

👉 O template `webapi` já vem com um endpoint de exemplo funcionando e o Swagger configurado para testar a API pelo navegador

---

# 🔀 Minimal APIs vs Controllers

Desde o .NET 6, existem duas formas principais de definir endpoints.

## 🔹 Minimal API (direto no `Program.cs`)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/produtos", () => new[] { "Notebook", "Mouse", "Teclado" });

app.Run();
```

👉 Ideal para APIs pequenas ou microsserviços — segue o mesmo espírito dos **top-level statements** que você viu no post sobre seu primeiro programa: menos cerimônia, direto ao ponto

## 🔹 Controllers (abordagem tradicional, ótima para APIs maiores)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    [HttpGet]
    public IActionResult ObterTodos()
    {
        var produtos = new[] { "Notebook", "Mouse", "Teclado" };
        return Ok(produtos);
    }
}
```

👉 Controllers organizam melhor APIs grandes, com muitos endpoints relacionados agrupados na mesma classe — é a abordagem mais comum em projetos corporativos

---

# 📦 Recebendo dados: model binding

```csharp
public record CriarProdutoRequest(string Nome, decimal Preco);

[HttpPost]
public IActionResult Criar([FromBody] CriarProdutoRequest request)
{
    // request.Nome e request.Preco já vêm preenchidos do corpo da requisição
    return Ok(request);
}

[HttpGet("{id}")]
public IActionResult ObterPorId([FromRoute] int id)
{
    return Ok($"Produto {id}");
}

[HttpGet("buscar")]
public IActionResult Buscar([FromQuery] string termo)
{
    return Ok($"Buscando: {termo}");
}
```

## 🔹 De onde vêm os dados

- `[FromBody]` → corpo da requisição (JSON), usado em POST/PUT  
- `[FromRoute]` → parte da URL (`/produtos/5`)  
- `[FromQuery]` → query string (`/produtos/buscar?termo=notebook`)  

👉 Repare que `CriarProdutoRequest` é um `record` — exatamente o recurso que você aprendeu no post sobre C# moderno, perfeito para representar dados de entrada imutáveis

---

# 🔌 Injeção de dependência embutida

O ASP.NET Core já vem com um container de injeção de dependência pronto — é a aplicação prática direta do que você viu no post sobre SOLID e Repository:

```csharp
// Program.cs
builder.Services.AddSingleton<IRepositorio<Produto>, RepositorioEmMemoria<Produto>>();
```

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    private readonly IRepositorio<Produto> _repositorio;

    public ProdutosController(IRepositorio<Produto> repositorio) // injetado automaticamente
    {
        _repositorio = repositorio;
    }

    [HttpGet]
    public IActionResult ObterTodos() => Ok(_repositorio.ListarTodos());
}
```

👉 O ASP.NET Core cria e entrega a instância de `IRepositorio<Produto>` automaticamente — você nunca escreve `new RepositorioEmMemoria<Produto>()` dentro do controller. Isso é Dependency Inversion Principle funcionando na prática, sem código manual

---

# 📊 Respostas HTTP corretas

```csharp
[HttpGet("{id}")]
public IActionResult ObterPorId(int id)
{
    var produto = _repositorio.ObterPorId(id);

    if (produto is null)
        return NotFound(); // 404

    return Ok(produto); // 200
}

[HttpPost]
public IActionResult Criar(CriarProdutoRequest request)
{
    var produto = new Produto { Nome = request.Nome };
    _repositorio.Adicionar(produto);

    return CreatedAtAction(nameof(ObterPorId), new { id = produto.Id }, produto); // 201
}
```

## 🔹 Os códigos mais usados

- `200 OK` → sucesso, com corpo de resposta  
- `201 Created` → recurso criado, com localização dele no header  
- `404 Not Found` → recurso não encontrado  
- `400 Bad Request` → requisição inválida (geralmente por falha de validação)  

👉 Devolver o código HTTP correto não é só formalidade — é o contrato que quem consome sua API depende para tratar erros corretamente

---

# 🧪 Testando com o Swagger

O template já sobe uma interface interativa em `/swagger`, onde você pode:

- Ver todos os endpoints disponíveis  
- Testar requisições direto do navegador  
- Ver o formato esperado de cada request/response  

👉 É a forma mais rápida de validar sua API sem precisar escrever um front-end ou usar ferramentas externas enquanto desenvolve

---

# ⚠️ Erros comuns

- Expor diretamente a classe de domínio (`Produto`) em vez de um DTO/record de request, misturando modelo de dados com contrato de API  
- Usar métodos síncronos para operações de I/O (banco de dados, chamadas externas) em vez de `async`/`await`, desperdiçando o que você aprendeu sobre programação assíncrona  
- Devolver `200 OK` para tudo, mesmo em casos de erro ou "não encontrado"  
- Colocar lógica de negócio direto no controller, em vez de delegar para classes de serviço — controllers devem ser finos  

---

# 📌 Conclusão

- ASP.NET Core expõe funcionalidades via endpoints HTTP  
- Minimal APIs são enxutas; Controllers organizam melhor APIs maiores  
- `[FromBody]`, `[FromRoute]` e `[FromQuery]` controlam de onde os dados vêm  
- A injeção de dependência do ASP.NET Core aplica DIP automaticamente, sem código manual  
- Códigos HTTP corretos (`200`, `201`, `404`, `400`) fazem parte do contrato da sua API  

👉 Você acabou de dar o passo que transforma conhecimento de linguagem em capacidade de construir sistemas reais

---

# 🔥 Próximo passo

Agora que você tem uma API funcionando, o próximo nível é:

👉 **Fundamentos do C#: Entity Framework Core — Persistindo Dados de Verdade**

Aqui você vai trocar o repositório em memória por um banco de dados real, sem quebrar nada do que já construiu.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
