# 🧠 Fundamentos do C#: Kafka

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Multi-tenancy para isolar clientes na mesma aplicação  
- RabbitMQ (post 41) para mensageria entre serviços  

RabbitMQ é excelente para filas de tarefas e comunicação ponto a ponto. Mas quando o volume é de milhões de eventos por segundo, e você precisa que múltiplos consumidores independentes leiam o **mesmo** stream de eventos, entra o Kafka.

👉 **Vamos aprender Kafka**

---

# 💡 Kafka não é uma fila, é um log distribuído

👉 **Kafka = um log de eventos imutável e ordenado, que múltiplos consumidores podem ler independentemente**

## 🔹 RabbitMQ (post 41): mensagem consumida, some da fila

```
Fila: [msg1, msg2, msg3]
Consumidor lê msg1 → fila: [msg2, msg3] (msg1 se foi)
```

## 🔹 Kafka: mensagem persiste no log

```
Tópico: [evento1, evento2, evento3, evento4, ...]
Consumidor A está no offset 2
Consumidor B está no offset 4
Ambos podem reler o histórico inteiro, de forma independente
```

👉 Isso lembra o Event Sourcing do post 56 — o Kafka é literalmente um log de eventos, e cada consumidor mantém seu próprio ponteiro (offset) de onde parou de ler

---

# 🏗️ Produzindo eventos

```bash
dotnet add package Confluent.Kafka
```

```csharp
var configuracao = new ProducerConfig { BootstrapServers = "localhost:9092" };

using var produtor = new ProducerBuilder<string, string>(configuracao).Build();

await produtor.ProduceAsync("pedidos-criados", new Message<string, string>
{
    Key = pedido.Id.ToString(),
    Value = JsonSerializer.Serialize(pedido)
});
```

👉 A `Key` determina a partição — mensagens com a mesma chave sempre vão para a mesma partição, garantindo ordem para aquele conjunto específico de eventos (por exemplo, todos os eventos de um mesmo pedido)

---

# 🎯 Consumindo eventos

```csharp
var configuracao = new ConsumerConfig
{
    BootstrapServers = "localhost:9092",
    GroupId = "servico-notificacoes",
    AutoOffsetReset = AutoOffsetReset.Earliest
};

using var consumidor = new ConsumerBuilder<string, string>(configuracao).Build();
consumidor.Subscribe("pedidos-criados");

while (true)
{
    var resultado = consumidor.Consume();
    var pedido = JsonSerializer.Deserialize<Pedido>(resultado.Message.Value);

    Console.WriteLine($"Notificando sobre pedido {pedido.Id}");
    consumidor.Commit(resultado);
}
```

👉 O `GroupId` agrupa consumidores — múltiplas instâncias com o mesmo `GroupId` dividem as partições entre si (escalabilidade horizontal), enquanto grupos diferentes leem o mesmo tópico de forma totalmente independente

---

# 🔀 Partições: paralelismo real

```
Tópico "pedidos-criados" com 3 partições:
  Partição 0: [evento1, evento4, evento7, ...]
  Partição 1: [evento2, evento5, evento8, ...]
  Partição 2: [evento3, evento6, evento9, ...]
```

👉 Cada partição pode ser processada em paralelo por consumidores diferentes do mesmo grupo — isso é o que permite ao Kafka escalar para milhões de eventos por segundo, algo que uma fila tradicional (post 41) não foi desenhada para fazer

---

# ⚖️ Kafka vs RabbitMQ: quando usar cada um

## 🔹 RabbitMQ
- Filas de tarefas, comandos que devem ser processados uma vez  
- Roteamento complexo de mensagens (exchanges, routing keys)  
- Volume moderado  

## 🔹 Kafka
- Streams de eventos de alto volume (cliques, logs, métricas, IoT)  
- Múltiplos consumidores independentes precisam reler o mesmo stream  
- Casos que se aproximam de Event Sourcing (post 56) em escala  

---

# ⚠️ Erros comuns

- Usar Kafka para filas de tarefas simples, quando RabbitMQ resolveria com muito menos complexidade operacional  
- Escolher uma `Key` de partição ruim, concentrando todo o volume em uma única partição e perdendo o paralelismo  
- Não commitar o offset corretamente, reprocessando eventos duplicados ou perdendo eventos  
- Subestimar a complexidade operacional do Kafka — ele exige Zookeeper (ou KRaft) e mais infraestrutura que uma fila tradicional  

---

# 📌 Conclusão

- Kafka é um log distribuído, não uma fila tradicional — eventos persistem e podem ser relidos  
- Múltiplos grupos de consumidores leem o mesmo tópico de forma independente  
- Partições permitem paralelismo real, essencial para volumes altíssimos  
- RabbitMQ e Kafka resolvem problemas diferentes — a escolha depende do padrão de consumo, não só do volume  

👉 Com Kafka, sistemas que precisam processar eventos em escala massiva ganham uma infraestrutura desenhada especificamente para isso

---

# 🔥 Próximo passo

Agora que você domina mensageria em escala, o próximo nível é:

👉 **Fundamentos do C#: Arquitetura Orientada a Eventos**

Aqui você vai aprender a conectar tudo que aprendeu sobre eventos — Event Sourcing, Saga, Outbox e Kafka — em uma arquitetura coesa.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
