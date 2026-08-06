# 🧠 Fundamentos do C#: Padrão Outbox

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Saga para coordenar transações distribuídas  
- RabbitMQ para publicar eventos entre serviços  

Existe um problema sutil escondido em código que parece correto: salvar no banco e publicar uma mensagem são duas operações separadas. O que acontece se uma funcionar e a outra falhar?

👉 **Vamos aprender o padrão Outbox**

---

# 💡 O problema da dupla escrita

```csharp
// ❌ Duas operações que podem falhar independentemente
public async Task CriarPedido(Pedido pedido)
{
    await _contexto.Pedidos.AddAsync(pedido);
    await _contexto.SaveChangesAsync();               // operação 1: banco

    await _barramento.Publicar(new PedidoCriado(pedido.Id)); // operação 2: mensageria
}
```

👉 Se o `SaveChangesAsync` funcionar mas o processo cair antes do `Publicar`, o pedido existe no banco, mas **nenhum outro serviço fica sabendo**. O estoque nunca é reservado, o pagamento nunca é cobrado

---

# 🏗️ A solução: a tabela Outbox

👉 **Outbox = gravar o evento na mesma transação do banco, e publicá-lo de verdade depois, de forma assíncrona**

```csharp
public class MensagemOutbox
{
    public Guid Id { get; set; }
    public string Tipo { get; set; } = default!;
    public string ConteudoJson { get; set; } = default!;
    public DateTime CriadoEm { get; set; }
    public DateTime? ProcessadoEm { get; set; }
}
```

```csharp
public async Task CriarPedido(Pedido pedido)
{
    await _contexto.Pedidos.AddAsync(pedido);

    _contexto.MensagensOutbox.Add(new MensagemOutbox
    {
        Id = Guid.NewGuid(),
        Tipo = nameof(PedidoCriado),
        ConteudoJson = JsonSerializer.Serialize(new PedidoCriado(pedido.Id)),
        CriadoEm = DateTime.UtcNow
    });

    await _contexto.SaveChangesAsync(); // ✅ uma única transação, atômica
}
```

👉 Como o pedido e a mensagem outbox são salvos na **mesma transação do EF Core** (lembra do post 32?), ou os dois são gravados, ou nenhum é — nunca um estado intermediário quebrado

---

# 📤 Publicando as mensagens: o processo de despacho

```csharp
public class DespachanteOutbox : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken cancellationToken)
    {
        while (!cancellationToken.IsCancellationRequested)
        {
            var pendentes = await _contexto.MensagensOutbox
                .Where(m => m.ProcessadoEm == null)
                .Take(50)
                .ToListAsync(cancellationToken);

            foreach (var mensagem in pendentes)
            {
                await _barramento.Publicar(mensagem.Tipo, mensagem.ConteudoJson);
                mensagem.ProcessadoEm = DateTime.UtcNow;
            }

            await _contexto.SaveChangesAsync(cancellationToken);
            await Task.Delay(TimeSpan.FromSeconds(5), cancellationToken);
        }
    }
}
```

👉 Isso é um `BackgroundService`, o mesmo mecanismo usado no post sobre Hangfire — um processo separado lê a tabela outbox e publica de verdade, tentando novamente se falhar, sem nunca perder a mensagem

---

# 🔁 Garantia at-least-once, não exactly-once

👉 **O padrão Outbox garante que a mensagem será publicada pelo menos uma vez — não exatamente uma vez**

```csharp
public async Task AoReceberPedidoCriado(PedidoCriado evento)
{
    if (await _processados.JaProcessou(evento.PedidoId))
        return; // idempotência: ignora duplicata

    await _estoque.Reservar(evento.PedidoId);
    await _processados.MarcarComoProcessado(evento.PedidoId);
}
```

👉 Se o processo de despacho falhar depois de publicar mas antes de marcar como processado, a mensagem pode ser reenviada — por isso, o consumidor precisa ser idempotente, igual discutimos no post sobre RabbitMQ

---

# ⚠️ Erros comuns

- Publicar a mensagem antes de fazer o `SaveChangesAsync`, reintroduzindo o problema da dupla escrita  
- Não limpar mensagens antigas já processadas, deixando a tabela outbox crescer indefinidamente  
- Esquecer de tornar os consumidores idempotentes, assumindo que outbox garante entrega única  
- Rodar múltiplas instâncias do despachante sem controle de concorrência, processando a mesma mensagem duas vezes ao mesmo tempo  

---

# 📌 Conclusão

- O problema da dupla escrita acontece quando salvar no banco e publicar um evento são operações separadas  
- O padrão Outbox grava o evento na mesma transação do banco, garantindo atomicidade  
- Um processo separado despacha as mensagens pendentes de forma assíncrona  
- A garantia é at-least-once — consumidores precisam ser idempotentes  

👉 Com o padrão Outbox, seu sistema garante que nenhum evento se perde, mesmo quando falhas acontecem no pior momento possível

---

# 🔥 Próximo passo

Agora que suas mensagens nunca se perdem, o próximo nível é:

👉 **Fundamentos do C#: Testes de Integração com WebApplicationFactory**

Aqui você vai aprender a testar sua API inteira, do HTTP ao banco de dados, com testes automatizados de verdade.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
