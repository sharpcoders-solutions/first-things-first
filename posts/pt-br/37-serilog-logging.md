# 🧠 Fundamentos do C#: Logging Estruturado com Serilog

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- IDisposable e o padrão Dispose  
- Streams e manipulação de arquivos  

Sua API está protegida e persiste dados de verdade. Mas quando algo dá errado às três da manhã, como você descobre **o quê**, **onde** e **por quê**?

👉 **É para isso que existe logging estruturado**

---

# 💡 `Console.WriteLine` não é suficiente

Você usou `Console.WriteLine` desde o post sobre seu primeiro programa — ótimo para aprender, péssimo para produção:

- Não tem níveis de severidade (é tudo igual: erro, aviso, informação)  
- Não é pesquisável nem filtrável  
- Some quando o container reinicia, se não for enviado para algum lugar persistente  

👉 **Logging estruturado** resolve isso: cada entrada de log vira um evento com dados pesquisáveis, não só uma linha de texto solta

---

# 🏗️ Configurando o Serilog

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

```csharp
// Program.cs
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

builder.Host.UseSerilog();
```

## 🔹 O conceito de "sink"

👉 **Sink = para onde o log é enviado**

Console, arquivo, banco de dados, ou serviços como Seq, Elasticsearch e Application Insights — você pode enviar o mesmo log para vários destinos ao mesmo tempo, sem mudar uma linha do código que gera o log

---

# 🎯 Níveis de log

```csharp
_logger.LogTrace("Detalhe extremamente granular");
_logger.LogDebug("Informação útil durante desenvolvimento");
_logger.LogInformation("Pedido {PedidoId} processado com sucesso", pedido.Id);
_logger.LogWarning("Estoque baixo para o produto {ProdutoId}", produto.Id);
_logger.LogError(ex, "Falha ao processar pagamento do pedido {PedidoId}", pedido.Id);
_logger.LogCritical("Banco de dados inacessível");
```

## 🔹 Quando usar cada nível

- `Trace`/`Debug` → detalhes só relevantes durante desenvolvimento  
- `Information` → eventos normais do fluxo da aplicação  
- `Warning` → algo inesperado, mas que não quebrou nada  
- `Error` → uma operação falhou (geralmente acompanhado de uma exceção)  
- `Critical` → a aplicação está em risco de parar de funcionar  

👉 Em produção, normalmente você configura o nível mínimo como `Information`, silenciando `Trace` e `Debug` para não gerar ruído

---

# 🧩 Logging estruturado de verdade: os `{placeholders}`

```csharp
_logger.LogInformation("Pedido {PedidoId} processado com sucesso", pedido.Id);
```

👉 Repare que **não** é interpolação de string (`$"Pedido {pedido.Id}"`). Os `{PedidoId}` viram **propriedades pesquisáveis** no log, não só texto formatado

Isso permite consultas como "me mostre todos os logs onde `PedidoId = 42`" em ferramentas de observabilidade — algo impossível se tudo virou uma string solta

---

# 🔌 Injetando o logger via DI

```csharp
public class ProdutosController : ControllerBase
{
    private readonly ILogger<ProdutosController> _logger;

    public ProdutosController(ILogger<ProdutosController> logger)
    {
        _logger = logger;
    }

    [HttpGet("{id}")]
    public IActionResult ObterPorId(int id)
    {
        _logger.LogInformation("Buscando produto {ProdutoId}", id);

        var produto = _repositorio.ObterPorId(id);

        if (produto is null)
        {
            _logger.LogWarning("Produto {ProdutoId} não encontrado", id);
            return NotFound();
        }

        return Ok(produto);
    }
}
```

👉 `ILogger<T>` já vem registrado no container de injeção de dependência do ASP.NET Core — mais uma vez, o mesmo mecanismo de DI que você já domina desde o post sobre construção de APIs

---

# 🌐 Middleware de log de requisições

```csharp
app.UseSerilogRequestLogging();
```

👉 Uma única linha registra automaticamente **toda** requisição HTTP: método, rota, status de resposta e tempo de execução — sem precisar adicionar log manual em cada endpoint

---

# ⚠️ Erros comuns

- Usar interpolação de string (`$"..."`) em vez de placeholders, perdendo a estrutura pesquisável do log  
- Logar dados sensíveis (senhas, tokens, números de cartão) sem perceber  
- Deixar o nível mínimo como `Debug` em produção, gerando volume de log desnecessário  
- Capturar uma exceção e não logar nada — o erro simplesmente desaparece sem deixar rastro  

---

# 📌 Conclusão

- Logging estruturado transforma logs em dados pesquisáveis, não só texto  
- Níveis de log (`Information`, `Warning`, `Error`...) classificam a severidade de cada evento  
- `{Placeholders}` viram propriedades consultáveis, diferente de interpolação de string  
- `ILogger<T>` é injetado via DI, exatamente como qualquer outra dependência da sua aplicação  

👉 Com logging estruturado, sua aplicação para de ser uma caixa-preta e passa a contar, em detalhe, o que está acontecendo por dentro

---

# 🔥 Próximo passo

Agora que você sabe enxergar o que sua aplicação está fazendo, o próximo nível é:

👉 **Fundamentos do C#: Health Checks e Monitoramento**

Aqui você vai aprender a fazer sua aplicação relatar, de forma automatizada, se ela (e suas dependências) estão saudáveis.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
