# 🧠 Fundamentos do C#: Channels

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- ArrayPool e Object Pooling para reduzir alocações  
- RabbitMQ (post 41) para mensageria entre serviços diferentes  

RabbitMQ resolve comunicação **entre processos**. Mas e quando você só precisa coordenar um produtor e um consumidor **dentro do mesmo processo**, sem a complexidade de uma fila externa? Channels resolvem exatamente isso.

👉 **Vamos aprender Channels**

---

# 💡 O problema: coordenar produtor e consumidor

```csharp
// ❌ Sem coordenação adequada, corre risco de condição de corrida
var fila = new Queue<Pedido>();

// Thread produtora
fila.Enqueue(pedido); // não é thread-safe!

// Thread consumidora
var proximo = fila.Dequeue(); // pode falhar concorrentemente
```

👉 Lembra do post sobre threads e concorrência (post 7)? Uma `Queue<T>` comum não é thread-safe — múltiplas threads acessando ao mesmo tempo geram condições de corrida

---

# 🏗️ Channel: uma fila thread-safe com backpressure

```csharp
using System.Threading.Channels;

var canal = Channel.CreateUnbounded<Pedido>();

// Produtor
await canal.Writer.WriteAsync(new Pedido(id: 1));
await canal.Writer.WriteAsync(new Pedido(id: 2));
canal.Writer.Complete();

// Consumidor
await foreach (var pedido in canal.Reader.ReadAllAsync())
{
    Console.WriteLine($"Processando pedido {pedido.Id}");
}
```

👉 `Channel<T>` é seguro para múltiplos produtores e consumidores simultâneos, e integra nativamente com `async`/`await` (post 26) — sem precisar de locks manuais

---

# 🎯 Channels limitados: controlando a pressão

```csharp
var canal = Channel.CreateBounded<Pedido>(new BoundedChannelOptions(100)
{
    FullMode = BoundedChannelFullMode.Wait
});
```

👉 Um canal ilimitado (`Unbounded`) pode crescer indefinidamente se o produtor for mais rápido que o consumidor, consumindo memória sem limite. Um canal `Bounded` aplica **backpressure**: se o buffer encher, o produtor espera até haver espaço — o mesmo princípio de resiliência que discutimos no post sobre Polly

---

# 🔄 Um caso real: pipeline produtor-consumidor

```csharp
public class ProcessadorDePedidos : BackgroundService
{
    private readonly Channel<Pedido> _canal = Channel.CreateBounded<Pedido>(50);

    public async Task EnfileirarPedido(Pedido pedido)
    {
        await _canal.Writer.WriteAsync(pedido);
    }

    protected override async Task ExecuteAsync(CancellationToken cancellationToken)
    {
        await foreach (var pedido in _canal.Reader.ReadAllAsync(cancellationToken))
        {
            await ProcessarPedido(pedido);
        }
    }
}
```

👉 Isso é o mesmo `BackgroundService` do post sobre Hangfire — mas ao invés de um job agendado, o processamento reage a itens chegando pelo canal, em tempo real, dentro do mesmo processo

---

# ⚖️ Channels vs RabbitMQ

## 🔹 Channels
- Dentro do mesmo processo, sem infraestrutura externa  
- Perde os dados se o processo cair — sem persistência  
- Ideal para coordenar trabalho concorrente dentro de uma única aplicação  

## 🔹 RabbitMQ (post 41)
- Entre processos e serviços diferentes  
- Persiste mensagens, sobrevive a reinícios  
- Necessário quando múltiplos serviços precisam se comunicar  

👉 Channels não substituem RabbitMQ — eles resolvem um problema menor e mais local: coordenação assíncrona dentro de um único processo

---

# ⚠️ Erros comuns

- Usar `Unbounded` sem necessidade, arriscando consumo de memória sem controle  
- Esquecer de chamar `Writer.Complete()`, fazendo o `ReadAllAsync` nunca terminar  
- Usar Channels quando o problema real exige persistência entre processos — nesse caso, RabbitMQ ou Outbox (post 58) são as ferramentas certas  
- Não tratar exceções dentro do loop consumidor, derrubando todo o processamento por causa de um item problemático  

---

# 📌 Conclusão

- Channels coordenam produtores e consumidores dentro do mesmo processo, de forma thread-safe  
- Diferente de uma `Queue<T>` comum, integram nativamente com `async`/`await`  
- Canais `Bounded` aplicam backpressure, evitando consumo descontrolado de memória  
- Channels resolvem um problema mais local que RabbitMQ, sem exigir infraestrutura externa  

👉 Com Channels, você coordena fluxos de trabalho concorrentes dentro da sua aplicação sem reinventar sincronização manual com locks

---

# 🔥 Próximo passo

Agora que você sabe coordenar produtores e consumidores, o próximo nível é:

👉 **Fundamentos do C#: Programação Funcional em C#**

Aqui você vai aprender conceitos funcionais que o C# incorporou ao longo dos anos: imutabilidade, funções puras e composição.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
