# 🧠 Fundamentos do C#: Observabilidade Completa (Métricas, Traces, Logs)

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Ao longo desta trilha, você construiu observabilidade em pedaços: Serilog (post 37), Health Checks (post 38), OpenTelemetry (post 55), e usou tudo isso para validar Canary Deployments (post 89) e experimentos de Chaos Engineering (post 90). Chegou a hora de ver como esses pedaços formam um sistema coeso.

👉 **Vamos consolidar Observabilidade Completa**

---

# 💡 Os três pilares, revisitados juntos

```csharp
// Logs (post 37): o que aconteceu
_logger.LogInformation("Pedido {PedidoId} criado para cliente {ClienteId}", pedido.Id, clienteId);

// Métricas (post 55): quantos, com que frequência
var contadorPedidos = medidor.CreateCounter<int>("pedidos.criados");
contadorPedidos.Add(1);

// Traces (post 55): o caminho completo da requisição
using var atividade = _fonteAtividade.StartActivity("ProcessarPedido");
```

👉 Cada pilar sozinho responde uma pergunta parcial. Juntos, respondem: **o que** aconteceu, **quantas vezes**, e **onde exatamente** no fluxo da requisição

---

# 🏗️ Um incidente real, investigado com os três pilares

```
1. Alerta dispara: latência p99 subiu de 200ms para 3s (métrica)
   
2. Você filtra traces com duração > 2s no período do alerta
   └─ Encontra: 80% dos traces lentos passam pelo ServicoEstoque
   
3. Você abre os logs correlacionados com o TraceId daquele trace específico
   └─ Encontra: "Timeout ao conectar no banco de dados de estoque"
   
4. Causa raiz identificada em minutos, não horas
```

👉 Sem métrica, você não saberia que havia um problema. Sem trace, não saberia **onde**. Sem log correlacionado (lembra do `TraceId` automático do post 55?), não saberia **por quê**

---

# 🎯 Dashboards: transformando dados em visão

```csharp
// Métricas customizadas de negócio, não só técnicas
var valorTotalPedidos = medidor.CreateCounter<decimal>("pedidos.valor_total");
var pedidosPorStatus = medidor.CreateCounter<int>("pedidos.por_status");

valorTotalPedidos.Add(pedido.Valor);
pedidosPorStatus.Add(1, new KeyValuePair<string, object?>("status", pedido.Status));
```

👉 Observabilidade não é só sobre erros e performance — métricas de negócio (quantos pedidos, qual valor total, taxa de conversão) no mesmo painel que métricas técnicas dão visibilidade completa do sistema, não só da infraestrutura

---

# 🔍 Correlacionando com tudo que você já construiu

```
Health Checks (post 38): "o serviço está de pé?"
  └─ Resposta binária, superficial

Observabilidade completa: "o serviço está de pé, respondendo em 150ms,
processando 500 req/s, com 0.1% de taxa de erro, e a última requisição 
lenta veio do endpoint de checkout, travada na chamada ao banco"
  └─ Resposta rica, acionável
```

👉 Health Checks (post 38) dizem se algo está vivo. Observabilidade completa diz **como** está vivo, e dá contexto suficiente para agir antes que vire um incidente

---

# 📊 SLIs, SLOs e SLAs: transformando dados em compromisso

```
SLI (Service Level Indicator): latência p99 medida via OpenTelemetry
SLO (Service Level Objective): "p99 < 500ms em 99.5% do tempo"
SLA (Service Level Agreement): compromisso contratual baseado no SLO
```

👉 As métricas que você coleta desde o post 55 não existem só para debugar — elas alimentam objetivos formais de qualidade de serviço, usados para decidir prioridades de engenharia e comunicar confiabilidade para o negócio

---

# ⚠️ Erros comuns

- Coletar dados sem definir o que fazer com eles — observabilidade sem alertas acionáveis é só ruído acumulado  
- Ter métricas técnicas mas nenhuma de negócio, perdendo visibilidade sobre o impacto real dos problemas  
- Não correlacionar os três pilares via `TraceId`, forçando investigação manual e lenta entre ferramentas separadas  
- Definir SLOs arbitrários, sem basear em dados reais de comportamento histórico do sistema  

---

# 📌 Conclusão

- Logs, métricas e traces respondem perguntas complementares, não substituem um ao outro  
- Correlacionar os três pilares via `TraceId` transforma investigação de horas em minutos  
- Métricas de negócio, lado a lado com métricas técnicas, dão visão completa do sistema  
- SLIs/SLOs transformam dados de observabilidade em compromissos formais de qualidade  

👉 Com observabilidade completa, seu sistema para de ser uma caixa-preta que só fala quando quebra, e passa a contar continuamente sua própria história

---

# 🔥 Próximo passo

Agora que você domina observabilidade de ponta a ponta, o próximo nível é:

👉 **Fundamentos do C#: Identity Server e OAuth2**

Aqui você vai aprofundar em autenticação além do JWT básico, entendendo os fluxos completos do OAuth2 e OpenID Connect.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
