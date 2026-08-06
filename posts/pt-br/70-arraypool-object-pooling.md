# 🧠 Fundamentos do C#: ArrayPool e Object Pooling

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Native AOT para eliminar o JIT em runtime  
- Span e Memory para acessar memória sem alocações extras (post 46)  

Alocar um array novo toda vez que você precisa de um buffer temporário parece inofensivo, mas em loops de alta frequência isso pressiona o Garbage Collector constantemente. ArrayPool e Object Pooling resolvem isso reutilizando memória.

👉 **Vamos aprender ArrayPool e Object Pooling**

---

# 💡 O problema: alocação excessiva

```csharp
// ❌ Aloca um array novo a cada chamada
public byte[] ProcessarDados(int tamanho)
{
    var buffer = new byte[tamanho]; // nova alocação, toda vez
    // ... processa o buffer
    return buffer;
} // o array vira lixo assim que sai de escopo
```

👉 Chamado mil vezes por segundo, isso gera pressão constante sobre o Garbage Collector — lembra do post sobre processos e memória? Cada coleta pausa a aplicação, mesmo que brevemente

---

# 🏗️ ArrayPool: reutilizando arrays

```csharp
public void ProcessarDados(int tamanho)
{
    byte[] buffer = ArrayPool<byte>.Shared.Rent(tamanho);

    try
    {
        // ... processa o buffer normalmente
    }
    finally
    {
        ArrayPool<byte>.Shared.Return(buffer);
    }
}
```

👉 `Rent` pega um array emprestado do pool (reutilizando um já alocado, se disponível), e `Return` devolve para reuso futuro — nenhuma alocação nova na maioria das chamadas

---

# ⚠️ Cuidado: o array emprestado pode ser maior

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(100);

Console.WriteLine(buffer.Length); // pode ser 128, não exatamente 100!
```

👉 `Rent` garante um array com **pelo menos** o tamanho pedido, mas pode devolver um maior (potências de 2, por exemplo) — sempre use o tamanho que você pediu, não `buffer.Length`, ao processar os dados

---

# 🎯 Object Pooling: reutilizando objetos completos

```csharp
public class PedidoPoolPolicy : PooledObjectPolicy<ProcessadorPedido>
{
    public override ProcessadorPedido Create() => new ProcessadorPedido();

    public override bool Return(ProcessadorPedido objeto)
    {
        objeto.Resetar(); // limpa o estado antes de devolver ao pool
        return true;
    }
}
```

```csharp
var pool = new DefaultObjectPool<ProcessadorPedido>(new PedidoPoolPolicy());

var processador = pool.Get();
try
{
    processador.Processar(pedido);
}
finally
{
    pool.Return(processador);
}
```

👉 Diferente do ArrayPool (que é só para arrays), `ObjectPool<T>` (do pacote `Microsoft.Extensions.ObjectPool`) reutiliza qualquer tipo de objeto caro de criar — a chave é sempre resetar o estado no `Return`, para o próximo consumidor não herdar dados antigos

---

# 🔍 Onde o ASP.NET Core já usa isso

👉 O próprio ASP.NET Core usa Object Pooling internamente para `StringBuilder` (lembra do post 46?) e buffers de resposta HTTP — cada requisição não aloca esses objetos do zero, eles são emprestados e devolvidos ao pool automaticamente

---

# ⚠️ Erros comuns

- Esquecer o `Return` (ou não usar `try/finally`), fazendo o objeto nunca voltar ao pool e negando todo o benefício  
- Não resetar o estado do objeto antes de devolvê-lo, vazando dados de um consumidor para o próximo  
- Usar pooling para objetos baratos de criar, adicionando complexidade sem ganho real de performance  
- Assumir que o array emprestado tem exatamente o tamanho pedido, causando bugs sutis  

---

# 📌 Conclusão

- Alocações repetidas em loops de alta frequência pressionam o Garbage Collector  
- `ArrayPool<T>.Shared` reutiliza arrays via `Rent`/`Return`  
- `ObjectPool<T>` generaliza esse padrão para qualquer objeto caro de criar  
- O ASP.NET Core já usa esse mecanismo internamente para buffers e `StringBuilder`  

👉 Com ArrayPool e Object Pooling, você reduz a pressão sobre o GC exatamente nos pontos onde alocação repetida realmente custa caro

---

# 🔥 Próximo passo

Agora que você sabe reduzir alocações reutilizando memória, o próximo nível é:

👉 **Fundamentos do C#: Channels**

Aqui você vai aprender a coordenar produtores e consumidores dentro do mesmo processo, sem precisar de uma fila externa como o RabbitMQ.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
