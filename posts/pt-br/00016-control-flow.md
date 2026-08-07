# 🧠 Fundamentos do C#: Estruturas de Controle (if, else, switch e loops)

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como criar e rodar seu primeiro programa  
- Variáveis, tipos e como converter dados  

Mas até agora, seu código só faz uma coisa: executa linha por linha, sem decidir nada.

👉 **Chegou a hora de controlar o fluxo do programa**

---

# 💡 Condicionais: `if`, `else if` e `else`

A estrutura mais básica de decisão em C#:

```csharp
int idade = 20;

if (idade >= 18)
{
    Console.WriteLine("Maior de idade");
}
else if (idade >= 12)
{
    Console.WriteLine("Adolescente");
}
else
{
    Console.WriteLine("Criança");
}
```

👉 O bloco executado é sempre o **primeiro** cuja condição for verdadeira

## 🔹 Operador ternário

Para decisões simples, existe uma versão enxuta:

```csharp
string status = idade >= 18 ? "Maior de idade" : "Menor de idade";
```

👉 Útil para atribuições curtas — evite usar em decisões complexas, isso prejudica a leitura

---

# 🔀 `switch`: quando há várias opções

Quando você tem muitas condições sobre o mesmo valor, o `switch` fica mais legível que uma cadeia de `if/else if`:

```csharp
switch (diaSemana)
{
    case 1:
        Console.WriteLine("Segunda-feira");
        break;
    case 2:
        Console.WriteLine("Terça-feira");
        break;
    default:
        Console.WriteLine("Outro dia");
        break;
}
```

👉 Diferente de outras linguagens, o C# **não** permite fallthrough silencioso — cada `case` precisa de `break` (ou `return`)

## 🔹 Switch expression (C# moderno)

Uma forma mais concisa, ideal quando o objetivo é apenas retornar um valor:

```csharp
string nomeDia = diaSemana switch
{
    1 => "Segunda-feira",
    2 => "Terça-feira",
    _ => "Outro dia"
};
```

👉 O `_` funciona como o `default` — captura qualquer valor não tratado

---

# 🔁 Loops: repetindo código

## 🔹 `for`

Ideal quando você sabe **quantas vezes** repetir:

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);
}
```

## 🔹 `while`

Repete **enquanto** a condição for verdadeira — útil quando você não sabe o número exato de repetições:

```csharp
int tentativas = 0;
while (tentativas < 3)
{
    Console.WriteLine("Tentando...");
    tentativas++;
}
```

## 🔹 `do while`

Igual ao `while`, mas garante que o bloco execute **pelo menos uma vez**:

```csharp
int numero;
do
{
    numero = new Random().Next(1, 10);
} while (numero != 7);
```

## 🔹 `foreach`

Feito para percorrer coleções, sem se preocupar com índices:

```csharp
string[] nomes = { "Maria", "João", "Valentina" };

foreach (string nome in nomes)
{
    Console.WriteLine(nome);
}
```

👉 Use `foreach` sempre que não precisar alterar a coleção ou controlar o índice manualmente

---

# ⏹️ Controlando o loop: `break` e `continue`

- `break` → encerra o loop imediatamente  
- `continue` → pula para a próxima iteração, sem terminar o loop  

```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 5) break;       // para no 5
    if (i % 2 == 0) continue; // pula pares
    Console.WriteLine(i);
}
```

---

# ⚠️ Erros comuns

- Esquecer o `break` em cada `case` do `switch` tradicional  
- Criar loops infinitos por esquecer de atualizar a condição  
- Usar `for` quando um `foreach` deixaria o código mais claro  
- Abusar do operador ternário em condições complexas, prejudicando a leitura  

---

# 📌 Conclusão

- `if/else` decide com base em condições booleanas  
- `switch` (tradicional ou expression) organiza múltiplas opções  
- `for`, `while`, `do while` e `foreach` cobrem os diferentes cenários de repetição  
- `break` e `continue` dão controle fino sobre o fluxo do loop  

👉 Com condicionais e loops, seu programa finalmente consegue tomar decisões e repetir tarefas

---

# 🔥 Próximo passo

Agora que seu programa sabe decidir e repetir, o próximo nível é:

👉 **Fundamentos do C#: Métodos, Parâmetros e Retorno de Valores**

Aqui você vai aprender a organizar seu código em blocos reutilizáveis.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
