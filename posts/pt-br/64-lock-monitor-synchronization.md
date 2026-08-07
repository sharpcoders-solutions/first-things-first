# 🧠 Fundamentos do C#: Lock, Monitor e Sincronização

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como empacotar e publicar seu próprio código como pacote NuGet  
- Coleções concorrentes para dados compartilhados entre threads (post 60)  

Coleções concorrentes resolvem um problema específico: estruturas de dados. Mas o que fazer quando a "seção crítica" que precisa de proteção não é uma coleção, e sim uma sequência arbitrária de operações no seu próprio código? É aí que entram `lock` e `Monitor`.

👉 **Vamos entender sincronização de threads a fundo, com `lock` e `Monitor`**

---

# 💡 O problema: race conditions em código comum

```csharp
public class ContaBancaria
{
    public decimal Saldo { get; private set; }

    public void Depositar(decimal valor)
    {
        var saldoAtual = Saldo;      // lê
        Thread.Sleep(1);             // simula processamento
        Saldo = saldoAtual + valor;  // escreve
    }
}

var conta = new ContaBancaria();
Parallel.For(0, 1000, i => conta.Depositar(10));

Console.WriteLine(conta.Saldo); // ❌ raramente é 10000
```

👉 **Race condition = quando o resultado final depende da ordem exata em que threads executam, de forma imprevisível**

Duas threads podem ler o mesmo `saldoAtual`, ambas somarem `10`, e ambas escreverem o mesmo resultado — um depósito inteiro se perde, sem nenhum erro ou exceção visível

---

# 🔐 `lock`: a forma mais simples de proteger uma seção crítica

```csharp
public class ContaBancaria
{
    private readonly object _trava = new object();
    public decimal Saldo { get; private set; }

    public void Depositar(decimal valor)
    {
        lock (_trava)
        {
            var saldoAtual = Saldo;
            Thread.Sleep(1);
            Saldo = saldoAtual + valor;
        }
    }
}
```

👉 **`lock` = garante que só uma thread por vez execute o bloco protegido; qualquer outra thread que tentar entrar espera até a primeira sair**

O objeto passado para `lock` (`_trava`) não importa pelo seu conteúdo — importa só como um "cadeado" compartilhado. Por convenção, use um `object` privado e dedicado só para essa finalidade, nunca `this` ou um objeto público que código externo também possa travar

---

# ⚙️ O que `lock` realmente é: açúcar sintático para `Monitor`

```csharp
// Isto:
lock (_trava)
{
    // seção crítica
}

// É açúcar sintático para isto:
Monitor.Enter(_trava);
try
{
    // seção crítica
}
finally
{
    Monitor.Exit(_trava);
}
```

👉 `Monitor.Enter`/`Monitor.Exit` é o mecanismo de baixo nível por trás do `lock`. O `try`/`finally` gerado automaticamente é crucial: mesmo que uma exceção aconteça dentro da seção crítica, `Monitor.Exit` sempre é chamado, liberando a trava para outras threads

---

# ⏱️ `Monitor.TryEnter`: evitando esperar para sempre

```csharp
if (Monitor.TryEnter(_trava, TimeSpan.FromSeconds(2)))
{
    try
    {
        // seção crítica
    }
    finally
    {
        Monitor.Exit(_trava);
    }
}
else
{
    Console.WriteLine("Não conseguiu adquirir a trava a tempo — seguindo em frente");
}
```

👉 Diferente de `lock`, que espera indefinidamente, `Monitor.TryEnter` com timeout permite desistir depois de um tempo — útil para evitar que uma thread trave para sempre esperando um recurso que nunca libera

---

# 🔔 `Monitor.Wait` e `Monitor.Pulse`: coordenando threads entre si

```csharp
public class FilaLimitada<T>
{
    private readonly Queue<T> _itens = new();
    private readonly int _capacidade;
    private readonly object _trava = new();

    public FilaLimitada(int capacidade) => _capacidade = capacidade;

    public void Adicionar(T item)
    {
        lock (_trava)
        {
            while (_itens.Count >= _capacidade)
                Monitor.Wait(_trava); // libera a trava e espera ser notificado

            _itens.Enqueue(item);
            Monitor.PulseAll(_trava); // acorda quem estava esperando
        }
    }

    public T Remover()
    {
        lock (_trava)
        {
            while (_itens.Count == 0)
                Monitor.Wait(_trava);

            var item = _itens.Dequeue();
            Monitor.PulseAll(_trava);
            return item;
        }
    }
}
```

👉 `Monitor.Wait` libera a trava temporariamente e coloca a thread para dormir até alguém chamar `Monitor.Pulse`/`PulseAll` — um padrão clássico para implementar filas produtor-consumidor manualmente. Na prática, você raramente precisa disso: `Channel<T>` (que você vai ver mais adiante nesta trilha) resolve o mesmo problema com uma API muito mais simples

---

# ⚠️ Deadlocks: quando duas travas se esperam mutuamente

```csharp
// Thread 1
lock (travaA)
{
    lock (travaB) { /* ... */ }
}

// Thread 2
lock (travaB)
{
    lock (travaA) { /* ... */ } // ❌ deadlock se as duas threads chegarem aqui ao mesmo tempo
}
```

👉 **Regra de ouro: sempre adquira múltiplas travas na mesma ordem, em todo o código.** Se a Thread 1 sempre pega `travaA` antes de `travaB`, e a Thread 2 faz o inverso, existe uma janela onde cada uma segura uma trava e espera pela outra — para sempre

---

# ⚠️ Erros comuns

- Usar `lock (this)` ou `lock` em um objeto público, permitindo que código externo trave o mesmo objeto e cause deadlocks inesperados  
- Segurar uma trava durante uma operação lenta (chamada de rede, I/O), bloqueando outras threads desnecessariamente  
- Esquecer que `lock` só protege código que **também** usa `lock` na mesma trava — não existe proteção automática contra acesso direto sem lock em outro lugar  
- Adquirir múltiplas travas em ordens diferentes em partes diferentes do código, criando risco de deadlock  

---

# 📌 Conclusão

- `lock` é açúcar sintático para `Monitor.Enter`/`Monitor.Exit` com um `try`/`finally` automático  
- Só uma thread por vez executa o código dentro de um `lock` sobre o mesmo objeto  
- `Monitor.TryEnter` com timeout evita esperar indefinidamente por uma trava  
- Deadlocks acontecem quando múltiplas travas são adquiridas em ordens inconsistentes entre threads  

👉 `lock` funciona bem para código síncrono — mas o que fazer quando a seção crítica envolve uma operação `async`? `lock` não permite `await` dentro dele, e é aí que entra a próxima ferramenta

---

# 🔥 Próximo passo

Agora que você sabe proteger seções críticas síncronas, o próximo nível é:

👉 **Fundamentos do C#: SemaphoreSlim e Concorrência Assíncrona**

Aqui você vai aprender a controlar acesso concorrente em código `async`, onde `lock` simplesmente não funciona.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
