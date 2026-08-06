# 🧠 Fundamentos do C#: CQRS e MediatR

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Mensageria para desacoplar sistemas no tempo  
- Clean Architecture organizando a aplicação em camadas  

Na Clean Architecture, você criou use cases como `CriarProdutoUseCase`. Conforme a aplicação cresce, esses casos de uso se multiplicam, e as classes de serviço começam a acumular métodos de leitura e escrita misturados. Existe um padrão que separa isso de forma ainda mais explícita.

👉 **Vamos conhecer CQRS e a biblioteca MediatR**

---

# 💡 O que é CQRS?

👉 **CQRS = Command Query Responsibility Segregation — separar operações que mudam estado (commands) das que só leem dados (queries)**

- **Command** → "criar pedido", "atualizar produto", "cancelar assinatura" — muda o sistema, geralmente não retorna dados além de uma confirmação  
- **Query** → "buscar pedido por id", "listar produtos" — só lê, nunca modifica nada  

👉 Essa separação existe porque leitura e escrita costumam ter necessidades muito diferentes: escrita precisa de validação e regras de negócio; leitura precisa de performance e formatos otimizados para exibição

---

# 📬 O padrão Mediator: um passo antes do MediatR

👉 **Mediator = um objeto central que recebe uma solicitação e a encaminha para o handler certo, sem que quem envia precise conhecer quem processa**

```
Controller → Mediator → Handler específico
```

Isso é, na prática, uma aplicação do **Dependency Inversion Principle**: o controller depende só do mediator (uma abstração), nunca diretamente de cada handler individual.

---

# 🏗️ Instalando o MediatR

```bash
dotnet add package MediatR
```

```csharp
// Program.cs
builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));
```

---

# ✍️ Definindo um Command

```csharp
public record CriarProdutoCommand(string Nome, decimal Preco) : IRequest<int>;

public class CriarProdutoHandler : IRequestHandler<CriarProdutoCommand, int>
{
    private readonly IProdutoRepositorio _repositorio;

    public CriarProdutoHandler(IProdutoRepositorio repositorio)
    {
        _repositorio = repositorio;
    }

    public async Task<int> Handle(CriarProdutoCommand command, CancellationToken cancellationToken)
    {
        var produto = new Produto(command.Nome, command.Preco);
        await _repositorio.AdicionarAsync(produto);
        return produto.Id;
    }
}
```

👉 Repare o `record` no command — exatamente o recurso do post sobre C# moderno, perfeito para representar uma intenção imutável de mudança de estado

## 🔹 Usando o Command no controller

```csharp
[HttpPost]
public async Task<IActionResult> Criar(CriarProdutoCommand command)
{
    var id = await _mediator.Send(command);
    return CreatedAtAction(nameof(ObterPorId), new { id }, null);
}
```

👉 O controller não conhece `CriarProdutoHandler` — ele só envia o command pelo `IMediator` e recebe o resultado. O mesmo desacoplamento do post sobre Clean Architecture, com uma sintaxe ainda mais explícita

---

# 🔍 Definindo uma Query

```csharp
public record ObterProdutoQuery(int Id) : IRequest<ProdutoDto>;

public class ObterProdutoHandler : IRequestHandler<ObterProdutoQuery, ProdutoDto>
{
    private readonly AppDbContext _contexto;

    public ObterProdutoHandler(AppDbContext contexto)
    {
        _contexto = contexto;
    }

    public async Task<ProdutoDto> Handle(ObterProdutoQuery query, CancellationToken cancellationToken)
    {
        return await _contexto.Produtos
            .Where(p => p.Id == query.Id)
            .Select(p => new ProdutoDto(p.Id, p.Nome, p.Preco))
            .FirstOrDefaultAsync(cancellationToken);
    }
}
```

👉 A query pode ir direto ao `DbContext` e usar LINQ para projetar exatamente os campos necessários (`ProdutoDto`), sem passar pelas regras de negócio da entidade — leitura tem liberdade que escrita não tem

---

# 🔌 Pipeline Behaviors: interceptando toda requisição

Uma das grandes vantagens do MediatR é poder interceptar **todo** command/query que passa por ele:

```csharp
public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : notnull
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;

    public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger)
    {
        _logger = logger;
    }

    public async Task<TResponse> Handle(
        TRequest request, RequestHandlerDelegate<TResponse> proximo, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Processando {Nome}", typeof(TRequest).Name);
        var resposta = await proximo();
        _logger.LogInformation("Concluído {Nome}", typeof(TRequest).Name);
        return resposta;
    }
}
```

👉 Isso aplica logging (do post sobre Serilog) automaticamente a **todos** os commands e queries, sem repetir código em cada handler — o mesmo princípio funciona para validação, cache e transações de banco

---

# 🔀 CQRS não exige dois bancos de dados

👉 Um erro comum é achar que CQRS sempre significa bancos separados para leitura e escrita. Na maioria dos projetos, **commands e queries usam o mesmo banco** — CQRS aqui é só uma separação de responsabilidade no código, não de infraestrutura. Separar bancos de dados é uma evolução opcional, útil só em sistemas com necessidades extremas de escala.

---

# ⚠️ Erros comuns

- Achar que CQRS exige bancos de dados separados desde o início  
- Criar um handler gigante que faz leitura e escrita ao mesmo tempo, perdendo o propósito da separação  
- Usar MediatR para tudo, mesmo operações triviais que não precisam dessa indireção  
- Colocar lógica de negócio pesada dentro do handler de uma query, quando queries deveriam ser simples e rápidas  

---

# 📌 Conclusão

- CQRS separa operações de escrita (commands) das de leitura (queries)  
- MediatR desacopla quem envia a solicitação de quem a processa, via `IMediator`  
- Pipeline behaviors aplicam comportamento transversal (log, validação) a todos os handlers  
- CQRS não exige infraestrutura separada — normalmente é só organização de código  

👉 Com CQRS e MediatR, sua aplicação ganha uma separação ainda mais clara entre o que muda o sistema e o que só consulta ele

---

# 🔥 Próximo passo

Agora que você sabe separar comandos de consultas, o próximo nível é:

👉 **Fundamentos do C#: Domain-Driven Design (DDD) — Introdução**

Aqui você vai aprender a modelar regras de negócio complexas de um jeito que reflete de verdade a linguagem do domínio que você está construindo.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
