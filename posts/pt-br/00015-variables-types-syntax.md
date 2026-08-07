# 🧠 Fundamentos do C#: Variáveis, Tipos e Sintaxe Básica

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- O que é IL, CLR e como o .NET executa seu código  
- A diferença entre SDK e Runtime  
- Como criar e rodar seu primeiro programa  

Agora é hora de aprender o que todo programa precisa fazer:

👉 **Guardar e manipular dados**

---

# 💡 O que é uma variável?

👉 **Variável = um espaço nomeado na memória que guarda um valor**

```csharp
int idade = 30;
```

Aqui você tem três partes:

- `int` → o **tipo** do dado  
- `idade` → o **nome** da variável  
- `30` → o **valor** atribuído  

👉 C# é uma linguagem **fortemente tipada**: toda variável tem um tipo definido, e o compilador verifica isso antes mesmo do programa rodar

---

# 🧱 Tipos mais usados no dia a dia

## 🔹 Números
- `int` → números inteiros (`10`, `-5`)  
- `double` → números com casas decimais (`3.14`)  
- `decimal` → precisão alta, ideal para valores financeiros  

## 🔹 Texto e caracteres
- `string` → texto (`"Olá, mundo"`)  
- `char` → um único caractere (`'A'`)  

## 🔹 Lógico
- `bool` → `true` ou `false`  

👉 Escolher o tipo certo evita bugs sutis, principalmente com números decimais e dinheiro

---

# 🔀 Tipos por valor vs tipos por referência

Essa é uma diferença fundamental em C#:

## 🔹 Tipos por valor (value types)
- `int`, `double`, `bool`, `struct`  
- Guardam o dado diretamente  
- Copiar a variável copia o valor  

## 🔹 Tipos por referência (reference types)
- `string`, `class`, arrays, objetos  
- Guardam um **endereço** que aponta para o dado  
- Copiar a variável copia a referência, não o dado  

👉 Entender isso evita surpresas quando você altera um objeto e a mudança "aparece" em outro lugar do código

---

# ⚙️ Inferência de tipo com `var`

Você pode deixar o compilador descobrir o tipo:

```csharp
var nome = "Vitor";     // vira string
var total = 99.90;      // vira double
```

👉 `var` não torna a variável dinâmica — o tipo é definido em tempo de compilação e não muda depois

## 🔹 Quando usar `var`

- Quando o tipo já é óbvio pelo valor  
- Quando o nome da variável já deixa claro o que ela representa  

👉 Em caso de dúvida, seja explícito — clareza vence "economia de digitação"

---

# 🔒 Constantes e valores imutáveis

Nem todo valor deve poder mudar:

```csharp
const double Pi = 3.14159;
readonly string AppName = "MeuApp";
```

## 🔹 `const`
- Valor fixo, definido em tempo de compilação  

## 🔹 `readonly`
- Só pode ser definido no construtor  
- Útil para valores que variam por instância, mas não devem mudar depois  

---

# 🔄 Conversão entre tipos

Às vezes você precisa transformar um tipo em outro:

```csharp
double preco = 10;              // conversão implícita (int → double)
int quantidade = (int)9.9;      // conversão explícita (cast) → 9
string texto = "42";
int numero = int.Parse(texto);  // conversão de string para int
```

## 🔹 Implícita vs explícita

- **Implícita**: segura, sem perda de dados (`int` → `double`)  
- **Explícita (cast)**: pode perder dados, exige `(tipo)` na frente  

👉 Conversões de `string` sempre podem falhar — use `int.TryParse` quando o valor não é garantido

---

# ⚠️ Erros comuns

- Usar `double` para dinheiro em vez de `decimal`  
- Achar que `var` torna C# uma linguagem "sem tipos"  
- Confundir cópia de valor com cópia de referência em objetos  
- Usar `Parse` sem validar a entrada, quebrando o app com uma exceção  

---

# 📌 Conclusão

- Toda variável em C# tem um **tipo** definido  
- Tipos por valor copiam o dado; tipos por referência copiam o endereço  
- `var` infere o tipo, mas não elimina a tipagem forte  
- `const` e `readonly` protegem valores que não devem mudar  

👉 Dominar variáveis e tipos é a base para escrever qualquer lógica em C#

---

# 🔥 Próximo passo

Agora que você sabe guardar e manipular dados, o próximo nível é:

👉 **Fundamentos do C#: Estruturas de Controle (if, else, switch e loops)**

Aqui você vai aprender a controlar o fluxo do seu programa com decisões e repetições.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
