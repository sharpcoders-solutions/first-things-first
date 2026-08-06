# 🧠 Fundamentos do C#: Versionamento de API

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Segurança avançada seguindo o OWASP Top 10  
- Como estruturar contratos de API com DTOs e records  

Sua API está no ar, sendo consumida por outros sistemas. Um dia você vai precisar mudar um contrato — e quebrar quem já depende dele não é uma opção.

👉 **Vamos aprender a evoluir uma API sem quebrar seus consumidores**

---

# 💡 Por que versionar?

👉 **Versionamento = permitir que múltiplas versões da mesma API coexistam, para que clientes migrem no próprio ritmo**

```csharp
public record ProdutoResponseV1(int Id, string Nome, decimal Preco);

public record ProdutoResponseV2(int Id, string Nome, decimal Preco, string Categoria, bool EmEstoque);
```

👉 Sem versionamento, adicionar `Categoria` como campo obrigatório pode quebrar um cliente que não esperava esse campo — ou pior, remover um campo que outro sistema ainda usa derruba integrações inteiras sem aviso

---

# 🏗️ Estratégias de versionamento

```bash
dotnet add package Asp.Versioning.Mvc
```

## 🔹 Versionamento por URL (a mais comum e explícita)

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProdutosController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult ObterTodosV1() => Ok(/* ... */);

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult ObterTodosV2() => Ok(/* ... */);
}
```

```
GET /api/v1/produtos
GET /api/v2/produtos
```

👉 Simples de entender e testar direto no navegador — a versão fica visível na própria URL

## 🔹 Versionamento por header

```csharp
[HttpGet]
public IActionResult ObterTodos([FromHeader(Name = "X-Api-Version")] string versao)
{
    return versao switch
    {
        "2.0" => Ok(_servico.ObterTodosV2()),
        _ => Ok(_servico.ObterTodosV1())
    };
}
```

```
GET /api/produtos
X-Api-Version: 2.0
```

👉 Mantém a URL "limpa", mas exige que o cliente saiba adicionar o header — menos descobrível, mais comum em APIs internas entre microsserviços

## 🔹 Versionamento por query string

```
GET /api/produtos?api-version=2.0
```

👉 Fácil de testar, mas polui a URL com um parâmetro que não é realmente sobre o recurso

---

# 📦 Registrando o versionamento no ASP.NET Core

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});
```

👉 `AssumeDefaultVersionWhenUnspecified` garante que clientes antigos, que nem sabem que versionamento existe, continuem funcionando na v1 sem precisar mudar nada

---

# 🔀 Mudanças que exigem nova versão vs mudanças seguras

## 🔹 Seguro, sem nova versão

- Adicionar um **novo endpoint**  
- Adicionar um campo **opcional** na resposta (a maioria dos clientes ignora campos desconhecidos)  

## 🔹 Exige nova versão

- Remover ou renomear um campo existente  
- Mudar o tipo de um campo (`string` para `int`, por exemplo)  
- Mudar o comportamento de um endpoint existente de forma incompatível  

👉 A regra geral: se um cliente que já integra com você **quebraria** sem mudar nada do lado dele, é uma mudança que exige versão nova

---

# 🗑️ Depreciando versões antigas com responsabilidade

```csharp
[ApiVersion("1.0", Deprecated = true)]
public class ProdutosController : ControllerBase
{
    // ...
}
```

```
Response Header: api-supported-versions: 1.0, 2.0
Response Header: api-deprecated-versions: 1.0
```

👉 Marcar como `Deprecated` não remove a versão — apenas sinaliza para os consumidores, através dos headers de resposta, que eles deveriam migrar. Isso dá tempo para uma transição planejada, em vez de uma quebra súbita

---

# ⚠️ Erros comuns

- Fazer mudanças incompatíveis em uma versão existente, em vez de criar uma nova versão  
- Nunca depreciar nem remover versões antigas, acumulando manutenção infinita  
- Versionar cedo demais, criando complexidade desnecessária para uma API que ainda não tem consumidores externos  
- Documentar mal as diferenças entre versões, deixando os consumidores adivinharem o que mudou  

---

# 📌 Conclusão

- Versionamento permite evoluir a API sem quebrar quem já depende dela  
- URL, header e query string são as três estratégias mais comuns — URL é a mais explícita  
- Adicionar campos opcionais geralmente é seguro; remover ou renomear campos exige nova versão  
- Depreciar (não remover abruptamente) dá tempo para os consumidores migrarem  

👉 Uma API que versiona bem consegue evoluir continuamente, sem nunca virar refém dos clientes que já dependem dela

---

# 🔥 Próximo passo

Agora que você sabe evoluir sua API com segurança, o próximo nível é:

👉 **Fundamentos do C#: Documentando APIs com OpenAPI/Swagger Avançado**

Aqui você vai aprender a documentar cada versão e cada detalhe da sua API de um jeito que realmente ajuda quem a consome.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
