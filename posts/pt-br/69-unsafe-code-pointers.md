# 🧠 Fundamentos do C#: Unsafe Code e Ponteiros

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Performance com Span e Memory (post 49)  
- C# como uma linguagem gerenciada, com Garbage Collector cuidando da memória  

Desde o post sobre hardware e software, você sabe que existe uma camada de memória por baixo de tudo. O C# normalmente esconde isso de você — mas em cenários extremos de performance, você pode acessar memória diretamente, do mesmo jeito que C e C++ fazem.

👉 **Vamos aprender Unsafe Code e Ponteiros**

---

# 💡 Gerenciado vs Unsafe

👉 **Código unsafe = código C# que manipula ponteiros diretamente, saindo da proteção do Garbage Collector**

```csharp
public unsafe void ExemploBasico()
{
    int valor = 42;
    int* ponteiro = &valor;

    Console.WriteLine(*ponteiro); // 42, acessando o valor através do ponteiro
    Console.WriteLine((long)ponteiro); // o endereço de memória em si
}
```

👉 `&` pega o endereço de memória de uma variável, `*` desreferencia (acessa o valor apontado) — os mesmos operadores que você veria em C

---

# 🏗️ Habilitando código unsafe

```xml
<!-- .csproj -->
<PropertyGroup>
  <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
```

```csharp
unsafe
{
    // código com ponteiros só é permitido dentro de um bloco unsafe
}
```

👉 O compilador exige essa flag explícita — o C# não deixa você acessar memória diretamente por acidente, precisa ser uma decisão consciente

---

# 🎯 Fixando memória com `fixed`

```csharp
public unsafe void ProcessarArray(int[] numeros)
{
    fixed (int* ponteiro = numeros)
    {
        for (int i = 0; i < numeros.Length; i++)
        {
            *(ponteiro + i) *= 2; // acessa e modifica direto na memória
        }
    }
}
```

👉 O Garbage Collector pode mover objetos na memória durante uma coleta. `fixed` "trava" o array no lugar temporariamente, garantindo que o ponteiro continue válido enquanto você o usa

---

# ⚡ Onde isso realmente importa: performance extrema

```csharp
public unsafe static void CopiarRapido(byte[] origem, byte[] destino)
{
    fixed (byte* pOrigem = origem)
    fixed (byte* pDestino = destino)
    {
        Buffer.MemoryCopy(pOrigem, pDestino, destino.Length, origem.Length);
    }
}
```

👉 Lembra do post sobre Span/Memory? Ponteiros vão além — eliminam completamente o overhead de bounds-checking do array em loops críticos de performance, algo que só faz sentido em cenários como processamento de imagem, parsers binários ou interoperabilidade com C

---

# 🔗 Interoperabilidade com código nativo

```csharp
[DllImport("kernel32.dll")]
public static unsafe extern bool ReadProcessMemory(
    IntPtr processo, IntPtr endereco, void* buffer, int tamanho, out int lidos);
```

👉 Chamar bibliotecas C/C++ existentes (P/Invoke) frequentemente exige ponteiros, porque é assim que essas linguagens representam memória — o C# precisa "falar a mesma língua" na fronteira entre os dois mundos

---

# ⚠️ Erros comuns

- Usar unsafe code sem necessidade real de performance, perdendo toda a segurança de memória que o C# oferece por padrão  
- Esquecer o `fixed`, deixando o Garbage Collector mover memória enquanto um ponteiro ainda aponta para ela — corrompendo dados silenciosamente  
- Não validar limites manualmente, reintroduzindo os mesmos bugs de buffer overflow que o C# normalmente previne  
- Achar que unsafe torna o código "mais rápido" automaticamente — na maioria dos casos, o JIT já otimiza código gerenciado muito bem  

---

# 📌 Conclusão

- Código unsafe permite manipular ponteiros diretamente, saindo da proteção do Garbage Collector  
- `AllowUnsafeBlocks` e o bloco `unsafe` tornam essa escolha explícita  
- `fixed` impede que o GC mova memória enquanto um ponteiro está em uso  
- O uso real está restrito a cenários extremos: performance crítica e interoperabilidade nativa  

👉 Com unsafe code, você entende que a segurança de memória do C# é uma escolha de design, não uma limitação — e sabe exatamente quando abrir mão dela

---

# 🔥 Próximo passo

Agora que você entende o que existe por baixo da camada gerenciada, o próximo nível é:

👉 **Fundamentos do C#: Reflection e Atributos Customizados**

Aqui você vai aprender a inspecionar e manipular tipos em tempo de execução, a base de frameworks como o próprio ASP.NET Core.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
