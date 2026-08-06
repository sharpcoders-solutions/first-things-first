# 🧠 Fundamentos do C#: Padrão Saga para Transações Distribuídas

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Event Sourcing para guardar o histórico completo de mudanças  
- Microsserviços — cada um com seu próprio banco de dados  

Se cada microsserviço tem seu próprio banco, o que acontece quando um pedido precisa: reservar estoque, cobrar pagamento e agendar entrega — três bancos diferentes? Uma transação `BEGIN/COMMIT` tradicional não atravessa serviços. É aqui que entra o padrão Saga.

👉 **Vamos aprender o padrão Saga**

---

# 💡 O problema: transações que cruzam serviços

```csharp
// ❌ Isso não funciona entre microsserviços diferentes
using var transacao = await _contexto.Database.BeginTransactionAsync();
_servicoEstoque.Reservar(pedidoId);   // banco do serviço de estoque
_servicoPagamento.Cobrar(pedidoId);   // banco do serviço de pagamento
_servicoEntrega.Agendar(pedidoId);    // banco do serviço de entrega
await transacao.CommitAsync();        // não existe transação distribuída aqui
```

👉 Cada serviço tem seu próprio banco (lembra do post sobre microsserviços?) — não há um `COMMIT` único que amarre os três

---

# 🏗️ Saga = uma sequência de transações locais compensáveis

👉 **Saga = uma cadeia de passos, onde cada passo tem uma ação e uma compensação para desfazê-lo se algo falhar depois**

## 🔹 Saga coreografada (eventos)

```csharp
public class ServicoPedidos
{
    public async Task CriarPedido(int pedidoId)
    {
        await _pedidos.Criar(pedidoId);
        await _publicador.Publicar(new PedidoCriado(pedidoId));
    }
}

public class ServicoEstoque
{
    public async Task AoReceberPedidoCriado(PedidoCriado evento)
    {
        var reservado = await _estoque.Reservar(evento.PedidoId);

        if (reservado)
            await _publicador.Publicar(new EstoqueReservado(evento.PedidoId));
        else
            await _publicador.Publicar(new EstoqueIndisponivel(evento.PedidoId));
    }
}
```

👉 Lembra do RabbitMQ (post 41)? Cada serviço reage ao evento do anterior e publica o próprio — não existe um "maestro" central, os serviços se coordenam via mensageria

## 🔹 Saga orquestrada (um coordenador central)

```csharp
public class OrquestradorPedido
{
    public async Task Executar(int pedidoId)
    {
        try
        {
            await _estoque.Reservar(pedidoId);
            await _pagamento.Cobrar(pedidoId);
            await _entrega.Agendar(pedidoId);
        }
        catch (Exception)
        {
            await Compensar(pedidoId);
            throw;
        }
    }

    private async Task Compensar(int pedidoId)
    {
        await _entrega.CancelarAgendamento(pedidoId);
        await _pagamento.Estornar(pedidoId);
        await _estoque.LiberarReserva(pedidoId);
    }
}
```

👉 Um orquestrador central conhece toda a sequência e decide o que fazer em cada falha — mais fácil de entender e depurar, mas cria um ponto central de coordenação

---

# ⚖️ Coreografada vs orquestrada

## 🔹 Coreografada
- Sem ponto central — serviços desacoplados, cada um só conhece seus próprios eventos  
- Difícil visualizar o fluxo completo só lendo o código  

## 🔹 Orquestrada
- Fluxo completo visível em um só lugar  
- O orquestrador vira uma dependência que todos os serviços precisam conhecer  

👉 Para sagas simples (2-3 passos), coreografada funciona bem. Para sagas complexas com muitas ramificações, orquestrada costuma ser mais fácil de manter

---

# 🔄 Compensação: o coração do padrão

👉 **Cada ação precisa de uma compensação equivalente e idempotente**

| Ação | Compensação |
|---|---|
| Reservar estoque | Liberar reserva |
| Cobrar pagamento | Estornar |
| Agendar entrega | Cancelar agendamento |

👉 Diferente de um `ROLLBACK` de banco, a compensação é código de negócio explícito — estornar um pagamento não é "desfazer", é uma nova operação que neutraliza a anterior

---

# ⚠️ Erros comuns

- Tratar Saga como uma transação ACID tradicional — durante a execução, o sistema fica em estados intermediários visíveis (consistência eventual, não imediata)  
- Esquecer de tornar as compensações idempotentes, causando estornos duplicados em caso de retry  
- Escolher orquestração para tudo, criando um serviço "central" que sabe demais sobre os outros  
- Não monitorar sagas travadas no meio do caminho — sem observabilidade (lembra do post 55?), uma saga incompleta pode passar despercebida  

---

# 📌 Conclusão

- Saga coordena transações que cruzam múltiplos serviços através de passos locais compensáveis  
- Coreografada usa eventos; orquestrada usa um coordenador central  
- Cada ação precisa de uma compensação idempotente para desfazer o efeito em caso de falha  
- Sagas trazem consistência eventual, não a atomicidade imediata de uma transação de banco tradicional  

👉 Com o padrão Saga, sistemas distribuídos ganham uma forma confiável de coordenar operações que nenhum banco sozinho consegue amarrar

---

# 🔥 Próximo passo

Agora que você sabe coordenar transações distribuídas, o próximo nível é:

👉 **Fundamentos do C#: Padrão Outbox**

Aqui você vai aprender a garantir que uma mudança no banco e a publicação de um evento aconteçam de forma confiável, juntas.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
