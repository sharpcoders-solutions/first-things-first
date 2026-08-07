# 🧠 Fundamentos do C#: Custom Awaiters e o Awaitable Pattern

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `Task<T>` vs `ValueTask<T>` e o custo de alocações em código assíncrono  
- Operator overloading e como ensinar o compilador a entender novos comportamentos (post 56)  

Você usa `await` desde os primeiros posts sobre programação assíncrona, sempre em cima de `Task` ou `Task<T>`. Mas você sabia que `await` não é mágica exclusiva de `Task`? Qualquer tipo pode ser "aguardável", desde que siga um padrão específico — nem precisa implementar interface nenhuma.

👉 **Vamos entender o que realmente torna algo aguardável com `await`**

---

# 💡 O segredo: `await` não exige uma interface, exige um padrão

```csharp
// await X funciona se X tiver um método GetAwaiter()
// que retorna algo com IsCompleted, GetResult() e que implemente INotifyCompletion
```

👉 **Diferente do que muita gente assume, `await` não exige que o tipo implemente `Task` ou qualquer interface específica — o compilador procura, em tempo de compilação, por um padrão de métodos conhecido como "awaitable pattern"**

Isso é chamado de **duck typing estrutural**: se o tipo "parece" aguardável (tem os métodos certos), o compilador aceita, mesmo sem herança ou interface formal — o mesmo espírito de flexibilidade que você já viu no post sobre operator overloading, onde o compilador reconhece um padrão de assinatura, não um tipo específico

---

# 🏗️ Construindo um awaiter do zero

```csharp
public class TarefaCustomizada
{
    public Awaiter GetAwaiter() => new Awaiter(this);

    public class Awaiter : INotifyCompletion
    {
        private readonly TarefaCustomizada _tarefa;

        public Awaiter(TarefaCustomizada tarefa) => _tarefa = tarefa;

        public bool IsCompleted => /* verifica se já terminou */ true;

        public void GetResult() { /* retorna o resultado, ou lança exceção se falhou */ }

        public void OnCompleted(Action continuacao) { /* agenda a continuação */ }
    }
}
```

👉 Os três membros obrigatórios são: **`IsCompleted`** (propriedade `bool`, indica se já terminou), **`GetResult()`** (retorna o resultado, ou `void`), e **`OnCompleted(Action)`** (vindo de `INotifyCompletion`, agenda o que rodar quando terminar). `GetAwaiter()` é o método que conecta seu tipo a esse objeto awaiter

---

# 🎯 Um exemplo prático: aguardar um `SemaphoreSlim` de forma mais natural

```csharp
public static class SemaphoreSlimExtensions
{
    public static SemaphoreSlimAwaiter GetAwaiter(this SemaphoreSlim semaforo) =>
        new SemaphoreSlimAwaiter(semaforo);
}

public readonly struct SemaphoreSlimAwaiter : INotifyCompletion
{
    private readonly SemaphoreSlim _semaforo;

    public SemaphoreSlimAwaiter(SemaphoreSlim semaforo) => _semaforo = semaforo;

    public bool IsCompleted => false;

    public void GetResult() { }

    public void OnCompleted(Action continuacao) =>
        _semaforo.WaitAsync().ContinueWith(_ => continuacao());
}

// Agora isso funciona:
await meuSemaforo; // em vez de: await meuSemaforo.WaitAsync();
```

👉 Lembra do post sobre extension methods? `GetAwaiter` pode ser adicionado como extension method a **qualquer** tipo, mesmo um que você não controla — tornando `SemaphoreSlim` (do post sobre concorrência assíncrona) diretamente aguardável, sem precisar chamar `.WaitAsync()` explicitamente

---

# ⚙️ `Task.GetAwaiter()` por baixo dos panos

```csharp
await minhaTask;

// É, na prática, açúcar sintático aproximado para:
var awaiter = minhaTask.GetAwaiter();
if (!awaiter.IsCompleted)
{
    // pausa a execução aqui, registra a continuação, e retoma quando pronta
}
var resultado = awaiter.GetResult();
```

👉 `Task<T>` e `Task` simplesmente implementam esse mesmo padrão internamente — não existe mágica especial embutida na linguagem só para `Task`. Todo o mecanismo de `async`/`await` que você usa desde os primeiros posts é construído inteiramente em cima desse padrão de três membros

---

# 🚀 `ConfigureAwait(false)`: outro awaiter customizado que você já usa

```csharp
await _httpClient.GetAsync(url).ConfigureAwait(false);
```

👉 `ConfigureAwait(false)` retorna um `ConfiguredTaskAwaitable`, um tipo diferente com seu próprio awaiter, que controla se a continuação deve voltar ao contexto de sincronização original ou não. É outro exemplo de como o padrão awaitable é usado dentro do próprio .NET para variar o comportamento do `await`, sem precisar de uma nova palavra-chave

---

# 🎯 Quando você realmente criaria um awaiter customizado

👉 **Na prática do dia a dia, você quase nunca vai escrever um awaiter do zero — mas entender o padrão explica por que `await` funciona em lugares que parecem "especiais"**

Casos raros onde faz sentido:

- Criar uma API de biblioteca onde `await meuObjeto` é mais natural que `await meuObjeto.AlgumMetodoAsync()`  
- Integrar com APIs de baixo nível (como loops de jogos ou frameworks customizados) que precisam de um comportamento de continuação diferente do padrão  
- Entender bibliotecas de terceiros que expõem tipos "aguardáveis" customizados, como certas APIs de UI  

---

# ⚠️ Erros comuns

- Achar que `await` só funciona com `Task`, sem perceber que é um padrão estrutural, não uma restrição de tipo  
- Criar awaiters customizados para problemas simples, quando `Task`/`ValueTask` já resolveriam com muito menos código  
- Esquecer de implementar `INotifyCompletion` corretamente, quebrando o mecanismo de continuação do `await`  
- Confundir `GetAwaiter()` (que habilita `await`) com `GetEnumerator()` (que habilita `foreach`, do post 55) — são padrões parecidos, mas para propósitos completamente diferentes  

---

# 📌 Conclusão

- `await` funciona em cima de um padrão estrutural (`GetAwaiter`, `IsCompleted`, `GetResult`, `OnCompleted`), não de uma interface obrigatória  
- Qualquer tipo pode se tornar aguardável, mesmo via extension method em um tipo que você não controla  
- `Task<T>` e `ConfigureAwait(false)` são exemplos do próprio .NET usando esse mesmo padrão internamente  
- Criar awaiters customizados é raro na prática, mas entender o padrão explica o funcionamento interno de tudo que você já usa  

👉 Com CancellationToken, IAsyncEnumerable, Task vs ValueTask e agora o awaitable pattern, você completa o quadro mais avançado de programação assíncrona em C# — a base perfeita para voltar ao mundo de comunicação em tempo real

---

# 🔥 Próximo passo

Agora que você entende os bastidores completos do `await`, o próximo nível é:

👉 **Fundamentos do C#: SignalR**

Aqui você vai aprender comunicação em tempo real entre servidor e cliente, para além do modelo request/response tradicional.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
