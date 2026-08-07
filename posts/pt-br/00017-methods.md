# 🧠 Fundamentos do C#: Métodos, Parâmetros e Retorno de Valores

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Variáveis, tipos e conversões  
- Condicionais e loops  

Seu código já toma decisões e repete tarefas. Mas conforme ele cresce, repetir os mesmos blocos em vários lugares vira um problema.

👉 **É hora de organizar seu código em blocos reutilizáveis: métodos**

---

# 💡 O que é um método?

👉 **Método = um bloco de código nomeado que executa uma tarefa específica**

```csharp
void Cumprimentar()
{
    Console.WriteLine("Olá!");
}

Cumprimentar(); // chamando o método
```

Um método tem quatro partes principais:

- **Tipo de retorno** (`void`, `int`, `string`...)  
- **Nome** (`Cumprimentar`)  
- **Parâmetros** (nesse caso, nenhum)  
- **Corpo** (o código entre `{ }`)  

👉 Sem métodos, todo programa viraria um único bloco gigante e difícil de manter

---

# 📥 Parâmetros: passando dados para dentro

```csharp
void Cumprimentar(string nome)
{
    Console.WriteLine($"Olá, {nome}!");
}

Cumprimentar("Vitor");
```

## 🔹 Parâmetros opcionais

```csharp
void Cumprimentar(string nome = "visitante")
{
    Console.WriteLine($"Olá, {nome}!");
}

Cumprimentar();          // "Olá, visitante!"
Cumprimentar("Vitor");   // "Olá, Vitor!"
```

## 🔹 Argumentos nomeados

```csharp
void CriarUsuario(string nome, int idade, bool ativo) { }

CriarUsuario(nome: "Maria", ativo: true, idade: 25);
```

👉 Argumentos nomeados deixam a chamada mais clara, principalmente com muitos parâmetros

---

# 📤 Retornando valores

```csharp
int Somar(int a, int b)
{
    return a + b;
}

int resultado = Somar(2, 3); // 5
```

👉 O tipo de retorno declarado (`int`) precisa bater com o que o `return` devolve

## 🔹 `void`: quando não há retorno

```csharp
void RegistrarLog(string mensagem)
{
    Console.WriteLine($"[LOG] {mensagem}");
}
```

👉 Métodos `void` executam uma ação, mas não devolvem valor para quem chamou

---

# 🔀 `ref`, `out` e `params`

C# tem formas especiais de lidar com parâmetros:

## 🔹 `ref` — passa por referência

```csharp
void Dobrar(ref int numero)
{
    numero *= 2;
}

int valor = 5;
Dobrar(ref valor); // valor agora é 10
```

👉 A variável original é alterada dentro do método — use com cuidado, pode dificultar a leitura

## 🔹 `out` — retorna valores adicionais

```csharp
bool TentarDividir(int a, int b, out int resultado)
{
    if (b == 0)
    {
        resultado = 0;
        return false;
    }
    resultado = a / b;
    return true;
}

if (TentarDividir(10, 2, out int r))
{
    Console.WriteLine(r); // 5
}
```

👉 Padrão comum em métodos `TryX`, como `int.TryParse`

## 🔹 `params` — número variável de argumentos

```csharp
int Somar(params int[] numeros)
{
    int total = 0;
    foreach (int n in numeros) total += n;
    return total;
}

Somar(1, 2, 3, 4); // 10
```

---

# ✍️ Sobrecarga de métodos (overloading)

Você pode ter métodos com o mesmo nome, desde que os parâmetros sejam diferentes:

```csharp
int Somar(int a, int b) => a + b;
double Somar(double a, double b) => a + b;
```

👉 O compilador escolhe a versão certa com base nos tipos passados na chamada

---

# ⚡ Métodos com corpo de expressão

Para métodos simples, existe uma sintaxe mais enxuta:

```csharp
int Somar(int a, int b) => a + b;
```

👉 Equivalente a escrever `{ return a + b; }`, mas mais direto para lógicas de uma linha

---

# ⚠️ Erros comuns

- Abusar de `ref`/`out` quando um retorno normal já resolveria  
- Criar métodos gigantes que fazem várias coisas ao mesmo tempo  
- Esquecer que parâmetros opcionais precisam vir depois dos obrigatórios  
- Confundir sobrecarga de método com métodos que fazem coisas totalmente diferentes  

---

# 📌 Conclusão

- Métodos organizam código em blocos reutilizáveis e testáveis  
- Parâmetros opcionais e nomeados deixam chamadas mais flexíveis e claras  
- `ref`, `out` e `params` cobrem casos especiais de passagem de dados  
- A sobrecarga permite variações do mesmo método para tipos diferentes  

👉 Com métodos bem definidos, seu código fica mais organizado, legível e fácil de testar

---

# 🔥 Próximo passo

Agora que você sabe organizar lógica em métodos, o próximo nível é:

👉 **Fundamentos do C#: Coleções (Arrays, Listas e Dicionários)**

Aqui você vai aprender a armazenar e manipular grupos de dados.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
