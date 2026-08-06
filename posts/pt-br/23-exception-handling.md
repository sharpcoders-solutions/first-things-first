# 🧠 Fundamentos do C#: Tratamento de Exceções (try, catch, finally e Exceções Customizadas)

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Classes, herança, polimorfismo e interfaces  
- Como modelar contratos e comportamento entre objetos  

Mas nenhum programa real roda sem erros. Arquivos não existem, APIs caem, usuários digitam valores inválidos.

👉 **A diferença entre um app amador e um profissional está em como ele lida com isso**

---

# 💡 O que é uma exceção?

👉 **Exceção = um evento inesperado que interrompe o fluxo normal do programa**

```csharp
int[] numeros = { 1, 2, 3 };
Console.WriteLine(numeros[5]); // IndexOutOfRangeException
```

Sem tratamento, uma exceção **derruba o programa**. Com tratamento, você decide o que fazer quando algo dá errado.

---

# 🏗️ `try`, `catch` e `finally`

```csharp
try
{
    int resultado = 10 / int.Parse("0");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Erro: {ex.Message}");
}
finally
{
    Console.WriteLine("Isso sempre executa, com ou sem erro");
}
```

## 🔹 Como funciona

- `try` → onde você coloca o código que pode falhar  
- `catch` → captura a exceção e decide o que fazer  
- `finally` → executa **sempre**, tenha havido erro ou não (ótimo para liberar recursos)  

👉 O `finally` roda mesmo se houver um `return` dentro do `try` ou do `catch`

---

# 🎯 Capturando tipos específicos

Você pode ter múltiplos blocos `catch`, do mais específico para o mais genérico:

```csharp
try
{
    string texto = null;
    Console.WriteLine(texto.Length);
}
catch (NullReferenceException ex)
{
    Console.WriteLine("Referência nula: " + ex.Message);
}
catch (Exception ex)
{
    Console.WriteLine("Erro inesperado: " + ex.Message);
}
```

👉 O C# testa os blocos `catch` **na ordem em que aparecem** — por isso, o mais específico sempre deve vir antes do genérico

## 🔹 Exceções mais comuns no dia a dia

- `NullReferenceException` → acessar um membro de algo que é `null`  
- `IndexOutOfRangeException` → acessar um índice que não existe  
- `InvalidOperationException` → operação inválida para o estado atual do objeto  
- `ArgumentException` / `ArgumentNullException` → argumento inválido passado para um método  
- `DivideByZeroException` → divisão por zero em tipos inteiros  

---

# ✋ Lançando suas próprias exceções

```csharp
void Sacar(decimal valor, decimal saldo)
{
    if (valor > saldo)
    {
        throw new InvalidOperationException("Saldo insuficiente");
    }
}
```

👉 `throw` interrompe a execução imediatamente e propaga a exceção para quem chamou o método

---

# 🧱 Exceções customizadas

Quando as exceções prontas do .NET não descrevem bem o problema do seu domínio, você pode criar as suas:

```csharp
class SaldoInsuficienteException : Exception
{
    public decimal Saldo { get; }
    public decimal ValorSolicitado { get; }

    public SaldoInsuficienteException(decimal saldo, decimal valorSolicitado)
        : base($"Saldo insuficiente: disponível R$ {saldo}, solicitado R$ {valorSolicitado}")
    {
        Saldo = saldo;
        ValorSolicitado = valorSolicitado;
    }
}
```

```csharp
void Sacar(decimal valor, decimal saldo)
{
    if (valor > saldo)
        throw new SaldoInsuficienteException(saldo, valor);
}

try
{
    Sacar(500, 100);
}
catch (SaldoInsuficienteException ex)
{
    Console.WriteLine(ex.Message);
    Console.WriteLine($"Faltam R$ {ex.ValorSolicitado - ex.Saldo}");
}
```

👉 Exceções customizadas herdam de `Exception` e podem carregar dados extras relevantes para quem for tratar o erro

---

# 🔁 `throw` vs `throw ex`: um detalhe que muda tudo

```csharp
catch (Exception ex)
{
    throw;      // ✅ preserva o stack trace original
    // throw ex; // ❌ reseta o stack trace, dificultando o diagnóstico
}
```

👉 Use `throw;` sozinho para relançar a exceção sem perder a informação de onde ela realmente aconteceu

---

# 🧹 `using`: uma forma mais segura que `finally` para recursos

Para recursos que implementam `IDisposable` (arquivos, conexões, streams), existe uma forma mais limpa de garantir a liberação:

```csharp
using (var arquivo = new StreamReader("dados.txt"))
{
    string conteudo = arquivo.ReadToEnd();
} // arquivo é liberado automaticamente aqui, mesmo se der erro
```

👉 O `using` chama `Dispose()` automaticamente — equivalente a um `try/finally` implícito

---

# ⚠️ Boas práticas (e erros comuns)

- **Não engula exceções silenciosamente**: um `catch` vazio esconde problemas em vez de resolvê-los  
- **Não use exceções para controle de fluxo normal**: são custosas e devem representar situações realmente excepcionais  
- **Evite capturar `Exception` genérico** quando você sabe qual erro específico pode ocorrer  
- **Sempre prefira `throw;`** a `throw ex;` ao relançar  
- **Libere recursos com `using`**, não confiando só em lembrar de chamar `Dispose()`  

```csharp
// ❌ Nunca faça isso
try
{
    ProcessarPedido();
}
catch (Exception)
{
    // nada aqui — erro desaparece silenciosamente
}
```

---

# 📌 Conclusão

- `try/catch/finally` estrutura como seu programa reage a erros  
- Capture exceções específicas antes de exceções genéricas  
- `throw` cria e propaga uma exceção; `throw;` relança preservando o stack trace  
- Exceções customizadas descrevem melhor os erros do seu domínio  
- `using` garante liberação de recursos de forma mais segura que `finally` manual  

👉 Tratar exceções bem é o que separa um código que só "funciona no feliz caminho" de um código pronto para produção

---

# 🔥 Próximo passo

Agora que seu código sabe lidar com o inesperado, o próximo nível é:

👉 **Fundamentos do C#: Generics**

Aqui você vai aprender a escrever código reutilizável e type-safe para qualquer tipo de dado.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
