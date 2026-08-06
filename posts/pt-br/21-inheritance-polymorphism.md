# 🧠 Fundamentos do C#: Herança e Polimorfismo

⏱️ Tempo de leitura: 9 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Classes e objetos  
- Campos, propriedades, construtores e métodos  
- Encapsulamento como o primeiro pilar da OOP  

Agora vamos aos dois pilares que dão à OOP seu verdadeiro poder de reaproveitamento e flexibilidade:

👉 **Herança e Polimorfismo**

Assim como no post anterior, vale a pena ler com calma — esse é o tipo de conceito que separa quem só "escreve C#" de quem realmente entende orientação a objetos.

---

# 💡 O que é Herança?

👉 **Herança = uma classe reaproveitar campos, propriedades e métodos de outra**

Em vez de repetir código entre classes parecidas, você extrai o que é comum para uma classe base:

```csharp
class Animal
{
    public string Nome { get; set; }

    public void Dormir()
    {
        Console.WriteLine($"{Nome} está dormindo.");
    }
}

class Cachorro : Animal
{
    public void Latir()
    {
        Console.WriteLine($"{Nome} está latindo.");
    }
}
```

```csharp
Cachorro rex = new Cachorro();
rex.Nome = "Rex";       // herdado de Animal
rex.Dormir();            // herdado de Animal
rex.Latir();              // próprio de Cachorro
```

👉 `Cachorro` **é um** `Animal` — essa relação "é um" é o teste mais simples para saber se herança faz sentido

---

# 🔑 O construtor da classe base: `base`

Quando a classe base tem um construtor, a classe filha precisa chamá-lo:

```csharp
class Animal
{
    public string Nome { get; }

    public Animal(string nome)
    {
        Nome = nome;
    }
}

class Cachorro : Animal
{
    public Cachorro(string nome) : base(nome)
    {
        // construtor de Cachorro pode adicionar mais lógica aqui
    }
}
```

👉 `: base(nome)` garante que o construtor de `Animal` rode antes do de `Cachorro`

---

# 🔒 `protected`: acesso liberado para quem herda

Lembra do `private` do post anterior? Existe um meio-termo:

```csharp
class Animal
{
    protected int Energia = 100;
}

class Cachorro : Animal
{
    public void Correr()
    {
        Energia -= 10; // acessível porque é protected, não private
    }
}
```

👉 `protected` é invisível para o resto do mundo, mas visível para as classes que herdam

---

# ⚙️ Sobrescrevendo comportamento: `virtual` e `override`

Aqui mora um dos erros mais comuns de quem está aprendendo herança.

```csharp
class Animal
{
    public virtual void EmitirSom()
    {
        Console.WriteLine("Som genérico de animal");
    }
}

class Cachorro : Animal
{
    public override void EmitirSom()
    {
        Console.WriteLine("Au au!");
    }
}

class Gato : Animal
{
    public override void EmitirSom()
    {
        Console.WriteLine("Miau!");
    }
}
```

👉 `virtual` na classe base **permite** que a classe filha sobrescreva o método com `override`

⚠️ Sem `virtual` no método original, `override` na classe filha **não compila** — e se você usar `new` em vez de `override`, o método é apenas "escondido", não sobrescrito de verdade (mais sobre isso adiante).

---

# 🧩 Polimorfismo: a mágica de tratar tudo de forma uniforme

👉 **Polimorfismo = tratar objetos de tipos diferentes através de uma interface comum, deixando cada um se comportar do seu próprio jeito**

```csharp
List<Animal> animais = new List<Animal>
{
    new Cachorro(),
    new Gato(),
    new Animal()
};

foreach (Animal animal in animais)
{
    animal.EmitirSom();
}

// Au au!
// Miau!
// Som genérico de animal
```

👉 O `foreach` não sabe (nem precisa saber) se está lidando com um `Cachorro` ou um `Gato` — cada objeto sabe executar sua própria versão de `EmitirSom()`

Isso é chamado de **polimorfismo em tempo de execução**: o C# decide qual método rodar com base no **tipo real do objeto**, não no tipo da variável que o referencia.

---

# ⚠️ A armadilha: `override` vs `new`

Esse é o erro mais silencioso da herança em C#:

```csharp
class Animal
{
    public virtual void EmitirSom() => Console.WriteLine("Som genérico");
}

class Gato : Animal
{
    public new void EmitirSom() => Console.WriteLine("Miau!"); // "new", não "override"
}
```

```csharp
Animal animal = new Gato();
animal.EmitirSom(); // imprime "Som genérico" — não "Miau!"
```

👉 Com `new`, o método é apenas **escondido**, não sobrescrito. O comportamento real depende do **tipo da variável**, não do objeto — quebrando o polimorfismo

**Regra prática:** se a intenção é sobrescrever comportamento, sempre use `override` (e nunca `new`, a menos que você saiba exatamente por que está fazendo isso)

---

# 🚫 `sealed`: impedindo mais herança

```csharp
sealed class Robo
{
    // ninguém pode herdar de Robo
}

class RoboAvancado : Animal
{
    public sealed override void EmitirSom() // ninguém pode sobrescrever de novo
    {
        Console.WriteLine("Bip bip");
    }
}
```

👉 Use `sealed` quando você quer garantir que uma classe (ou um `override` específico) seja a versão final, sem mais especializações

---

# 🧱 Classes abstratas: modelos que não podem virar objeto

Às vezes a classe base não faz sentido sozinha — ela só existe para ser herdada:

```csharp
abstract class Forma
{
    public abstract double CalcularArea(); // sem corpo — cada filha implementa
}

class Circulo : Forma
{
    public double Raio { get; set; }

    public override double CalcularArea() => Math.PI * Raio * Raio;
}

class Retangulo : Forma
{
    public double Largura { get; set; }
    public double Altura { get; set; }

    public override double CalcularArea() => Largura * Altura;
}
```

```csharp
// Forma forma = new Forma(); // ❌ erro: não é possível instanciar uma classe abstrata

List<Forma> formas = new List<Forma>
{
    new Circulo { Raio = 2 },
    new Retangulo { Largura = 3, Altura = 4 }
};

foreach (Forma forma in formas)
{
    Console.WriteLine(forma.CalcularArea());
}
```

👉 `abstract` obriga toda classe filha a implementar aquele comportamento — é uma forma de garantir um "contrato" dentro da própria herança

---

# ⚠️ Erros comuns

- Esquecer `virtual` no método base e não entender por que `override` não compila  
- Usar `new` em vez de `override`, quebrando o polimorfismo silenciosamente  
- Criar cadeias de herança muito profundas (`A : B : C : D`), difíceis de entender e manter  
- Herdar só para reaproveitar código, sem que a relação "é um" realmente exista  
- Tentar instanciar uma classe `abstract` diretamente  

---

# 📌 Conclusão

- Herança reaproveita campos, propriedades e métodos entre classes relacionadas  
- `base` chama o construtor (ou membros) da classe pai  
- `virtual` + `override` permitem sobrescrever comportamento de verdade  
- Polimorfismo deixa cada objeto executar sua própria versão do comportamento, através de um tipo comum  
- Classes `abstract` definem contratos que as classes filhas são obrigadas a cumprir  

👉 Com herança e polimorfismo, seu código para de repetir lógica e passa a modelar relações reais entre conceitos

---

# 🔥 Próximo passo

Agora que você entende herança e polimorfismo, o próximo nível é:

👉 **Fundamentos do C#: Interfaces e Contratos**

Aqui você vai aprender uma forma ainda mais flexível de garantir comportamento entre classes que não têm relação de herança direta.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
