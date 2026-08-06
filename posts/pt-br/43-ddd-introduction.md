# 🧠 Fundamentos do C#: Domain-Driven Design (DDD) — Introdução

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- CQRS separando comandos de consultas  
- Clean Architecture organizando o código em camadas  

Você já sabe **onde** colocar cada tipo de código. DDD é sobre uma pergunta diferente e mais profunda: como garantir que suas classes realmente representem as regras do negócio, e não só campos e métodos soltos?

👉 **Vamos conhecer os conceitos fundamentais de Domain-Driven Design**

---

# 💡 O que é DDD?

👉 **DDD = uma abordagem para modelar software que espelha, o mais fielmente possível, a linguagem e as regras do negócio real**

A ideia central: converse com quem entende do negócio (não só de código), e use os **mesmos termos** que eles usam nas suas classes, métodos e nomes de variáveis. Isso é chamado de **Linguagem Ubíqua** — todo mundo, técnico ou não, fala a mesma língua sobre o sistema.

---

# 🧱 Entidades: identidade importa

```csharp
public class Pedido
{
    public int Id { get; private set; } // identidade

    public string Cliente { get; private set; }
    public decimal Total { get; private set; }
}
```

👉 **Entidade = um objeto definido pela sua identidade, não pelos seus valores**

Dois pedidos com o mesmo cliente e o mesmo total ainda são **pedidos diferentes** se tiverem IDs diferentes. Você já viu essa distinção no post sobre records: entidades usam igualdade por identidade, diferente de records, que usam igualdade por valor

---

# 💎 Value Objects: quando só o valor importa

```csharp
public record Endereco(string Rua, string Cidade, string Cep);

public record Dinheiro(decimal Valor, string Moeda)
{
    public static Dinheiro operator +(Dinheiro a, Dinheiro b)
    {
        if (a.Moeda != b.Moeda)
            throw new InvalidOperationException("Não é possível somar moedas diferentes");

        return new Dinheiro(a.Valor + b.Valor, a.Moeda);
    }
}
```

👉 **Value Object = um objeto definido inteiramente pelo seu valor, sem identidade própria**

Dois `Endereco` com os mesmos dados **são o mesmo endereço**, não importa onde foram criados. Aqui, `record` é a ferramenta perfeita — igualdade por valor é exatamente o comportamento que um Value Object precisa

---

# 🛡️ Agregados: protegendo a consistência

```csharp
public class Pedido
{
    private readonly List<ItemPedido> _itens = new();
    public IReadOnlyList<ItemPedido> Itens => _itens.AsReadOnly();

    public decimal Total => _itens.Sum(i => i.Subtotal);

    public void AdicionarItem(Produto produto, int quantidade)
    {
        if (quantidade <= 0)
            throw new ArgumentException("Quantidade deve ser maior que zero");

        _itens.Add(new ItemPedido(produto, quantidade));
    }

    public void RemoverItem(int itemId)
    {
        var item = _itens.FirstOrDefault(i => i.Id == itemId);
        if (item is null)
            throw new InvalidOperationException("Item não encontrado no pedido");

        _itens.Remove(item);
    }
}
```

👉 **Agregado = um conjunto de objetos relacionados, tratados como uma única unidade de consistência, com uma "raiz" (aggregate root) controlando o acesso**

`Pedido` é a raiz do agregado. Ninguém adiciona um `ItemPedido` diretamente na lista de fora — tudo passa pelos métodos de `Pedido`, que garantem que as regras (quantidade positiva, item existente para remover) sejam sempre respeitadas

👉 Isso é encapsulamento (do post sobre classes e objetos) elevado ao nível de um conjunto inteiro de objetos, não só de uma classe isolada

---

# ⚙️ Domain Services: quando a regra não pertence a uma única entidade

```csharp
public class ServicoDeTransferencia
{
    public void Transferir(ContaBancaria origem, ContaBancaria destino, decimal valor)
    {
        origem.Sacar(valor);
        destino.Depositar(valor);
    }
}
```

👉 Algumas regras de negócio envolvem **mais de uma entidade** e não pertencem naturalmente a nenhuma delas sozinha. Um Domain Service existe só para isso — sem guardar estado, apenas orquestrando uma regra que atravessa múltiplos objetos

---

# 📣 Domain Events: comunicando o que aconteceu

```csharp
public record PedidoCriadoEvento(int PedidoId, string Cliente);

public class Pedido
{
    public void Confirmar()
    {
        Status = "Confirmado";
        // dispara o evento para quem quiser reagir (enviar e-mail, atualizar estoque...)
    }
}
```

👉 Lembra dos `events` do post sobre delegates? Domain Events aplicam essa mesma ideia ao domínio: "algo importante aconteceu" (`PedidoCriado`, `PagamentoConfirmado`), e outras partes do sistema reagem sem que a entidade precise conhecer quem está ouvindo — a mesma filosofia da mensageria que você viu no post sobre RabbitMQ, aqui aplicada dentro do próprio domínio

---

# 🗺️ Bounded Context: nem tudo é um sistema só

👉 **Bounded Context = um limite claro dentro do qual um modelo de domínio específico faz sentido**

A palavra "Cliente" pode significar coisas diferentes no contexto de **Vendas** (quem compra) e no contexto de **Suporte** (quem abre chamados). Em vez de forçar uma única classe `Cliente` gigante que serve para tudo, DDD reconhece que cada contexto pode ter seu próprio modelo, focado no que importa para aquele pedaço do sistema.

👉 Isso conecta diretamente com o que você aprendeu sobre microsserviços: cada Bounded Context é um candidato natural a se tornar um serviço independente

---

# ⚠️ Erros comuns

- Criar um "Anemic Domain Model": entidades que são só `get`/`set`, com toda a lógica de negócio espalhada em serviços externos — isso é o oposto do que DDD propõe  
- Aplicar DDD tático (Entidades, Value Objects, Agregados) sem entender o DDD estratégico (Linguagem Ubíqua, Bounded Context) por trás  
- Modelar agregados gigantes, tentando colocar o sistema inteiro dentro de uma única unidade de consistência  
- Usar DDD em domínios simples, tipo CRUD, onde a complexidade extra não traz benefício real  

---

# 📌 Conclusão

- Entidades têm identidade; Value Objects são definidos só pelo valor  
- Agregados protegem a consistência através de uma raiz que controla o acesso  
- Domain Services lidam com regras que atravessam múltiplas entidades  
- Domain Events comunicam mudanças importantes sem acoplar quem dispara a quem reage  
- Bounded Context reconhece que o mesmo termo pode significar coisas diferentes em partes diferentes do sistema  

👉 Com DDD, seu código para de ser só uma estrutura técnica e passa a contar, de verdade, a história do negócio que ele representa

---

# 🔥 Próximo passo

Agora que você sabe modelar domínios complexos, o próximo nível é:

👉 **Fundamentos do C#: Microsserviços — Introdução e Quando Usar**

Aqui você vai aprender quando (e quando não) vale a pena dividir uma aplicação em serviços independentes.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
