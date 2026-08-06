# 🧠 Fundamentos do C#: Mensageria com RabbitMQ

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Resiliência com Polly para chamadas síncronas  
- Cache para reduzir carga em consultas repetidas  

Até agora, todo sistema externo que sua API chama é síncrono: você pede, espera, recebe a resposta. Mas e quando a resposta pode esperar? Enviar um e-mail de confirmação não precisa travar a resposta do pedido para o usuário.

👉 **É aqui que entra a mensageria**

---

# 💡 O problema da comunicação síncrona

```csharp
[HttpPost]
public async Task<IActionResult> CriarPedido(CriarPedidoRequest request)
{
    var pedido = await _servicoPedido.CriarAsync(request);

    await _servicoEmail.EnviarConfirmacaoAsync(pedido);      // e se o serviço de e-mail estiver lento?
    await _servicoEstoque.AtualizarAsync(pedido);             // e se o serviço de estoque estiver fora do ar?

    return Ok(pedido);
}
```

👉 O usuário fica esperando o pedido inteiro — criar + notificar + atualizar estoque — mesmo que só a criação do pedido seja realmente urgente. Se qualquer uma dessas etapas falhar ou demorar, a experiência inteira sofre

---

# 📬 O que é uma fila de mensagens?

👉 **Fila de mensagens = um intermediário que recebe mensagens de um produtor e as entrega a um ou mais consumidores, de forma assíncrona e desacoplada**

```
Produtor → [ Fila (RabbitMQ) ] → Consumidor
```

O produtor publica a mensagem e **segue em frente imediatamente** — não espera o consumidor processá-la. Isso é a mesma ideia de `async`/`await` que você já conhece, só que aplicada entre **sistemas diferentes**, não entre métodos do mesmo processo.

---

# 🏗️ Publicando uma mensagem

```bash
dotnet add package RabbitMQ.Client
```

```csharp
public class PublicadorPedido
{
    public void Publicar(PedidoCriadoEvento evento)
    {
        var fabrica = new ConnectionFactory { HostName = "localhost" };

        using var conexao = fabrica.CreateConnection();
        using var canal = conexao.CreateModel();

        canal.QueueDeclare(queue: "pedidos-criados", durable: true, exclusive: false, autoDelete: false);

        var corpo = Encoding.UTF8.GetBytes(JsonSerializer.Serialize(evento));

        canal.BasicPublish(exchange: "", routingKey: "pedidos-criados", body: corpo);
    }
}
```

```csharp
[HttpPost]
public async Task<IActionResult> CriarPedido(CriarPedidoRequest request)
{
    var pedido = await _servicoPedido.CriarAsync(request);

    _publicador.Publicar(new PedidoCriadoEvento(pedido.Id, pedido.Cliente));

    return Ok(pedido); // responde imediatamente, sem esperar e-mail nem estoque
}
```

👉 A API responde ao usuário assim que o pedido é criado. O envio de e-mail e a atualização de estoque acontecem **depois**, de forma independente, consumidos por outro processo

---

# 📥 Consumindo mensagens

```csharp
public class ConsumidorEmail : BackgroundService
{
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var fabrica = new ConnectionFactory { HostName = "localhost" };
        var conexao = fabrica.CreateConnection();
        var canal = conexao.CreateModel();

        canal.QueueDeclare(queue: "pedidos-criados", durable: true, exclusive: false, autoDelete: false);

        var consumidor = new EventingBasicConsumer(canal);
        consumidor.Received += async (modelo, evento) =>
        {
            var mensagem = Encoding.UTF8.GetString(evento.Body.ToArray());
            var pedidoEvento = JsonSerializer.Deserialize<PedidoCriadoEvento>(mensagem);

            await _servicoEmail.EnviarConfirmacaoAsync(pedidoEvento);

            canal.BasicAck(evento.DeliveryTag, multiple: false);
        };

        canal.BasicConsume(queue: "pedidos-criados", autoAck: false, consumer: consumidor);

        return Task.CompletedTask;
    }
}
```

👉 `BackgroundService` roda continuamente, ouvindo a fila. `BasicAck` confirma que a mensagem foi processada com sucesso — se o consumidor cair antes de confirmar, a mensagem **volta para a fila** e é entregue novamente

---

# 🔀 Filas vs Chamadas HTTP diretas: quando usar cada uma

## 🔹 Use chamada HTTP direta quando:
- A resposta é necessária **imediatamente** para o usuário  
- A operação é simples e rápida  

## 🔹 Use fila de mensagens quando:
- A operação pode acontecer **depois**, sem bloquear o usuário  
- Você quer que uma falha temporária no consumidor não afete o produtor  
- Múltiplos sistemas precisam reagir ao mesmo evento (e-mail, estoque, analytics, tudo consumindo a mesma mensagem)  

👉 A fila também funciona como um **buffer**: se o consumidor cair por alguns minutos, as mensagens só se acumulam e são processadas quando ele voltar — nada se perde

---

# ⚠️ Erros comuns

- Usar `autoAck: true` sem pensar nas consequências: se o processo cair no meio do processamento, a mensagem é perdida, não reprocessada  
- Colocar lógica que precisa de resposta imediata (ex: validar pagamento) dentro de uma fila assíncrona  
- Não tratar mensagens duplicadas — filas geralmente garantem "pelo menos uma entrega", então o consumidor deve ser capaz de processar a mesma mensagem duas vezes sem efeito colateral  
- Ignorar falhas no consumidor, deixando mensagens acumularem indefinidamente sem alerta  

---

# 📌 Conclusão

- Filas de mensagens desacoplam produtor e consumidor no tempo, sem exigir resposta imediata  
- Publicar uma mensagem libera a API para responder ao usuário sem esperar tarefas secundárias  
- `BasicAck`/`autoAck: false` garante que mensagens não processadas voltem para a fila  
- Filas funcionam como buffer: consumidores fora do ar não derrubam o sistema inteiro  

👉 Com mensageria, seu sistema para de depender de tudo acontecer na mesma linha do tempo síncrona, ganhando resiliência e escalabilidade

---

# 🔥 Próximo passo

Agora que você sabe desacoplar sistemas com filas, o próximo nível é:

👉 **Fundamentos do C#: CQRS e MediatR**

Aqui você vai aprender a separar operações de escrita e leitura, organizando ainda melhor a lógica de negócio da sua aplicação.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
