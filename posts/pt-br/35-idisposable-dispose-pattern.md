# 🧠 Fundamentos do C#: IDisposable e o Padrão Dispose

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Extension Methods e LINQ Customizado  
- Autenticação e Autorização com JWT  

Você já usou `using var contexto = new AppDbContext(...)` dezenas de vezes desde o post sobre EF Core, sem parar para entender o que exatamente `using` está fazendo. Chegou a hora de abrir essa caixa.

👉 **Vamos aprender IDisposable e o Padrão Dispose**

---

# 💡 O problema: recursos que o Garbage Collector não conhece

Lembra do post sobre processos e memória? O Garbage Collector gerencia memória **gerenciada** — objetos C# comuns. Mas conexões de banco, arquivos abertos e sockets de rede são recursos **não gerenciados**, controlados pelo sistema operacional. O GC não sabe quando liberá-los.

```csharp
// ❌ A conexão pode nunca ser fechada explicitamente
var conexao = new SqlConnection(connectionString);
conexao.Open();
// se uma exceção acontecer aqui, a conexão fica aberta indefinidamente
```

---

# 🏗️ A interface `IDisposable`

```csharp
public interface IDisposable
{
    void Dispose();
}
```

👉 Qualquer classe que gerencia um recurso não gerenciado implementa essa interface — `Dispose()` é o sinal explícito de "terminei, pode liberar isso agora"

```csharp
var conexao = new SqlConnection(connectionString);
try
{
    conexao.Open();
    // usa a conexão
}
finally
{
    conexao.Dispose(); // sempre executa, mesmo com exceção
}
```

---

# 🎯 `using`: o `try/finally` que você já usa

```csharp
using var conexao = new SqlConnection(connectionString);
conexao.Open();
// Dispose() é chamado automaticamente no fim do escopo
```

👉 Essa é exatamente a `using var contexto = new AppDbContext(...)` que você escreve desde o post sobre EF Core — o compilador transforma isso no mesmo `try/finally` que você acabou de ver, só que sem precisar escrever manualmente

```csharp
// Forma com bloco explícito, equivalente
using (var conexao = new SqlConnection(connectionString))
{
    conexao.Open();
} // Dispose() chamado aqui
```

---

# 🔨 Implementando `IDisposable` na sua própria classe

```csharp
public class ArquivoTemporario : IDisposable
{
    private readonly FileStream _stream;
    private bool _descartado;

    public ArquivoTemporario(string caminho)
    {
        _stream = new FileStream(caminho, FileMode.Create);
    }

    public void EscreverDados(byte[] dados) => _stream.Write(dados, 0, dados.Length);

    public void Dispose()
    {
        if (_descartado) return;

        _stream.Dispose();
        _descartado = true;
    }
}
```

```csharp
using var arquivo = new ArquivoTemporario("dados.tmp");
arquivo.EscreverDados(bytes);
// Dispose() libera o FileStream automaticamente
```

👉 A flag `_descartado` evita chamar `Dispose()` duas vezes — um cuidado comum, já que `using` pode coexistir com uma chamada manual em algum outro lugar do código

---

# ⚡ Async: `IAsyncDisposable`

```csharp
public class ConexaoAssincrona : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await LiberarRecursosAsync();
    }
}
```

```csharp
await using var conexao = new ConexaoAssincrona();
```

👉 Lembra do post sobre async/await? Liberar um recurso às vezes exige uma operação assíncrona (fechar uma conexão de rede, por exemplo) — `await using` é o equivalente assíncrono do `using` comum, chamando `DisposeAsync()` em vez de `Dispose()`

---

# ⚠️ Erros comuns

- Esquecer `using` (ou `Dispose()` manual) em recursos como `SqlConnection`, `FileStream` e `HttpClient` fora de um `using`, vazando recursos do sistema operacional aos poucos  
- Implementar `IDisposable` sem realmente ter um recurso não gerenciado para liberar — a interface existe para um propósito específico, não é um padrão genérico de "limpeza"  
- Chamar métodos do objeto depois de `Dispose()`, resultando em `ObjectDisposedException`  
- Não propagar `IDisposable` para cima quando uma classe contém outro objeto descartável como campo — quem cria o campo geralmente é responsável por descartá-lo  

---

# 📌 Conclusão

- `IDisposable` sinaliza que uma classe controla recursos não gerenciados, fora do alcance do Garbage Collector  
- `using` é açúcar sintático para `try/finally` chamando `Dispose()` automaticamente  
- `IAsyncDisposable`/`await using` cobrem o caso onde liberar o recurso exige uma operação assíncrona  
- Implementar a interface só faz sentido quando existe um recurso real para liberar  

👉 Com IDisposable, o `using var` que você digitou dezenas de vezes desde o post sobre EF Core deixa de ser sintaxe memorizada e passa a ser um mecanismo que você entende de ponta a ponta

---

# 🔥 Próximo passo

Agora que você sabe liberar recursos corretamente, o próximo nível é:

👉 **Fundamentos do C#: Streams e Manipulação de Arquivos em C#**

Aqui você vai aprender a trabalhar com arquivos e fluxos de dados de forma eficiente, usando a mesma disciplina de recursos que acabou de aprender.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
