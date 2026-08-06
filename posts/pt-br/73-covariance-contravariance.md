# 🧠 Fundamentos do C#: Covariância e Contravariância

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Programação funcional e composição de funções  
- Generics (post 24) e herança/polimorfismo (post 21)  

Você já escreveu código assim sem parar para pensar por quê funciona: `IEnumerable<string>` pode ser usado onde se espera `IEnumerable<object>`. Isso não é acidente — é covariância, e tem regras precisas.

👉 **Vamos aprender Covariância e Contravariância**

---

# 💡 O problema que motiva isso

```csharp
public class Animal { }
public class Cachorro : Animal { }

List<Cachorro> cachorros = new();
List<Animal> animais = cachorros; // ❌ não compila!
```

👉 Mesmo `Cachorro` sendo um `Animal` (lembra da herança do post 21?), `List<Cachorro>` **não** é automaticamente um `List<Animal>` — porque listas são mutáveis, e permitir isso quebraria a segurança de tipos

```csharp
animais.Add(new Gato()); // se isso compilasse, um List<Cachorro> teria um Gato dentro!
```

---

# 🏗️ Covariância: quando é seguro (só leitura)

```csharp
IEnumerable<Cachorro> cachorros = new List<Cachorro> { new Cachorro() };
IEnumerable<Animal> animais = cachorros; // ✅ compila!

foreach (Animal animal in animais)
{
    Console.WriteLine(animal.GetType().Name);
}
```

👉 `IEnumerable<T>` é covariante (declarado como `IEnumerable<out T>`) porque é **somente leitura** — você só consegue ler itens dele, nunca adicionar. Sem risco de colocar um `Gato` onde deveria ter um `Cachorro`

---

# 🎯 Contravariância: o inverso, para entrada

```csharp
public class ComparadorDeAnimais : IComparer<Animal>
{
    public int Compare(Animal x, Animal y) => 0; // exemplo simplificado
}

IComparer<Cachorro> comparador = new ComparadorDeAnimais(); // ✅ compila!
```

👉 `IComparer<T>` é contravariante (`IComparer<in T>`) — um comparador que sabe comparar **qualquer** `Animal` também sabe comparar `Cachorro` especificamente. A direção é invertida em relação à covariância

---

# 🔍 Onde você já usou isso, sem perceber

```csharp
Func<Animal, string> descreverAnimal = a => $"Animal: {a.GetType().Name}";
Func<Cachorro, string> descreverCachorro = descreverAnimal; // ✅ contravariância no parâmetro

Action<Animal> processarAnimal = a => Console.WriteLine(a);
Action<Cachorro> processarCachorro = processarAnimal; // ✅ mesma lógica
```

👉 Lembra do post sobre delegates (25)? `Func<in T, out TResult>` combina os dois: contravariante no parâmetro de entrada, covariante no valor de retorno — essa é a regra geral: **saída covariante, entrada contravariante**

---

# ⚠️ Erros comuns

- Tentar aplicar covariância em coleções mutáveis (`List<T>`, `T[]`), esperando o mesmo comportamento de `IEnumerable<T>`  
- Confundir a direção — lembrar que "covariante" segue a mesma direção da herança (out), "contravariante" é o oposto (in)  
- Marcar um generic como `out` ou `in` sem garantir que a interface realmente respeita essas restrições, gerando erros de compilação difíceis de entender  
- Ignorar que arrays em C# são covariantes por herança histórica da linguagem, mas de forma insegura (podem lançar `ArrayTypeMismatchException` em runtime)  

---

# 📌 Conclusão

- Covariância (`out`) permite substituir um tipo mais derivado onde um mais genérico é esperado, em contextos somente leitura  
- Contravariância (`in`) permite o inverso, em contextos de entrada/parâmetros  
- `IEnumerable<T>`, `Func<>` e `IComparer<T>` já usam essas regras nos bastidores  
- A regra geral: saída covariante, entrada contravariante  

👉 Com covariância e contravariância, você entende por que certas conversões de generics compilam e outras não — não é inconsistência do compilador, é segurança de tipos em ação

---

# 🔥 Próximo passo

Agora que você entende variância em generics, o próximo nível é:

👉 **Fundamentos do C#: Expression Trees**

Aqui você vai aprender como o C# representa código como dados, a base do LINQ to SQL e de bibliotecas de ORM.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
