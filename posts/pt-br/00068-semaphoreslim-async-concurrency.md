# 🧠 Fundamentos do C#: SemaphoreSlim e Concorrência Assíncrona

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `lock` e `Monitor` para proteger seções críticas síncronas  
- Deadlocks e como evitá-los adquirindo travas em ordem consistente  

`lock` resolve sincronização entre threads — mas tente colocar um `await` dentro de um bloco `lock`, e o compilador vai reclamar. Código assíncrono precisa de uma ferramenta diferente para controlar acesso concorrente.

👉 **Vamos aprender `SemaphoreSlim`, a ferramenta certa para concorrência em código `async`**

---

# 💡 Por que `lock` não funciona com `async`

```csharp
private readonly object _trava = new();

public async Task ProcessarAsync()
{
    lock (_trava)
    {
        await FazerAlgoAsync(); // ❌ Erro de compilação: não pode usar 'await' dentro de 'lock'
    }
}
```

👉 `lock` foi projetado para segurar uma trava do início ao fim de um bloco síncrono, na mesma thread. Um método `async` pode **trocar de thread** entre um `await` e o próximo (lembra do post sobre async/await?) — e `Monitor`, por trás do `lock`, está fortemente ligado à noção de "thread que segura a trava". Misturar os dois quebra essa garantia

---

# 🎯 `SemaphoreSlim`: a versão assíncrona de uma trava

```csharp
private readonly SemaphoreSlim _semaforo = new SemaphoreSlim(1, 1);

public async Task ProcessarAsync()
{
    await _semaforo.WaitAsync();
    try
    {
        await FazerAlgoAsync(); // ✅ funciona perfeitamente aqui dentro
    }
    finally
    {
        _semaforo.Release();
    }
}
```

👉 **`SemaphoreSlim` = um contador que permite um número limitado de threads (ou tarefas) acessarem uma seção ao mesmo tempo**

Com `SemaphoreSlim(1, 1)` (contagem inicial 1, máximo 1), você tem o equivalente assíncrono de um `lock`: só uma tarefa por vez passa do `WaitAsync()`. A diferença crucial é que `WaitAsync()` é compatível com `await`, sem prender uma thread do sistema operacional enquanto espera

---

# 🔢 Além de 1: limitando concorrência real

```csharp
private readonly SemaphoreSlim _semaforo = new SemaphoreSlim(3, 3); // no máximo 3 simultâneas

public async Task<Produto> BuscarProdutoAsync(int id)
{
    await _semaforo.WaitAsync();
    try
    {
        return await _httpClient.GetFromJsonAsync<Produto>($"/produtos/{id}");
    }
    finally
    {
        _semaforo.Release();
    }
}
```

👉 Diferente de `lock` (sempre 1 por vez), `SemaphoreSlim` pode permitir **N** operações simultâneas. Isso é exatamente o que você usa para limitar quantas chamadas HTTP simultâneas seu código faz para uma API externa, evitando sobrecarregá-la ou estourar limites de rate limiting (lembra do post 56?)

---

# 🚦 Caso de uso real: limitando chamadas paralelas

```csharp
public async Task<List<Produto>> BuscarVariosAsync(IEnumerable<int> ids)
{
    var semaforo = new SemaphoreSlim(5); // no máximo 5 chamadas HTTP ao mesmo tempo

    var tarefas = ids.Select(async id =>
    {
        await semaforo.WaitAsync();
        try
        {
            return await BuscarProdutoAsync(id);
        }
        finally
        {
            semaforo.Release();
        }
    });

    return (await Task.WhenAll(tarefas)).ToList();
}
```

👉 Sem o semáforo, `Task.WhenAll` (do post sobre async/await) dispararia todas as chamadas de uma vez — para 1000 IDs, seriam 1000 chamadas HTTP simultâneas, provavelmente derrubando o serviço remoto ou estourando um rate limit. O semáforo garante que só 5 rodem por vez, enfileirando o resto naturalmente

---

# ⏱️ `WaitAsync` com timeout e cancelamento

```csharp
public async Task<bool> TentarProcessarAsync(CancellationToken cancellationToken)
{
    if (await _semaforo.WaitAsync(TimeSpan.FromSeconds(5), cancellationToken))
    {
        try
        {
            await FazerAlgoAsync();
            return true;
        }
        finally
        {
            _semaforo.Release();
        }
    }

    return false; // não conseguiu a vaga a tempo
}
```

👉 Assim como `Monitor.TryEnter`, `WaitAsync` aceita um timeout — e também aceita um `CancellationToken` (que você vai explorar em profundidade no próximo bloco desta trilha), permitindo cancelar a espera de fora

---

# ⚖️ `SemaphoreSlim` vs `lock`: quando usar cada um

| | `lock` | `SemaphoreSlim` |
|---|---|---|
| Código síncrono | ✅ ideal | Funciona, mas é mais pesado que necessário |
| Código `async` | ❌ não compila com `await` dentro | ✅ ideal |
| Limitar a N concorrentes (N > 1) | ❌ só permite 1 | ✅ suporta qualquer N |
| Overhead | Muito baixo | Um pouco mais alto |

👉 **Regra prática: use `lock` para seções críticas puramente síncronas. Use `SemaphoreSlim` sempre que precisar de `await` dentro da seção protegida, ou quando precisar limitar a mais de uma operação simultânea**

---

# ⚠️ Erros comuns

- Tentar usar `lock` em volta de código com `await`, batendo de frente com o erro de compilação  
- Esquecer o `Release()` no `finally`, vazando "vagas" do semáforo até ele travar tudo permanentemente  
- Criar um `SemaphoreSlim` novo a cada chamada de método, em vez de reutilizar uma instância compartilhada — isso não protege nada, porque cada chamada teria seu próprio contador independente  
- Usar `SemaphoreSlim(1, 1)` quando um `lock` simples resolveria com menos overhead, em código que nunca precisa de `await` na seção crítica  

---

# 📌 Conclusão

- `lock` não pode conter `await` — a troca de thread quebra a garantia do `Monitor` por trás dele  
- `SemaphoreSlim.WaitAsync()` é o equivalente assíncrono de adquirir uma trava, compatível com `await`  
- Com contagem maior que 1, `SemaphoreSlim` limita concorrência a N operações simultâneas, não só 1  
- `WaitAsync` aceita timeout e `CancellationToken`, assim como `Monitor.TryEnter`  

👉 Com `lock`/`Monitor` para código síncrono e `SemaphoreSlim` para código assíncrono, você tem as duas ferramentas fundamentais para controlar acesso concorrente em qualquer cenário — a base perfeita para voltar ao mundo prático de APIs assíncronas avançadas

---

# 🔥 Próximo passo

Agora que você domina sincronização síncrona e assíncrona, o próximo nível é:

👉 **Fundamentos do C#: Unsafe Code e Ponteiros**

Aqui você vai aprender a sair da segurança gerenciada do C# quando performance extrema exige controle direto de memória.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
