# 🧠 Fundamentos do C#: Arquitetura Orientada a Eventos

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu peças isoladas: Event Sourcing (56), Saga (57), Outbox (58) e Kafka (78). Chegou a hora de ver como elas se encaixam juntas em um sistema real, orientado a eventos do início ao fim.

👉 **Vamos conectar tudo em uma Arquitetura Orientada a Eventos**

---

# 💡 O que muda em uma arquitetura orientada a eventos?

👉 **Em vez de serviços se chamando diretamente (request/response), serviços reagem a eventos que já aconteceram**

## 🔹 Arquitetura tradicional (acoplada)

```csharp
public async Task CriarPedido(Pedido pedido)
{
    await _servicoEstoque.Reservar(pedido);      // acoplamento direto
    await _servicoPagamento.Cobrar(pedido);       // se um cair, tudo trava
    await _servicoNotificacao.Enviar(pedido);
}
```

## 🔹 Orientada a eventos (desacoplada)

```csharp
public async Task CriarPedido(Pedido pedido)
{
    await _repositorio.Salvar(pedido);
    await _publicadorEventos.Publicar(new PedidoCriado(pedido.Id)); // e pronto
}
```

👉 O serviço de pedidos não sabe (nem precisa saber) quem reage ao evento — estoque, pagamento e notificação reagem de forma independente, cada um no seu próprio ritmo

---

# 🏗️ Juntando as peças: o fluxo completo

```
1. Cliente cria pedido
   └─ ServicoPedidos salva no banco + grava evento na tabela Outbox (post 58)
   
2. DespachanteOutbox publica o evento no Kafka (post 78)
   └─ Tópico "pedidos-criados"

3. Múltiplos consumidores reagem independentemente:
   ├─ ServicoEstoque reserva o produto
   ├─ ServicoPagamento inicia a cobrança
   └─ ServicoNotificacao envia e-mail de confirmação

4. Se algo falhar, a Saga (post 57) coordena a compensação
   └─ ServicoEstoque libera a reserva
   └─ ServicoPagamento estorna, se já tiver cobrado

5. Cada mudança de estado do pedido vira um evento persistido (post 56)
   └─ Histórico completo: PedidoCriado → EstoqueReservado → PagamentoConfirmado → PedidoEnviado
```

👉 Isso é exatamente a soma dos quatro posts anteriores — Outbox garante que o evento nunca se perde, Kafka distribui em escala, Saga coordena falhas, e Event Sourcing guarda o histórico completo

---

# 🎯 Vantagens reais dessa combinação

- **Desacoplamento temporal** — serviços não precisam estar todos disponíveis ao mesmo tempo  
- **Escalabilidade independente** — o serviço de notificação pode escalar separadamente do serviço de pagamento  
- **Resiliência a falhas parciais** — se o serviço de notificação cair, pedidos continuam sendo processados; as notificações são enviadas quando ele voltar  
- **Auditoria natural** — o histórico completo de eventos já existe, sem esforço extra  

---

# ⚠️ O trade-off: consistência eventual

```csharp
// Um pedido recém-criado pode não ter estoque reservado ainda
var pedido = await _repositorio.Buscar(pedidoId);
Console.WriteLine(pedido.Status); // "Criado" — o evento de reserva ainda está sendo processado
```

👉 Lembra do que discutimos no post sobre Saga? Arquiteturas orientadas a eventos trocam consistência imediata por consistência eventual — a UI precisa ser desenhada para lidar com estados intermediários visíveis, não assumir que tudo já aconteceu instantaneamente

---

# ⚠️ Erros comuns

- Adotar arquitetura orientada a eventos para todo o sistema, quando partes dele são naturalmente síncronas e simples  
- Não investir em observabilidade (post 55) — sem tracing distribuído, depurar um fluxo de eventos que passa por 5 serviços é extremamente difícil  
- Esquecer que eventos são um contrato público entre serviços — mudar o formato de um evento publicado quebra todos os consumidores  
- Ignorar idempotência em cada consumidor, assumindo que cada evento chega exatamente uma vez  

---

# 📌 Conclusão

- Arquitetura orientada a eventos substitui chamadas diretas por reação a eventos publicados  
- Outbox, Kafka, Saga e Event Sourcing se combinam para formar um fluxo completo e resiliente  
- O ganho é desacoplamento, escalabilidade independente e auditoria natural  
- O custo é consistência eventual — um trade-off consciente, não um efeito colateral indesejado  

👉 Com arquitetura orientada a eventos, você projeta sistemas que continuam funcionando mesmo quando partes individuais falham temporariamente

---

# 🔥 Próximo passo

Agora que você conecta arquiteturas orientadas a eventos, o próximo nível é:

👉 **Fundamentos do C#: GraphQL com HotChocolate**

Aqui você vai aprender uma alternativa ao REST para consultas de API mais flexíveis, deixando o cliente decidir exatamente quais dados precisa.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
