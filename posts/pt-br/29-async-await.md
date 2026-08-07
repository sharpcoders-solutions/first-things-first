# 🧠 Fundamentos do C#: Async/Await na Prática

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Threads e concorrência, em teoria  
- Delegates, eventos e expressões lambda  

Lá no começo da trilha, você entendeu **por que** a programação assíncrona existe. Agora é hora de aplicar isso com a sintaxe real do C#.

👉 **Vamos colocar `async` e `await` para trabalhar de verdade**

---

# 💡 O problema que `async`/`await` resolve

Imagine buscar dados de uma API:

```csharp
string dados = BuscarDadosDaApi(); // isso pode levar 2 segundos
Console.WriteLine("Processando...");
```

👉 Enquanto a chamada espera a resposta, a **thread fica bloqueada**, sem fazer mais nada — em uma aplicação web, isso significa menos usuários atendidos ao mesmo tempo

Com `async`/`await`, a thread é **liberada** enquanto espera, e volta a executar o método quando a resposta chega.

---

# 🏗️ Sintaxe básica

```csharp
async Task<string> BuscarDadosAsync()
{
    await Task.Delay(2000); // simula uma operação demorada (ex: chamada de API)
    return "Dados recebidos";
}
```

```csharp
string resultado = await BuscarDadosAsync();
Console.WriteLine(resultado);
```

## 🔹 As três peças da sintaxe

- `async` → marca o método como assíncrono  
- `Task` / `Task<T>` → representa uma operação que ainda não terminou  
- `await` → pausa a execução **daquele método** até a tarefa terminar, sem bloquear a thread  

👉 `Task` é como um `void` assíncrono; `Task<T>` é como um método que retorna `T`, só que de forma assíncrona

---

# ⚠️ `async void`: a exceção perigosa

```csharp
async void Salvar() // ❌ evite isso
{
    await SalvarNoBancoAsync();
}
```

👉 Métodos `async void` não podem ser aguardados (`await`) por quem os chama, e exceções lançados neles podem derrubar a aplicação sem aviso

**Regra prática:** use `async Task` (ou `async Task<T>`) sempre. A única exceção aceitável é em manipuladores de evento (`async void OnClick(...)`), que já seguem essa assinatura por convenção do .NET

---

# 🧵 `async Main`: o ponto de entrada assíncrono

```csharp
async Task Main()
{
    string dados = await BuscarDadosAsync();
    Console.WriteLine(dados);
}
```

👉 Desde versões modernas do C#, o próprio `Main` pode ser assíncrono, permitindo usar `await` diretamente no ponto de entrada do programa

---

# 🔀 Rodando tarefas em paralelo: `Task.WhenAll`

Se as tarefas não dependem uma da outra, esperar uma de cada vez desperdiça tempo:

```csharp
// ❌ Sequencial: soma o tempo de cada chamada
string dados1 = await BuscarDadosAsync();
string dados2 = await BuscarDadosAsync();

// ✅ Paralelo: espera todas ao mesmo tempo
Task<string> tarefa1 = BuscarDadosAsync();
Task<string> tarefa2 = BuscarDadosAsync();

string[] resultados = await Task.WhenAll(tarefa1, tarefa2);
```

👉 `Task.WhenAll` dispara todas as tarefas de uma vez e só continua quando **todas** terminarem — muito mais rápido quando as chamadas são independentes

---

# 🧨 A armadilha do `.Result` e `.Wait()`

```csharp
string dados = BuscarDadosAsync().Result; // ❌ pode causar deadlock
```

👉 Chamar `.Result` ou `.Wait()` em código síncrono para "forçar" uma tarefa assíncrona a terminar pode travar a aplicação inteira, especialmente em apps com UI ou ASP.NET clássico

**Regra prática:** use `await` do início ao fim. Se um método chama código assíncrono, ele também deveria ser assíncrono — essa propagação é chamada de "async all the way"

---

# 🧰 Tratamento de exceções em código assíncrono

A boa notícia: `try/catch` funciona normalmente com `await`.

```csharp
async Task ProcessarAsync()
{
    try
    {
        await BuscarDadosAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Erro: {ex.Message}");
    }
}
```

👉 Não é preciso nenhuma sintaxe especial — a exceção lançada dentro do `Task` é capturada normalmente pelo `catch` de quem deu `await`

---

# ⚠️ Erros comuns

- Usar `async void` fora de manipuladores de evento  
- Misturar `.Result`/`.Wait()` com código assíncrono, arriscando deadlock  
- Esperar tarefas independentes em sequência em vez de usar `Task.WhenAll`  
- Esquecer o `await`, fazendo o método continuar sem esperar o resultado ("fire and forget" acidental)  
- Achar que `async` sozinho já torna o código mais rápido — ele torna o código **não bloqueante**, que é diferente de mais rápido  

---

# 📌 Conclusão

- `async`/`await` libera a thread enquanto uma operação demorada não termina  
- `Task` e `Task<T>` representam operações assíncronas, com ou sem retorno  
- Evite `async void`, exceto em manipuladores de evento  
- `Task.WhenAll` executa tarefas independentes em paralelo  
- `.Result`/`.Wait()` misturados com `await` são a receita mais comum para deadlocks  

👉 Com `async`/`await` na prática, você já sabe escrever código C# que escala de verdade em cenários de I/O

---

# 🔥 Próximo passo

Agora que você aplicou programação assíncrona na prática, o próximo nível é:

👉 **Fundamentos do C#: Records e Pattern Matching (C# Moderno)**

Aqui você vai conhecer os recursos mais recentes da linguagem que tornam o código ainda mais conciso e expressivo.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
