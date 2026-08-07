# 🧠 Fundamentos do C#: Event Sourcing — Introdução

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Observabilidade para rastrear requisições através de serviços  
- DDD, com entidades, agregados e eventos de domínio  

No modelo tradicional, o banco guarda só o **estado atual**: um pedido está "Enviado", ponto final. Você perdeu toda a história de como ele chegou lá. Event Sourcing resolve isso guardando cada mudança como um evento imutável.

👉 **Vamos aprender Event Sourcing**

---

# 💡 O que é Event Sourcing?

👉 **Event Sourcing = guardar a sequência completa de eventos que levaram ao estado atual, em vez de só o estado atual**

## 🔹 Modelo tradicional (CRUD)

```sql
UPDATE Pedidos SET Status = 'Enviado' WHERE Id = 123
```

👉 O `UPDATE` sobrescreve o valor anterior — "Confirmado" e "Pago" já se foram para sempre

## 🔹 Event Sourcing

```csharp
public record PedidoCriado(int PedidoId, DateTime Quando);
public record PedidoConfirmado(int PedidoId, DateTime Quando);
public record PedidoPago(int PedidoId, decimal Valor, DateTime Quando);
public record PedidoEnviado(int PedidoId, string CodigoRastreio, DateTime Quando);
```

👉 Cada mudança de estado vira um evento, gravado e nunca apagado. O estado atual é **derivado** da sequência de eventos — lembra dos records do post 27? Eles são perfeitos para representar eventos imutáveis

---

# 🏗️ Reconstruindo o estado a partir dos eventos

```csharp
public class Pedido
{
    public int Id { get; private set; }
    public string Status { get; private set; } = "Novo";
    public decimal ValorPago { get; private set; }

    public static Pedido ReconstruirDeEventos(IEnumerable<object> eventos)
    {
        var pedido = new Pedido();

        foreach (var evento in eventos)
        {
            pedido.Aplicar(evento);
        }

        return pedido;
    }

    private void Aplicar(object evento)
    {
        switch (evento)
        {
            case PedidoCriado e:
                Id = e.PedidoId;
                Status = "Criado";
                break;
            case PedidoConfirmado:
                Status = "Confirmado";
                break;
            case PedidoPago e:
                Status = "Pago";
                ValorPago = e.Valor;
                break;
            case PedidoEnviado:
                Status = "Enviado";
                break;
        }
    }
}
```

👉 Isso lembra o pattern matching do post 27 — cada evento sabe como transformar o estado, e o estado final é só o resultado de "tocar" todos os eventos em ordem

---

# 📦 Guardando eventos: o Event Store

```csharp
public class ArmazenadorEventos
{
    private readonly List<EventoArmazenado> _eventos = new();

    public void Salvar(int agregadoId, object evento)
    {
        _eventos.Add(new EventoArmazenado(agregadoId, evento, DateTime.UtcNow));
    }

    public IEnumerable<object> ObterEventos(int agregadoId) =>
        _eventos.Where(e => e.AgregadoId == agregadoId).Select(e => e.Evento);
}
```

👉 Na prática, o event store costuma ser um banco especializado (EventStoreDB) ou uma tabela append-only em um banco relacional — nunca fazemos `UPDATE` ou `DELETE`, só `INSERT`

---

# 🎯 As vantagens reais

- **Auditoria completa e gratuita** — você sabe exatamente quando e por que cada mudança aconteceu  
- **Reconstrução de estado em qualquer ponto no tempo** — "como estava o pedido às 14h de ontem?"  
- **Debug de produção** — reproduzir exatamente a sequência de eventos que levou a um bug  

---

# ⚠️ Erros comuns

- Aplicar Event Sourcing em todo o sistema, quando só agregados com histórico valioso (pedidos, transações financeiras) realmente se beneficiam  
- Não versionar os eventos — quando o formato de `PedidoCriado` muda, eventos antigos precisam continuar sendo lidos corretamente  
- Reconstruir o estado do zero a cada leitura sem usar snapshots, tornando agregados com muitos eventos lentos para carregar  
- Subestimar a complexidade — Event Sourcing troca simplicidade de CRUD por poder de auditoria; use onde o histórico realmente importa  

---

# 📌 Conclusão

- Event Sourcing guarda a sequência completa de eventos, não só o estado final  
- O estado atual é derivado aplicando os eventos em ordem  
- O Event Store é append-only — eventos nunca são alterados ou apagados  
- O ganho real é auditoria completa e a capacidade de reconstruir o estado em qualquer ponto do tempo  

👉 Com Event Sourcing, seu sistema para de perguntar só "qual é o estado atual?" e passa a saber responder "como chegamos até aqui?"

---

# 🔥 Próximo passo

Agora que você guarda o histórico completo de mudanças, o próximo nível é:

👉 **Fundamentos do C#: Padrão Saga para Transações Distribuídas**

Aqui você vai aprender a coordenar transações que envolvem múltiplos serviços, sem uma transação de banco tradicional.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
