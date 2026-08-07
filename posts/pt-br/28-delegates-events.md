# 🧠 Fundamentos do C#: Delegates, Eventos e Expressões Lambda

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Generics: código reutilizável e type-safe  
- Interfaces e contratos entre classes  

Até agora, métodos sempre foram chamados diretamente. Mas e se você precisasse **passar um método como parâmetro**, ou avisar outras partes do sistema quando algo acontece?

👉 **É para isso que existem delegates, eventos e lambdas**

---

# 💡 O que é um delegate?

👉 **Delegate = um tipo que representa a referência a um método**

```csharp
delegate int Operacao(int a, int b);

int Somar(int a, int b) => a + b;
int Multiplicar(int a, int b) => a * b;

Operacao op = Somar;
Console.WriteLine(op(2, 3)); // 5

op = Multiplicar;
Console.WriteLine(op(2, 3)); // 6
```

👉 Assim como uma variável `int` guarda um número, uma variável `Operacao` guarda **um método** — e você pode trocar qual método ela aponta em tempo de execução

---

# 📦 Delegates prontos: `Action`, `Func` e `Predicate`

Você raramente precisa criar seus próprios delegates — o .NET já fornece os mais comuns:

## 🔹 `Action<T>` — método sem retorno

```csharp
Action<string> imprimir = mensagem => Console.WriteLine(mensagem);
imprimir("Olá!");
```

## 🔹 `Func<T, TResult>` — método com retorno

```csharp
Func<int, int, int> somar = (a, b) => a + b;
int resultado = somar(2, 3); // 5
```

## 🔹 `Predicate<T>` — método que retorna `bool`

```csharp
Predicate<int> ehPar = numero => numero % 2 == 0;
bool resultado2 = ehPar(4); // true
```

👉 `Func` e `Predicate` já apareceram disfarçados no post sobre LINQ — `Where(n => n % 2 == 0)` recebe exatamente um `Func<T, bool>`

---

# ⚡ Expressões lambda: a sintaxe por trás do `=>`

```csharp
Func<int, int> dobro = numero => numero * 2;

// equivalente a:
int Dobro(int numero)
{
    return numero * 2;
}
```

👉 Lambda é só uma forma enxuta de escrever um método sem precisar nomeá-lo — muito usada quando o método só existe para ser passado como argumento

## 🔹 Métodos como parâmetros (callbacks)

```csharp
void ProcessarLista(List<int> numeros, Action<int> acao)
{
    foreach (int numero in numeros)
    {
        acao(numero);
    }
}

ProcessarLista(new List<int> { 1, 2, 3 }, n => Console.WriteLine(n * 10));
```

👉 `ProcessarLista` não sabe (nem precisa saber) o que a `acao` faz — ela só executa o que foi passado

---

# 🔔 Eventos: comunicação entre objetos

👉 **Evento = um delegate especial que avisa outras partes do sistema quando algo acontece**

```csharp
class Pedido
{
    public event Action<string> StatusAlterado;

    private string status;

    public string Status
    {
        get => status;
        set
        {
            status = value;
            StatusAlterado?.Invoke(status); // avisa quem estiver "escutando"
        }
    }
}
```

```csharp
Pedido pedido = new Pedido();
pedido.StatusAlterado += mensagem => Console.WriteLine($"Novo status: {mensagem}");
pedido.StatusAlterado += mensagem => Console.WriteLine($"Enviando notificação: {mensagem}");

pedido.Status = "Enviado";
// Novo status: Enviado
// Enviando notificação: Enviado
```

👉 Com `+=`, você **inscreve** um método para ser chamado quando o evento disparar. Com `-=`, você se **desinscreve**

## 🔹 Por que não usar um `Action` público direto?

```csharp
public Action<string> StatusAlterado; // ❌ qualquer código externo pode chamar isso diretamente

pedido.StatusAlterado("Hackeado!"); // isso não deveria ser permitido de fora
```

👉 A palavra-chave `event` impede que código externo **dispare** o evento diretamente — só a própria classe pode invocá-lo. Isso é encapsulamento aplicado à comunicação entre objetos

---

# ✅ `?.Invoke`: evitando `NullReferenceException`

```csharp
StatusAlterado?.Invoke(status);
```

👉 Se ninguém se inscreveu no evento, ele é `null` — o `?.` evita lançar uma exceção ao tentar chamar um evento sem nenhum ouvinte

---

# ⚠️ Erros comuns

- Esquecer o `?.` antes de `Invoke`, causando `NullReferenceException` quando não há inscritos  
- Deixar de se desinscrever (`-=`) de eventos, causando vazamento de memória em objetos de longa duração  
- Usar `Action` público em vez de `event`, perdendo o encapsulamento  
- Escrever lambdas grandes e complexas, quando um método nomeado seria mais legível  

---

# 📌 Conclusão

- Delegates permitem tratar métodos como valores, guardados em variáveis  
- `Action`, `Func` e `Predicate` cobrem a grande maioria dos casos do dia a dia  
- Lambdas são a forma enxuta de escrever métodos anônimos  
- `event` adiciona uma camada de encapsulamento sobre delegates, essencial para notificar mudanças entre objetos  

👉 Com delegates, lambdas e eventos, seu código ganha um novo nível de flexibilidade — a base de callbacks, LINQ e programação orientada a eventos em C#

---

# 🔥 Próximo passo

Agora que você sabe tratar métodos como valores, o próximo nível é:

👉 **Fundamentos do C#: Async/Await na Prática**

Aqui você vai aplicar, com sintaxe real de C#, os conceitos de programação assíncrona que você já viu na teoria.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
