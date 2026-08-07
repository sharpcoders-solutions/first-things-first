# 🧠 Fundamentos do C#: Observabilidade com OpenTelemetry

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- API Gateway como ponto de entrada único  
- Logging estruturado com Serilog  
- Health Checks para monitorar disponibilidade  

Cada peça isolada (logs, métricas, health checks) conta parte da história. Mas quando uma requisição passa pelo gateway, por três microsserviços e por um banco de dados, você precisa enxergar o caminho inteiro — não pedaços soltos.

👉 **Vamos aprender Observabilidade com OpenTelemetry**

---

# 💡 Os três pilares da observabilidade

👉 **Observabilidade = conseguir entender o que acontece dentro do sistema, olhando de fora**

- **Logs** — eventos discretos ("pedido criado", "pagamento falhou")  
- **Métricas** — números agregados ao longo do tempo (requisições/segundo, latência média)  
- **Traces** — o caminho completo de uma requisição através de múltiplos serviços  

O OpenTelemetry (OTel) é o padrão aberto que unifica os três, sem prender você a um fornecedor específico.

---

# 🏗️ Configurando o OpenTelemetry

```bash
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

```csharp
// Program.cs
builder.Services.AddOpenTelemetry()
    .ConfigureResource(recurso => recurso.AddService("servico-pedidos"))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddOtlpExporter())
    .WithMetrics(metricas => metricas
        .AddAspNetCoreInstrumentation()
        .AddRuntimeInstrumentation()
        .AddOtlpExporter());
```

👉 `AddOtlpExporter` envia os dados para um coletor (como Jaeger, Grafana Tempo ou o Application Insights) — o código C# não precisa saber para onde os dados vão

---

# 🔗 Traces distribuídos: seguindo a requisição

```csharp
using var activity = _fonteAtividade.StartActivity("ProcessarPedido");
activity?.SetTag("pedido.id", pedidoId);

var estoque = await _clienteEstoque.VerificarDisponibilidade(pedidoId);
var pagamento = await _clientePagamento.Processar(pedidoId);
```

👉 Cada `Activity` (span) representa uma etapa. Quando o serviço de pedidos chama o serviço de estoque via HTTP, o `TraceId` viaja automaticamente no cabeçalho da requisição — no coletor, você vê a árvore inteira:

```
ProcessarPedido (servico-pedidos) — 340ms
  └─ VerificarDisponibilidade (servico-estoque) — 120ms
  └─ ProcessarPagamento (servico-pagamento) — 180ms
```

👉 Lembra do gRPC e do RabbitMQ? Um trace distribuído mostra exatamente onde os 340ms foram gastos, mesmo cruzando três serviços diferentes

---

# 📊 Métricas customizadas

```csharp
var medidor = new Meter("ServicoPedidos");
var contadorPedidos = medidor.CreateCounter<int>("pedidos.criados");

contadorPedidos.Add(1, new KeyValuePair<string, object?>("status", "sucesso"));
```

👉 Diferente de logs (eventos individuais), métricas são agregadas — "quantos pedidos foram criados nos últimos 5 minutos" é uma pergunta de métrica, não de log

---

# 🔍 Correlacionando logs, métricas e traces

```csharp
builder.Logging.AddOpenTelemetry(opcoes =>
{
    opcoes.IncludeFormattedMessage = true;
    opcoes.IncludeScopes = true;
});
```

👉 Combinado com o Serilog (post 37), cada linha de log ganha automaticamente o `TraceId` da requisição atual — ao investigar um erro, você salta direto do log para o trace completo que causou aquele erro

---

# ⚠️ Erros comuns

- Instrumentar só um serviço e achar que já tem observabilidade — o valor real está em ver a cadeia **completa**  
- Não propagar o contexto de trace entre chamadas assíncronas (filas, jobs), quebrando a cadeia de rastreamento  
- Criar métricas com alta cardinalidade (como usar o ID do usuário como tag), explodindo o custo de armazenamento  
- Confundir observabilidade com monitoramento — monitoramento avisa que algo quebrou, observabilidade ajuda a entender **por quê**  

---

# 📌 Conclusão

- Observabilidade une logs, métricas e traces para entender sistemas distribuídos  
- OpenTelemetry é o padrão aberto e independente de fornecedor para instrumentação  
- Traces distribuídos mostram o caminho completo de uma requisição através de múltiplos serviços  
- Correlacionar logs com `TraceId` acelera drasticamente a investigação de problemas  

👉 Com OpenTelemetry, sistemas distribuídos deixam de ser uma caixa-preta e passam a contar sua própria história

---

# 🔥 Próximo passo

Agora que você consegue rastrear requisições através de serviços, o próximo nível é:

👉 **Fundamentos do C#: Event Sourcing — Introdução**

Aqui você vai aprender a guardar o histórico completo de mudanças de estado, não só o estado atual.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
