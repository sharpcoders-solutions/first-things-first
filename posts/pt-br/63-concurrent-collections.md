# 🧠 Fundamentos do C#: Coleções Concorrentes (ConcurrentDictionary e Cia.)

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Testes de integração com banco in-memory  
- Static abstract interface members e o resto dos recursos avançados de tipos  

Você já usou `Dictionary<TKey, TValue>` e `List<T>` centenas de vezes. Mas o que acontece quando duas threads tentam escrever nelas ao mesmo tempo? A resposta não é bonita — e é exatamente o problema que as coleções concorrentes resolvem.

👉 **Vamos aprender sobre `ConcurrentDictionary` e as outras coleções thread-safe do .NET**

---

# 💡 O problema: coleções comuns não são thread-safe

```csharp
var contador = new Dictionary<string, int>();

// Duas threads incrementando o mesmo contador ao mesmo tempo
Parallel.For(0, 1000, i =>
{
    if (contador.ContainsKey("total"))
        contador["total"]++;
    else
        contador["total"] = 1;
});

Console.WriteLine(contador["total"]); // ❌ resultado imprevisível, e pode até lançar exceção
```

👉 `Dictionary<TKey, TValue>` **não** foi projetado para múltiplas threads escrevendo simultaneamente. O resultado pode ser um número menor que 1000 (incrementos perdidos), ou até uma `InvalidOperationException` ("Collection was modified") se duas threads mexerem na estrutura interna ao mesmo tempo

---

# 🔐 A solução ingênua: lock em tudo

```csharp
var contador = new Dictionary<string, int>();
var trava = new object();

Parallel.For(0, 1000, i =>
{
    lock (trava)
    {
        if (contador.ContainsKey("total"))
            contador["total"]++;
        else
            contador["total"] = 1;
    }
});
```

👉 Isso funciona, mas serializa **todo** acesso à coleção — mesmo threads que só querem ler ficam esperando na fila. Em cenários de alta concorrência, isso vira um gargalo

---

# 🎯 `ConcurrentDictionary<TKey, TValue>`: thread-safe por padrão

```csharp
var contador = new ConcurrentDictionary<string, int>();

Parallel.For(0, 1000, i =>
{
    contador.AddOrUpdate("total", 1, (chave, valorAtual) => valorAtual + 1);
});

Console.WriteLine(contador["total"]); // ✅ sempre 1000
```

👉 `AddOrUpdate` é **atômico**: ou adiciona o valor inicial, ou aplica a função de atualização — sem espaço para duas threads pisarem uma na outra no meio do caminho. Por dentro, `ConcurrentDictionary` usa uma combinação de locks finos (por partição de dados) em vez de um lock único e global, o que permite muito mais paralelismo real que a solução com `lock`

---

# 🔧 Os métodos atômicos mais usados

```csharp
var cache = new ConcurrentDictionary<int, string>();

// GetOrAdd: busca, ou cria e adiciona se não existir — tudo atomicamente
var valor = cache.GetOrAdd(1, id => BuscarDoBancoDeDados(id));

// TryAdd: adiciona só se a chave não existir
bool adicionou = cache.TryAdd(2, "novo valor");

// TryUpdate: atualiza só se o valor atual bater com o esperado
bool atualizou = cache.TryUpdate(1, "valor novo", "valor antigo");

// TryRemove: remove e retorna o valor removido
bool removeu = cache.TryRemove(1, out var valorRemovido);
```

👉 Cada um desses métodos resolve um problema clássico de concorrência: "verificar e depois agir" (`ContainsKey` seguido de `this[chave] = valor`) tem uma janela onde outra thread pode intervir entre as duas operações. Os métodos atômicos eliminam essa janela

---

# 📦 Outras coleções concorrentes do namespace `System.Collections.Concurrent`

```csharp
// ConcurrentBag<T>: coleção não ordenada, otimizada para múltiplos produtores/consumidores
var bag = new ConcurrentBag<int>();
Parallel.For(0, 100, i => bag.Add(i));

// ConcurrentQueue<T>: fila FIFO thread-safe
var fila = new ConcurrentQueue<string>();
fila.Enqueue("tarefa 1");
fila.TryDequeue(out var proximaTarefa);

// ConcurrentStack<T>: pilha LIFO thread-safe
var pilha = new ConcurrentStack<int>();
pilha.Push(1);
pilha.TryPop(out var topo);
```

👉 Cada coleção resolve um padrão de acesso diferente — `ConcurrentQueue<T>` é a estrutura natural para uma fila de trabalho processada por múltiplos workers, algo que você usaria em conjunto com o `BackgroundService` do post sobre Hangfire

---

# ⚖️ Quando ainda vale a pena usar `lock`

```csharp
lock (trava)
{
    var saldoOrigem = contas[origemId] - valor;
    var saldoDestino = contas[destinoId] + valor;

    if (saldoOrigem < 0)
        throw new InvalidOperationException("Saldo insuficiente");

    contas[origemId] = saldoOrigem;
    contas[destinoId] = saldoDestino;
}
```

👉 **Coleções concorrentes garantem que cada operação individual é atômica — não que uma sequência de várias operações relacionadas seja atômica como um todo.** Quando você precisa que múltiplas operações aconteçam como uma unidade indivisível (como transferir saldo entre duas contas), ainda precisa de um `lock` explícito envolvendo toda a sequência

---

# ⚠️ Erros comuns

- Usar `Dictionary<TKey, TValue>` comum em código acessado por múltiplas threads, achando que "geralmente funciona" é o mesmo que "é seguro"  
- Combinar `ContainsKey` seguido de indexação (`dict[chave]`) mesmo em coleções concorrentes — use `TryGetValue` para evitar a janela entre as duas chamadas  
- Achar que coleções concorrentes tornam qualquer sequência de operações atômica, quando só a operação individual é garantida  
- Usar `lock` em toda a coleção quando `ConcurrentDictionary` resolveria o mesmo problema com melhor performance  

---

# 📌 Conclusão

- `Dictionary<TKey, TValue>` e outras coleções comuns não são seguras para múltiplas threads escrevendo simultaneamente  
- `ConcurrentDictionary` usa locks finos internos, permitindo mais paralelismo real que um `lock` manual sobre tudo  
- Métodos como `GetOrAdd`, `TryUpdate` e `TryRemove` são atômicos, eliminando janelas de corrida entre "verificar" e "agir"  
- `ConcurrentBag`, `ConcurrentQueue` e `ConcurrentStack` cobrem outros padrões de acesso concorrente  
- Sequências de múltiplas operações relacionadas ainda podem exigir `lock` explícito, mesmo com coleções concorrentes  

👉 Coleções concorrentes resolvem o problema de estruturas de dados compartilhadas entre threads — mas testar esse tipo de código exige uma disciplina própria, que é onde entra a próxima etapa da sua jornada em qualidade de software

---

# 🔥 Próximo passo

Agora que você sabe lidar com dados compartilhados entre threads, o próximo nível é:

👉 **Fundamentos do C#: Mutation Testing**

Aqui você vai aprender a testar se seus próprios testes realmente pegam bugs, não só se eles passam.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
