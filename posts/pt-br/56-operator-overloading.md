# 🧠 Fundamentos do C#: Operator Overloading

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como `foreach` funciona por baixo dos panos, via `IEnumerable<T>`  
- `yield return` e lazy evaluation em iteradores customizados  

Você já usou `+` para somar `int`s e `decimal`s a vida inteira. Mas sabia que pode ensinar o compilador a entender `+`, `==`, `<` e outros operadores para os **seus próprios tipos**? É isso que operator overloading faz.

👉 **Vamos aprender a sobrecarregar operadores em C#**

---

# 💡 O problema: operadores não fazem sentido em tipos customizados por padrão

```csharp
public class Dinheiro
{
    public decimal Valor { get; }
    public string Moeda { get; }

    public Dinheiro(decimal valor, string moeda)
    {
        Valor = valor;
        Moeda = moeda;
    }
}

var a = new Dinheiro(10, "BRL");
var b = new Dinheiro(20, "BRL");

var total = a + b; // ❌ Erro de compilação: operador + não existe para Dinheiro
```

👉 O compilador não sabe, por padrão, o que significa "somar" dois objetos `Dinheiro` — você precisa ensinar isso explicitamente

---

# ➕ Sobrecarregando o operador `+`

```csharp
public class Dinheiro
{
    public decimal Valor { get; }
    public string Moeda { get; }

    public Dinheiro(decimal valor, string moeda)
    {
        Valor = valor;
        Moeda = moeda;
    }

    public static Dinheiro operator +(Dinheiro a, Dinheiro b)
    {
        if (a.Moeda != b.Moeda)
            throw new InvalidOperationException("Não é possível somar moedas diferentes");

        return new Dinheiro(a.Valor + b.Valor, a.Moeda);
    }
}

var a = new Dinheiro(10, "BRL");
var b = new Dinheiro(20, "BRL");
var total = a + b; // ✅ agora funciona: Dinheiro { Valor = 30, Moeda = "BRL" }
```

👉 **`operator +` = um método estático especial que define o que o símbolo `+` significa quando aplicado ao seu tipo**

O corpo é código C# normal — nada impede você de validar regras de negócio (como impedir somar moedas diferentes) dentro do próprio operador

---

# ⚖️ Sobrecarregando comparações: `==` e `!=`

```csharp
public class Dinheiro
{
    public decimal Valor { get; }
    public string Moeda { get; }

    // ... construtor ...

    public static bool operator ==(Dinheiro a, Dinheiro b)
    {
        if (a is null || b is null)
            return a is null && b is null;

        return a.Valor == b.Valor && a.Moeda == b.Moeda;
    }

    public static bool operator !=(Dinheiro a, Dinheiro b) => !(a == b);

    public override bool Equals(object obj) => obj is Dinheiro outro && this == outro;
    public override int GetHashCode() => HashCode.Combine(Valor, Moeda);
}
```

👉 **Regra obrigatória: sempre que você sobrecarrega `==` e `!=`, também precisa sobrescrever `Equals` e `GetHashCode`** — os três precisam concordar entre si, ou você acaba com um tipo cujo comportamento de igualdade é inconsistente dependendo de como ele é comparado

Lembra dos `record`s? Eles fazem exatamente isso automaticamente — geram `==`, `Equals` e `GetHashCode` consistentes por baixo dos panos, sem você escrever nada disso manualmente

---

# 🔢 Operadores de comparação relacional: `<`, `>`, `<=`, `>=`

```csharp
public class Dinheiro : IComparable<Dinheiro>
{
    public decimal Valor { get; }

    // ...

    public static bool operator <(Dinheiro a, Dinheiro b) => a.Valor < b.Valor;
    public static bool operator >(Dinheiro a, Dinheiro b) => a.Valor > b.Valor;
    public static bool operator <=(Dinheiro a, Dinheiro b) => a.Valor <= b.Valor;
    public static bool operator >=(Dinheiro a, Dinheiro b) => a.Valor >= b.Valor;

    public int CompareTo(Dinheiro outro) => Valor.CompareTo(outro.Valor);
}

var lista = new List<Dinheiro> { /* ... */ };
lista.Sort(); // funciona porque Dinheiro implementa IComparable<Dinheiro>
```

👉 Implementar `IComparable<T>` junto com os operadores relacionais permite que seu tipo funcione com `List<T>.Sort()`, `OrderBy` do LINQ, e qualquer API que espera comparação de ordem

---

# 🎯 Conversões customizadas: `implicit` e `explicit`

```csharp
public class Dinheiro
{
    public decimal Valor { get; }
    public string Moeda { get; }

    public Dinheiro(decimal valor, string moeda) { Valor = valor; Moeda = moeda; }

    public static implicit operator decimal(Dinheiro dinheiro) => dinheiro.Valor;
    public static explicit operator Dinheiro(decimal valor) => new Dinheiro(valor, "BRL");
}

decimal valor = new Dinheiro(100, "BRL"); // implícita: conversão automática, sem cast
Dinheiro dinheiro = (Dinheiro)50m;         // explícita: exige cast, porque assume moeda "BRL"
```

👉 **Regra prática: use `implicit` só quando a conversão nunca perde informação e nunca pode falhar. Use `explicit` quando a conversão é "arriscada" ou assume alguma coisa** — como no exemplo acima, converter `decimal` para `Dinheiro` exige assumir uma moeda, o que é arriscado o suficiente para exigir um cast explícito

---

# ⚠️ Erros comuns

- Sobrecarregar `==` sem sobrescrever `Equals` e `GetHashCode`, criando um tipo com comportamento de igualdade inconsistente  
- Usar `implicit` para conversões que podem falhar ou perder precisão, surpreendendo quem usa o tipo  
- Sobrecarregar operadores para tipos que não representam um conceito matemático ou de comparação natural, tornando o código mais confuso, não mais claro  
- Esquecer que `Dinheiro a = null` faz `a == b` chamar o operador sobrecarregado — sem checar `null` primeiro, isso gera `NullReferenceException`  

---

# 📌 Conclusão

- Operadores sobrecarregados são métodos estáticos especiais que ensinam o compilador a lidar com `+`, `==`, `<` no seu tipo  
- Sobrecarregar `==`/`!=` exige também sobrescrever `Equals` e `GetHashCode`, para manter consistência  
- `IComparable<T>` junto com operadores relacionais habilita ordenação nativa (`Sort`, `OrderBy`)  
- `implicit` para conversões seguras; `explicit` para conversões que exigem atenção explícita de quem chama  

👉 Operadores customizados fazem seus tipos se comportarem como cidadãos de primeira classe da linguagem — o próximo recurso permite algo parecido: fazer seus tipos serem acessados com colchetes, como um array

---

# 🔥 Próximo passo

Agora que você sabe ensinar operadores aos seus tipos, o próximo nível é:

👉 **Fundamentos do C#: Indexers em Tipos Customizados**

Aqui você vai aprender a usar a sintaxe `objeto[indice]` nos seus próprios tipos, do mesmo jeito que já usa em arrays e listas.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
