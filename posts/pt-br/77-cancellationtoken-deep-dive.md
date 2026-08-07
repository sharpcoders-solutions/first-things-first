# 🧠 Fundamentos do C#: CancellationToken em Profundidade

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Options Pattern para configuração fortemente tipada  
- `async`/`await` desde os primeiros posts sobre programação assíncrona  

Você já viu `CancellationToken` como parâmetro em métodos assíncronos ao longo desta trilha, quase sempre passado adiante sem muita explicação. Chegou a hora de entender de verdade como o cancelamento cooperativo funciona em C#.

👉 **Vamos entender `CancellationToken` a fundo**

---

# 💡 O problema: como interromper uma operação em andamento?

```csharp
public async Task<List<Produto>> BuscarProdutosAsync()
{
    // Se o usuário fechar a página no meio dessa busca, como avisamos isso ao código?
    return await _contexto.Produtos.ToListAsync();
}
```

👉 Threads não podem simplesmente ser "mortas" de fora com segurança — isso deixaria recursos em estado inconsistente (arquivos abertos, transações pendentes). C# resolve isso com **cancelamento cooperativo**: quem está executando o trabalho precisa checar periodicamente se foi pedido para parar, e parar voluntariamente

---

# 🏗️ `CancellationTokenSource` e `CancellationToken`

```csharp
using var cts = new CancellationTokenSource();

var tarefa = BuscarProdutosAsync(cts.Token);

// Em algum outro lugar do código, talvez em resposta a um clique de "cancelar"
cts.Cancel();
```

👉 **`CancellationTokenSource` = quem tem o poder de cancelar. `CancellationToken` = o "sinal" somente-leitura que é passado adiante para quem executa o trabalho**

Você nunca cria um `CancellationToken` diretamente — ele sempre vem de um `CancellationTokenSource`, que é quem decide **quando** disparar o cancelamento

---

# 🔍 Verificando o cancelamento dentro do código

```csharp
public async Task ProcessarLoteAsync(List<Pedido> pedidos, CancellationToken cancellationToken)
{
    foreach (var pedido in pedidos)
    {
        cancellationToken.ThrowIfCancellationRequested(); // lança OperationCanceledException se cancelado

        await ProcessarPedidoAsync(pedido, cancellationToken);
    }
}
```

👉 `ThrowIfCancellationRequested()` é o padrão mais comum: verifica o token e, se o cancelamento foi pedido, lança `OperationCanceledException` imediatamente — interrompendo o loop de forma limpa e previsível

---

# ⏱️ Cancelamento por timeout

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));

try
{
    await BuscarProdutosAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("A operação excedeu o tempo limite de 10 segundos");
}
```

👉 Passar um `TimeSpan` direto no construtor do `CancellationTokenSource` agenda o cancelamento automático depois daquele tempo — um jeito simples de implementar timeout sem gerenciar um `Timer` manualmente

---

# 🔗 Combinando múltiplos tokens

```csharp
public async Task ExecutarComTimeoutAsync(CancellationToken tokenExterno)
{
    using var ctsTimeout = new CancellationTokenSource(TimeSpan.FromSeconds(30));
    using var ctsCombinado = CancellationTokenSource.CreateLinkedTokenSource(
        tokenExterno, ctsTimeout.Token);

    await FazerTrabalhoAsync(ctsCombinado.Token);
}
```

👉 `CreateLinkedTokenSource` combina múltiplos tokens em um só — a operação cancela se **qualquer um** dos tokens originais for cancelado. Isso é essencial quando você recebe um token de fora (do ASP.NET Core, por exemplo, quando o cliente HTTP desconecta) e ainda quer aplicar seu próprio timeout interno

---

# 🌐 De onde vêm os tokens na prática: ASP.NET Core

```csharp
[HttpGet]
public async Task<IActionResult> ObterPedidos(CancellationToken cancellationToken)
{
    var pedidos = await _contexto.Pedidos.ToListAsync(cancellationToken);
    return Ok(pedidos);
}
```

👉 O ASP.NET Core injeta automaticamente um `CancellationToken` ligado à conexão HTTP — se o cliente fechar a aba do navegador ou cancelar a requisição, esse token é cancelado, e passá-lo adiante para `ToListAsync` interrompe a query no banco imediatamente, em vez de desperdiçar trabalho para uma resposta que ninguém vai receber

---

# ⚠️ Erros comuns

- Aceitar um `CancellationToken` como parâmetro, mas nunca passá-lo adiante para as chamadas assíncronas internas — o parâmetro vira decoração, sem efeito real  
- Capturar `OperationCanceledException` e tratá-la como um erro genérico, quando ela representa um cancelamento intencional, não uma falha  
- Ignorar o `CancellationToken` injetado pelo ASP.NET Core, continuando a processar uma requisição que o cliente já abandonou  
- Não usar `using` no `CancellationTokenSource`, vazando o `Timer` interno quando um timeout é configurado  

---

# 📌 Conclusão

- Cancelamento em C# é cooperativo — quem executa o trabalho precisa checar o token periodicamente  
- `CancellationTokenSource` dispara o cancelamento; `CancellationToken` é o sinal somente-leitura propagado  
- `ThrowIfCancellationRequested()` interrompe a execução lançando `OperationCanceledException`  
- `CreateLinkedTokenSource` combina múltiplos tokens, cancelando se qualquer um deles for cancelado  
- O ASP.NET Core injeta automaticamente um token ligado ao ciclo de vida da requisição HTTP  

👉 Com cancelamento sob controle, o próximo passo é ver como consumir sequências de dados assíncronas — combinando tudo que você já sabe sobre iteradores com o mundo `async`

---

# 🔥 Próximo passo

Agora que você domina cancelamento cooperativo, o próximo nível é:

👉 **Fundamentos do C#: IAsyncEnumerable e Async Streams**

Aqui você vai aprender a combinar `yield return` com `async`, criando sequências que produzem valores assíncronos, um de cada vez.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
