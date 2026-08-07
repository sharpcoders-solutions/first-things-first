# 🧠 Fundamentos do C#: Task vs ValueTask

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `IAsyncEnumerable<T>` para sequências assíncronas geradas sob demanda  
- Boxing, unboxing e o custo de alocações desnecessárias (post 47)  

Todo método `async` que você escreveu retorna `Task` ou `Task<T>`. Isso funciona bem na esmagadora maioria dos casos — mas existe um custo escondido em cada `Task` criada, e em código de altíssima frequência, esse custo importa.

👉 **Vamos entender quando (e por que) trocar `Task<T>` por `ValueTask<T>`**

---

# 💡 O custo escondido: `Task<T>` é uma classe

```csharp
public async Task<int> ObterValorAsync()
{
    return 42; // parece simples, mas...
}
```

👉 Lembra do post sobre boxing e do post sobre value types vs reference types? `Task<T>` é uma **classe** — toda vez que um método `async` retorna, mesmo que o resultado já esteja disponível de forma síncrona, o runtime pode precisar alocar um objeto `Task<T>` no heap para representar esse resultado

---

# 🎯 O caso comum: caminho síncrono em um método "assíncrono"

```csharp
private readonly Dictionary<int, Produto> _cache = new();

public async Task<Produto> ObterProdutoAsync(int id)
{
    if (_cache.TryGetValue(id, out var produtoEmCache))
        return produtoEmCache; // caminho síncrono: já temos o valor, sem I/O nenhum

    var produto = await _repositorio.BuscarPorIdAsync(id); // caminho assíncrono real
    _cache[id] = produto;
    return produto;
}
```

👉 Quando o item já está em cache, esse método não faz nenhum I/O de verdade — mas ainda assim, sendo `async Task<Produto>`, o compilador pode alocar uma `Task<Produto>` só para embrulhar um valor que já estava pronto. Se esse método for chamado milhões de vezes por segundo (um cache muito "quente"), essas alocações desnecessárias se acumulam

---

# ⚡ `ValueTask<T>`: evitando a alocação no caminho rápido

```csharp
public ValueTask<Produto> ObterProdutoAsync(int id)
{
    if (_cache.TryGetValue(id, out var produtoEmCache))
        return new ValueTask<Produto>(produtoEmCache); // sem alocação no heap

    return BuscarEArmazenarAsync(id);
}

private async ValueTask<Produto> BuscarEArmazenarAsync(int id)
{
    var produto = await _repositorio.BuscarPorIdAsync(id);
    _cache[id] = produto;
    return produto;
}
```

👉 **`ValueTask<T>` = um `struct` que pode representar tanto um resultado síncrono já disponível (sem alocação) quanto um resultado assíncrono real (delegando para uma `Task<T>` internamente quando necessário)**

Lembra da diferença entre value types e reference types (post 46)? Esse é exatamente o motivo pelo qual `ValueTask<T>` existe: evitar a alocação de heap de uma `Task<T>` no caminho onde o valor já está disponível

---

# ⚠️ A restrição mais importante: `ValueTask<T>` só pode ser aguardado uma vez

```csharp
var valueTask = ObterProdutoAsync(42);

var produto1 = await valueTask; // ✅ ok
var produto2 = await valueTask; // ❌ comportamento indefinido — NÃO faça isso
```

👉 **Regra absoluta: nunca faça `await` em um `ValueTask<T>` mais de uma vez, e nunca acesse `.Result` nele antes de dar `await`.** Diferente de `Task<T>`, que pode ser aguardada múltiplas vezes ou consultada livremente, `ValueTask<T>` pode reutilizar internamente um objeto reciclado — usá-la incorretamente causa bugs sutis e difíceis de depurar

Se você precisa aguardar o mesmo resultado mais de uma vez, converta explicitamente: `var task = valueTask.AsTask();`

---

# ⚖️ Quando usar cada um: a regra prática

| | `Task<T>` | `ValueTask<T>` |
|---|---|---|
| API pública, uso geral | ✅ padrão recomendado | Evite, exceto com forte justificativa |
| Caminho frequentemente síncrono (cache, buffers) | Alocações desnecessárias | ✅ ideal |
| Precisa ser aguardado múltiplas vezes | ✅ suporta naturalmente | ❌ requer `.AsTask()` |
| Simplicidade de uso | ✅ mais simples | Mais regras a seguir |

👉 **Regra prática: comece sempre com `Task<T>`. Só migre para `ValueTask<T>` depois de medir (lembra do `BenchmarkDotNet`, do post sobre performance?) e confirmar que as alocações de `Task<T>` são um gargalo real e mensurável** — na maioria dos métodos assíncronos de uma aplicação de negócio comum, a diferença é irrelevante perto do custo de I/O real (banco de dados, rede)

---

# 🔍 Onde `ValueTask<T>` já é usado no próprio .NET

```csharp
// Stream.ReadAsync retorna ValueTask<int> desde .NET Core 2.1+
public override ValueTask<int> ReadAsync(Memory<byte> buffer, CancellationToken cancellationToken = default)
```

👉 APIs de baixíssimo nível do próprio .NET (como `Stream`, do post sobre I/O de arquivos) usam `ValueTask<T>` justamente porque são chamadas em loops de altíssima frequência, onde o caminho síncrono (dados já em buffer) é extremamente comum

---

# ⚠️ Erros comuns

- Trocar todo `Task<T>` por `ValueTask<T>` "por precaução", sem medir se existe ganho real, só adicionando complexidade  
- Aguardar o mesmo `ValueTask<T>` mais de uma vez, causando comportamento indefinido  
- Guardar um `ValueTask<T>` em uma variável e usá-la depois de um tempo, quando o objeto reciclado por trás pode já ter sido reutilizado  
- Expor `ValueTask<T>` em APIs públicas de bibliotecas sem documentar claramente a restrição de "aguardar só uma vez" para quem consome  

---

# 📌 Conclusão

- `Task<T>` é uma classe — pode envolver alocação no heap mesmo para resultados síncronos  
- `ValueTask<T>` é um `struct` que evita essa alocação no caminho síncrono  
- `ValueTask<T>` só pode ser aguardado uma única vez — violar isso causa bugs sutis  
- Use `Task<T>` como padrão; migre para `ValueTask<T>` só com medição real de que vale a pena  

👉 Você já entende o custo de criar uma `Task`, mas há um nível ainda mais profundo: o que realmente acontece quando você escreve `await` — e como criar seus próprios tipos aguardáveis

---

# 🔥 Próximo passo

Agora que você sabe otimizar alocações em código assíncrono de alta frequência, o próximo nível é:

👉 **Fundamentos do C#: Custom Awaiters e o Awaitable Pattern**

Aqui você vai aprender o que realmente torna um tipo "aguardável" com `await`, e como criar seus próprios awaiters customizados.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
