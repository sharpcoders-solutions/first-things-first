# 🧠 Fundamentos do C#: IAsyncEnumerable e Async Streams

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `CancellationToken` para cancelamento cooperativo de operações assíncronas  
- `yield return` e iteradores customizados (post 55)  

Você já sabe usar `yield return` para gerar sequências lazy, e já sabe usar `async`/`await` para operações assíncronas. Mas o que fazer quando você precisa das duas coisas ao mesmo tempo — uma sequência gerada aos poucos, onde cada item exige uma operação assíncrona?

👉 **Vamos aprender `IAsyncEnumerable<T>` e async streams**

---

# 💡 O problema: `IEnumerable<T>` não é compatível com `async`

```csharp
public IEnumerable<Produto> BuscarProdutosPaginado()
{
    int pagina = 0;
    while (true)
    {
        var produtos = _repositorio.BuscarPagina(pagina); // ❌ e se isso precisar ser await?
        if (!produtos.Any()) yield break;

        foreach (var produto in produtos)
            yield return produto;

        pagina++;
    }
}
```

👉 Lembra do post sobre iteradores (55)? `yield return` dentro de um método `IEnumerable<T>` não permite `await` no meio do caminho — o mecanismo de state machine gerado para iteradores síncronos é diferente do gerado para métodos `async`

---

# 🎯 `IAsyncEnumerable<T>`: a combinação dos dois mundos

```csharp
public async IAsyncEnumerable<Produto> BuscarProdutosPaginadoAsync()
{
    int pagina = 0;
    while (true)
    {
        var produtos = await _repositorio.BuscarPaginaAsync(pagina); // ✅ await funciona aqui
        if (!produtos.Any()) yield break;

        foreach (var produto in produtos)
            yield return produto;

        pagina++;
    }
}
```

👉 **`async IAsyncEnumerable<T>` = um método que combina `yield return` (geração lazy) com `await` (operações assíncronas), produzindo valores um de cada vez, de forma assíncrona**

O compilador gera uma state machine que suporta ambos os comportamentos simultaneamente — algo impossível de expressar antes desse recurso existir

---

# 🔄 Consumindo com `await foreach`

```csharp
await foreach (var produto in BuscarProdutosPaginadoAsync())
{
    Console.WriteLine(produto.Nome);
}
```

👉 **`await foreach` = a versão assíncrona do `foreach`, feita especificamente para consumir `IAsyncEnumerable<T>`**

Cada iteração espera (`await`) a próxima página ser buscada antes de continuar — sem bloquear a thread enquanto isso acontece, exatamente como qualquer outro `await` que você já usa

---

# 🚫 Cancelamento em async streams

```csharp
public async IAsyncEnumerable<Produto> BuscarProdutosPaginadoAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    int pagina = 0;
    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        var produtos = await _repositorio.BuscarPaginaAsync(pagina, cancellationToken);
        if (!produtos.Any()) yield break;

        foreach (var produto in produtos)
            yield return produto;

        pagina++;
    }
}

// No consumo:
await foreach (var produto in BuscarProdutosPaginadoAsync(cts.Token))
{
    // ...
}
```

👉 Lembra do post anterior sobre `CancellationToken`? O atributo `[EnumeratorCancellation]` é necessário porque, sem ele, o compilador não sabe automaticamente conectar o token passado no `await foreach` ao parâmetro do método gerador

---

# 🌐 Caso de uso real: streaming de resultados de uma API

```csharp
[HttpGet("pedidos/stream")]
public async IAsyncEnumerable<PedidoDto> StreamPedidosAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken)
{
    await foreach (var pedido in _contexto.Pedidos.AsAsyncEnumerable().WithCancellation(cancellationToken))
    {
        yield return new PedidoDto(pedido.Id, pedido.Valor);
    }
}
```

👉 O ASP.NET Core suporta `IAsyncEnumerable<T>` como retorno direto de um endpoint — os resultados são serializados e enviados ao cliente **conforme são gerados**, em vez de esperar a query inteira terminar e materializar tudo em uma lista antes de responder. Isso reduz o tempo até o primeiro byte de resposta, especialmente valioso para grandes volumes de dados

---

# ⚖️ `IAsyncEnumerable<T>` vs `Task<List<T>>`

```csharp
// ❌ Espera TUDO terminar antes de retornar qualquer coisa
public async Task<List<Produto>> BuscarTodosAsync()
{
    var todos = new List<Produto>();
    int pagina = 0;
    while (true)
    {
        var produtos = await _repositorio.BuscarPaginaAsync(pagina);
        if (!produtos.Any()) break;
        todos.AddRange(produtos);
        pagina++;
    }
    return todos;
}

// ✅ Entrega cada item assim que fica disponível
public async IAsyncEnumerable<Produto> BuscarTodosStreamAsync() { /* ... */ }
```

👉 **Regra prática: use `Task<List<T>>` quando você precisa de todos os resultados de uma vez para processá-los. Use `IAsyncEnumerable<T>` quando o consumidor pode (e deve) começar a processar itens antes que todos estejam disponíveis** — a mesma filosofia de lazy evaluation do post sobre `yield return`, agora aplicada ao mundo assíncrono

---

# ⚠️ Erros comuns

- Materializar um `IAsyncEnumerable<T>` inteiro com `.ToListAsync()` cedo demais, perdendo o benefício de streaming que ele oferece  
- Esquecer o atributo `[EnumeratorCancellation]`, fazendo com que o cancelamento não se propague corretamente para dentro do gerador  
- Usar `IAsyncEnumerable<T>` para sequências pequenas e já totalmente disponíveis, onde `Task<List<T>>` seria mais simples e igualmente eficiente  
- Misturar `foreach` comum (sem `await`) com um `IAsyncEnumerable<T>`, gerando erro de compilação por incompatibilidade de tipos  

---

# 📌 Conclusão

- `IAsyncEnumerable<T>` combina `yield return` com `await`, algo impossível com `IEnumerable<T>` comum  
- `await foreach` consome async streams sem bloquear a thread entre itens  
- `[EnumeratorCancellation]` conecta o `CancellationToken` do consumo ao gerador  
- O ASP.NET Core suporta `IAsyncEnumerable<T>` como retorno direto de endpoint, fazendo streaming real da resposta  

👉 Com async streams dominados, o próximo passo é entender uma otimização específica de `Task` — quando até a alocação de um objeto `Task` pode ser custosa demais para código extremamente sensível a performance

---

# 🔥 Próximo passo

Agora que você sabe combinar iteração lazy com código assíncrono, o próximo nível é:

👉 **Fundamentos do C#: Task vs ValueTask**

Aqui você vai aprender quando trocar `Task<T>` por `ValueTask<T>` para eliminar alocações desnecessárias em código assíncrono de alta frequência.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
