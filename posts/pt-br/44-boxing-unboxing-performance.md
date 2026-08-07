# 🧠 Fundamentos do C#: Boxing, Unboxing e Performance de Tipos

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como value types e reference types se comportam de forma diferente  
- Onde cada um vive na memória: stack vs heap  

Você sabe que `int` é um value type. Mas já parou pra pensar no que acontece quando você guarda um `int` numa `ArrayList`, ou passa ele como `object` para um método? Existe um custo escondido nessa conversão, e ele aparece exatamente nos lugares onde performance costuma importar.

👉 **Vamos entender boxing, unboxing, e como evitar o custo deles**

---

# 💡 O que é boxing?

```csharp
int numero = 42;
object caixa = numero; // BOXING: o valor 42 é copiado para o heap
```

👉 **Boxing = empacotar um value type dentro de um objeto no heap, para que ele possa ser tratado como `object`**

Quando isso acontece, o runtime aloca um novo objeto no heap, copia o valor do `int` para dentro dele, e a variável `caixa` guarda uma referência para esse "pacote". O `int` original na stack continua existindo, intocado — mas agora existe uma **cópia inteiramente separada** no heap.

---

# 📦 Unboxing: o caminho de volta

```csharp
object caixa = 42;
int numero = (int)caixa; // UNBOXING: copia o valor de volta para a stack
```

👉 **Unboxing = extrair o value type de volta de dentro do objeto, com uma verificação de tipo em tempo de execução**

Se o tipo dentro da caixa não bater exatamente com o tipo esperado, o unboxing lança `InvalidCastException`:

```csharp
object caixa = 42;
long numero = (long)caixa; // ❌ InvalidCastException — a caixa contém int, não long
```

---

# 💸 Por que isso custa caro

```csharp
// Cenário com boxing implícito
var lista = new System.Collections.ArrayList();
for (int i = 0; i < 1_000_000; i++)
{
    lista.Add(i); // boxing em CADA iteração
}
```

👉 `ArrayList` guarda `object`, então cada `int` adicionado sofre boxing — um milhão de alocações no heap, uma por número, além do trabalho extra do garbage collector para depois liberar tudo isso

```csharp
// Cenário sem boxing
var lista = new List<int>();
for (int i = 0; i < 1_000_000; i++)
{
    lista.Add(i); // nenhum boxing — List<int> guarda int diretamente
}
```

👉 `List<T>` é genérica: quando `T` é `int`, ela guarda `int`s de verdade, sem converter para `object`. Essa é uma das razões pelas quais coleções genéricas (`List<T>`, `Dictionary<TKey, TValue>`) substituíram completamente as coleções não genéricas (`ArrayList`, `Hashtable`) do início do .NET

---

# 🕵️ Onde boxing aparece escondido

```csharp
// Interpolação de string com value types — sem boxing no C# moderno,
// mas string.Format ainda pode gerar boxing dependendo do overload
Console.WriteLine("Valor: {0}", 42); // pode causar boxing dependendo da assinatura usada

// Passando um struct para um parâmetro object
void Registrar(object valor) { }
Registrar(42); // boxing — 42 vira object

// Guardando value types em coleções não genéricas antigas
Hashtable tabela = new Hashtable();
tabela["chave"] = 42; // boxing
```

👉 O padrão comum: **sempre que um value type precisa ser tratado como `object` (ou como uma interface que ele implementa), acontece boxing**. Até implementar uma interface pode causar boxing:

```csharp
interface IIdentificavel { int Id { get; } }
struct Item : IIdentificavel { public int Id { get; set; } }

void Processar(IIdentificavel item) { } // recebe pela interface

var item = new Item { Id = 1 };
Processar(item); // boxing — struct convertido para a interface
```

---

# ✅ Como evitar boxing desnecessário

- **Prefira coleções genéricas** (`List<T>`, `Dictionary<TKey, TValue>`) em vez das não genéricas (`ArrayList`, `Hashtable`)  
- **Evite parâmetros `object` para value types** quando um genérico (`T`) resolve o mesmo problema sem conversão  
- **Cuidado com LINQ sobre `IEnumerable` não genérico** — iterar coleções antigas com LINQ pode introduzir boxing silenciosamente  
- **Em código realmente sensível a performance** (loops apertados, hot paths), meça com `BenchmarkDotNet` antes de assumir que boxing é o problema — otimização prematura tem seu próprio custo  

```csharp
// Genérico evita boxing completamente
void Processar<T>(T valor) where T : IIdentificavel { }
```

👉 Usar um método genérico com constraint de interface, em vez de receber a interface diretamente, é o jeito de ter polimorfismo sem pagar o custo do boxing — o compilador gera uma versão especializada do método para cada tipo de valor usado

---

# ⚠️ Erros comuns

- Usar `ArrayList` ou `Hashtable` em código novo, sem perceber o boxing implícito em cada inserção  
- Assumir que todo `object valor` em uma assinatura de método é "só um detalhe de tipo", ignorando o custo de alocação  
- Otimizar boxing em código que roda uma vez por request, onde o custo é irrelevante perto de uma chamada de rede ou banco de dados  
- Fazer unboxing para o tipo errado e ser surpreendido por uma `InvalidCastException` em produção  

---

# 📌 Conclusão

- Boxing empacota um value type dentro de um objeto no heap; unboxing faz o caminho inverso  
- Cada boxing é uma alocação no heap — caro quando acontece em loops ou código de alta frequência  
- Coleções genéricas (`List<T>`) evitam boxing que coleções não genéricas (`ArrayList`) sempre pagam  
- Implementar uma interface em um struct e passá-lo pela interface também causa boxing  
- Meça antes de otimizar: boxing só importa de verdade em hot paths reais  

👉 Com value types, reference types, e agora boxing/unboxing dominados, você tem a base completa para entender por que o C# se comporta do jeito que se comporta em tempo de execução

---

# 🔥 Próximo passo

Agora que você entende o custo real das conversões entre value e reference types, o próximo nível é:

👉 **Fundamentos do C#: gRPC — Comunicação Eficiente entre Serviços**

Aqui você vai aprender uma alternativa mais rápida e tipada que REST para comunicação entre serviços.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
