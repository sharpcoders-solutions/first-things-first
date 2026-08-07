# 🧠 Fundamentos do C#: P/Invoke e Interoperabilidade Nativa

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `System.Text.Json` avançado e o source generator de serialização  
- Unsafe code e ponteiros (post 66), incluindo uma introdução rápida a P/Invoke  

O post sobre unsafe code te mostrou um exemplo rápido de `[DllImport]`. Chegou a hora de entender P/Invoke a fundo — a ponte que permite C# chamar bibliotecas nativas escritas em C, C++ ou qualquer linguagem que exponha uma API compatível com C.

👉 **Vamos aprender P/Invoke em profundidade**

---

# 💡 O que é P/Invoke?

👉 **P/Invoke (Platform Invoke) = o mecanismo do .NET para chamar funções de bibliotecas nativas (`.dll`, `.so`, `.dylib`) a partir de código gerenciado**

```csharp
[DllImport("user32.dll")]
public static extern int MessageBox(IntPtr hWnd, string texto, string titulo, uint tipo);

MessageBox(IntPtr.Zero, "Olá do C#!", "P/Invoke", 0);
```

👉 `[DllImport]` diz ao runtime onde encontrar a função nativa. O método `extern` não tem corpo em C# — a implementação vive inteiramente na biblioteca nativa, e o CLR faz a ponte entre os dois mundos em tempo de execução

---

# 🔄 Marshaling: traduzindo tipos entre mundos

```csharp
[DllImport("meubibnativa.dll", CharSet = CharSet.Unicode)]
public static extern int ProcessarTexto(string entrada, StringBuilder saida, int tamanhoBuffer);

var saida = new StringBuilder(256);
ProcessarTexto("dados de entrada", saida, saida.Capacity);
Console.WriteLine(saida.ToString());
```

👉 **Marshaling = o processo de converter tipos entre a representação gerenciada do .NET e a representação nativa usada pela biblioteca C/C++**

`string` em C# não é o mesmo que `char*` em C — o marshaling cuida dessa conversão automaticamente na maioria dos casos, mas `CharSet.Unicode` (ou `Ansi`) controla exatamente como essa tradução acontece, algo que precisa bater exatamente com o que a biblioteca nativa espera

---

# 📦 Structs e marshaling de dados complexos

```csharp
[StructLayout(LayoutKind.Sequential)]
public struct PontoNativo
{
    public int X;
    public int Y;
}

[DllImport("minhabib.dll")]
public static extern void ProcessarPonto(ref PontoNativo ponto);

var ponto = new PontoNativo { X = 10, Y = 20 };
ProcessarPonto(ref ponto);
```

👉 `[StructLayout(LayoutKind.Sequential)]` garante que os campos do struct fiquem na memória exatamente na mesma ordem e layout que a biblioteca nativa espera — lembra do post sobre value types (post 43)? Esse é um caso onde o layout exato na memória, geralmente uma preocupação interna do runtime, se torna sua responsabilidade explícita

---

# 🎯 LibraryImport: a evolução moderna de `DllImport`

```csharp
public static partial class Nativa
{
    [LibraryImport("minhabib.dll")]
    public static partial int Somar(int a, int b);
}
```

👉 **`[LibraryImport]` = a versão moderna de `[DllImport]`, baseada em source generator (lembra do `GeneratedRegex` e do `JsonSerializerContext`?)**

Assim como as outras funcionalidades baseadas em source generator que você já viu, `[LibraryImport]` gera código de marshaling em tempo de **compilação**, em vez de usar reflection em runtime — mais rápido, e compatível com Native AOT (post 69), onde `[DllImport]` tradicional pode ter limitações

---

# 🔗 Callbacks: C chamando C#

```csharp
public delegate void CallbackProgresso(int percentual);

[DllImport("minhabib.dll")]
public static extern void ProcessarComProgresso(CallbackProgresso callback);

ProcessarComProgresso(percentual =>
{
    Console.WriteLine($"Progresso: {percentual}%");
});
```

👉 Lembra dos delegates (post sobre eventos)? Um delegate C# pode ser passado como ponteiro de função para código nativo — a biblioteca C chama de volta para dentro do C# durante o processamento, útil para relatar progresso de operações longas

---

# ⚠️ Cuidados de segurança de memória

```csharp
[DllImport("minhabib.dll")]
public static extern IntPtr AlocarBuffer(int tamanho);

[DllImport("minhabib.dll")]
public static extern void LiberarBuffer(IntPtr buffer);

IntPtr buffer = AlocarBuffer(1024);
try
{
    // usar o buffer
}
finally
{
    LiberarBuffer(buffer); // OBRIGATÓRIO — o GC não gerencia memória nativa
}
```

👉 Lembra do post sobre `IDisposable`? Memória alocada do lado nativo **nunca** é rastreada pelo Garbage Collector do .NET — esquecer de liberá-la explicitamente é um vazamento de memória real, do mesmo tipo que existia em C antes de qualquer runtime gerenciado

---

# ⚠️ Erros comuns

- Esquecer de liberar memória alocada nativamente, causando vazamentos que o GC nunca vai detectar ou coletar  
- Configurar `CharSet` incorretamente, causando corrupção de strings ao cruzar a fronteira gerenciado/nativo  
- Usar `[DllImport]` tradicional em projetos Native AOT sem testar, quando `[LibraryImport]` resolveria com mais compatibilidade  
- Ignorar diferenças de convenção de chamada (calling convention) entre plataformas, causando crashes difíceis de depurar  

---

# 📌 Conclusão

- P/Invoke permite chamar bibliotecas nativas C/C++ diretamente do C#  
- Marshaling converte tipos entre a representação gerenciada e a nativa automaticamente, com controle fino via atributos  
- `[LibraryImport]` é a evolução moderna, baseada em source generator, mais rápida e compatível com Native AOT  
- Memória alocada nativamente nunca é gerenciada pelo GC — liberar explicitamente é responsabilidade sua  

👉 Falando em GC: você já viu ele mencionado dezenas de vezes nesta trilha, mas nunca em profundidade. É hora de entender exatamente como ele decide quando coletar um objeto

---

# 🔥 Próximo passo

Agora que você sabe interagir com código nativo, o próximo nível é:

👉 **Fundamentos do C#: WeakReference, Finalizers e o Garbage Collector**

Aqui você vai aprender como o GC realmente decide quando coletar um objeto, e como referências fracas evitam vazamentos de memória em cenários específicos.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
