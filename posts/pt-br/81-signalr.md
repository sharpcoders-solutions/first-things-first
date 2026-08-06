# 🧠 Fundamentos do C#: SignalR

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- GraphQL como alternativa ao REST tradicional  
- Todo o modelo request/response desde o post 31 — o cliente sempre pergunta, o servidor sempre responde  

E se o servidor precisar avisar o cliente sobre algo, sem esperar uma pergunta? Um chat, uma notificação de pedido atualizado, um placar ao vivo — nada disso encaixa bem no request/response. SignalR resolve isso.

👉 **Vamos aprender SignalR**

---

# 💡 O problema do request/response tradicional

```csharp
// ❌ O cliente precisa ficar perguntando repetidamente
setInterval(async () => {
    const resposta = await fetch('/pedidos/123/status');
    // verifica se mudou...
}, 5000);
```

👉 Polling (perguntar repetidamente) desperdiça requisições e ainda tem atraso — o cliente só descobre a mudança na próxima verificação, não no instante em que ela acontece

---

# 🏗️ Configurando um Hub do SignalR

```bash
dotnet add package Microsoft.AspNetCore.SignalR
```

```csharp
public class PedidosHub : Hub
{
    public async Task EntrarNoGrupo(string pedidoId)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, $"pedido-{pedidoId}");
    }
}
```

```csharp
// Program.cs
builder.Services.AddSignalR();

// ...

app.MapHub<PedidosHub>("/hubs/pedidos");
```

👉 Um `Hub` mantém conexões abertas (via WebSockets, com fallback automático) — diferente de um controller REST (post 31), que responde e encerra a conexão, o Hub mantém o canal aberto para o servidor falar quando quiser

---

# 🎯 Enviando atualizações do servidor para o cliente

```csharp
public class ServicoPedidos
{
    private readonly IHubContext<PedidosHub> _hubContext;

    public async Task AtualizarStatus(int pedidoId, string novoStatus)
    {
        await _repositorio.AtualizarStatus(pedidoId, novoStatus);

        await _hubContext.Clients
            .Group($"pedido-{pedidoId}")
            .SendAsync("StatusAtualizado", novoStatus);
    }
}
```

👉 Lembra do Outbox (post 58) e da arquitetura orientada a eventos (post 79)? Esse é o mesmo espírito — quando algo acontece no servidor, ele **empurra** a notificação, em vez de esperar o cliente perguntar

---

# 🔍 Consumindo no lado do cliente

```javascript
const conexao = new signalR.HubConnectionBuilder()
    .withUrl("/hubs/pedidos")
    .build();

conexao.on("StatusAtualizado", (novoStatus) => {
    console.log(`Pedido atualizado: ${novoStatus}`);
});

await conexao.start();
await conexao.invoke("EntrarNoGrupo", "123");
```

👉 O cliente se registra em um grupo específico (o pedido 123), e recebe atualizações em tempo real só daquele pedido — sem precisar perguntar, sem polling

---

# 🔄 Transportes: WebSockets com fallback automático

```
1ª tentativa: WebSockets (bidirecional, mais eficiente)
2ª tentativa: Server-Sent Events (se WebSockets não disponível)
3ª tentativa: Long Polling (compatibilidade máxima)
```

👉 O SignalR escolhe automaticamente o melhor transporte disponível — você escreve o código uma vez, e ele se adapta ao ambiente do cliente (proxy corporativo bloqueando WebSockets, navegador antigo, etc.)

---

# ⚖️ SignalR vs os padrões anteriores

## 🔹 REST (post 31)
- Cliente pergunta, servidor responde — ideal para operações pontuais  

## 🔹 GraphQL (post 80)
- Cliente pergunta com precisão sobre a estrutura de dados — ainda request/response  

## 🔹 SignalR
- Servidor empurra dados sem o cliente perguntar — ideal para tempo real  

👉 Eles não competem entre si — uma aplicação real frequentemente combina os três: REST/GraphQL para operações normais, SignalR só para os fluxos que precisam de atualização instantânea

---

# ⚠️ Erros comuns

- Usar SignalR para tudo, quando a maioria das operações não precisa de tempo real — REST continua sendo mais simples para isso  
- Não escalar corretamente com múltiplas instâncias — SignalR precisa de um backplane (Redis) para sincronizar conexões entre servidores diferentes  
- Esquecer de limpar grupos/conexões ao desconectar, acumulando estado obsoleto  
- Enviar dados sensíveis sem autenticação no Hub — lembre-se do JWT (post 34), Hubs também precisam de autorização  

---

# 📌 Conclusão

- SignalR permite que o servidor empurre dados para o cliente, sem polling  
- Hubs mantêm conexões abertas via WebSockets, com fallback automático  
- Grupos permitem enviar atualizações só para clientes interessados em um recurso específico  
- SignalR complementa REST/GraphQL, não os substitui — cada um resolve um padrão de comunicação diferente  

👉 Com SignalR, sua aplicação ganha um canal de comunicação verdadeiramente bidirecional, essencial para experiências em tempo real

---

# 🔥 Próximo passo

Agora que você domina comunicação em tempo real, o próximo nível é:

👉 **Fundamentos do C#: Blazor — Introdução**

Aqui você vai aprender a construir interfaces web inteiras usando C# no lugar de JavaScript.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
