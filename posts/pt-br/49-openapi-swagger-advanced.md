# 🧠 Fundamentos do C#: Documentando APIs com OpenAPI/Swagger Avançado

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Versionamento de API  
- Segurança seguindo o OWASP Top 10  

Desde o post sobre ASP.NET Core, você usa o Swagger básico gerado automaticamente. Mas para uma API com múltiplas versões e endpoints protegidos, a documentação padrão já não é suficiente.

👉 **Vamos deixar sua documentação realmente pronta para quem consome sua API de fora**

---

# 💡 O que é OpenAPI?

👉 **OpenAPI = a especificação que descreve formalmente uma API REST. Swagger = o conjunto de ferramentas que lê essa especificação e gera a interface interativa**

Toda a configuração que você já viu (`[HttpGet]`, `[FromBody]`, `[ApiVersion]`) já alimenta essa especificação automaticamente — agora vamos enriquecê-la

---

# 🏗️ Enriquecendo a configuração básica

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Minha API",
        Version = "v1",
        Description = "API para gerenciamento de produtos e pedidos",
        Contact = new OpenApiContact { Name = "Vitor Santos", Email = "contato@empresa.com" }
    });

    options.SwaggerDoc("v2", new OpenApiInfo { Title = "Minha API", Version = "v2" });
});
```

👉 Cada versão registrada aqui gera uma documentação **separada**, conectando diretamente com o post sobre versionamento — quem consome a v1 vê só os endpoints da v1

---

# 📝 Documentando endpoints com XML comments

```bash
dotnet add package Swashbuckle.AspNetCore.Annotations
```

```csharp
/// <summary>
/// Cria um novo produto no catálogo.
/// </summary>
/// <param name="request">Nome e preço do produto a ser criado.</param>
/// <returns>O produto recém-criado, incluindo seu Id.</returns>
/// <response code="201">Produto criado com sucesso.</response>
/// <response code="400">Dados de entrada inválidos.</response>
[HttpPost]
[ProducesResponseType(typeof(ProdutoResponse), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public IActionResult Criar(CriarProdutoRequest request)
{
    // ...
}
```

```xml
<!-- No .csproj -->
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

👉 `[ProducesResponseType]` documenta explicitamente **cada** código HTTP possível — lembra do post sobre respostas HTTP corretas? Agora o Swagger mostra isso para quem consome, sem precisar ler o código-fonte

---

# 🔒 Documentando autenticação no Swagger

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        Description = "Insira o token JWT no formato: Bearer {seu token}"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" }
            },
            Array.Empty<string>()
        }
    });
});
```

👉 Isso adiciona um botão **"Authorize"** na interface do Swagger — quem for testar a API consegue colar o token JWT (do post sobre autenticação) uma vez e testar todos os endpoints protegidos sem repetir o header manualmente

---

# 📋 Exemplos de request/response

```csharp
public record CriarProdutoRequest(string Nome, decimal Preco)
{
    /// <example>Notebook Dell</example>
    public string Nome { get; init; } = Nome;

    /// <example>3500.00</example>
    public decimal Preco { get; init; } = Preco;
}
```

👉 Exemplos concretos no corpo da requisição eliminam a ambiguidade — em vez de adivinhar o formato esperado, quem consome a API vê exatamente como preencher cada campo

---

# 🗂️ Organizando com Tags

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[Tags("Produtos")]
public class ProdutosController : ControllerBase { }

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[Tags("Pedidos")]
public class PedidosController : ControllerBase { }
```

👉 Em APIs com muitos controllers, tags agrupam os endpoints na interface do Swagger de forma lógica — a diferença entre uma documentação navegável e uma lista confusa de cinquenta endpoints soltos

---

# ⚠️ Erros comuns

- Deixar a documentação genérica gerada automaticamente, sem descrições reais do que cada endpoint faz  
- Esquecer de documentar os códigos de erro possíveis, deixando quem consome descobrir na marra  
- Não manter a documentação sincronizada entre versões, confundindo quem ainda usa a v1  
- Expor o Swagger publicamente em produção sem nenhuma proteção, revelando toda a estrutura da API para qualquer pessoa  

---

# 📌 Conclusão

- OpenAPI é a especificação; Swagger é a ferramenta que a torna interativa  
- `[ProducesResponseType]` documenta explicitamente cada resposta HTTP possível  
- Configurar autenticação no Swagger permite testar endpoints protegidos direto na interface  
- Tags organizam APIs grandes em grupos navegáveis  

👉 Uma API bem documentada é tão importante quanto uma API bem construída — é o que permite que outros times (e você mesmo, no futuro) a usem sem precisar ler o código-fonte inteiro

---

# 🔥 Próximo passo

Você chegou ao fim da jornada técnica desta trilha — do primeiro `Console.WriteLine` até uma API completa, segura, documentada e pronta para produção. O próximo (e último) passo é sobre você:

👉 **Fundamentos do C#: Carreira — Preparando-se para Entrevistas de C#/.NET**

Aqui você vai aprender a transformar tudo que construiu nesta trilha em uma carreira de verdade.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
