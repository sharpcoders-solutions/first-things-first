# 🧠 Fundamentos do C#: Interfaces e Contratos

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Classes, objetos e encapsulamento  
- Herança e polimorfismo com `virtual`/`override`  
- Classes abstratas como contratos dentro de uma hierarquia  

Mas nem toda relação entre classes é uma relação de herança. Às vezes você só quer garantir que **classes completamente diferentes** sigam a mesma regra.

👉 **É para isso que existem as interfaces**

---

# 💡 O que é uma interface?

👉 **Interface = um contrato que define o que uma classe deve fazer, sem dizer como**

```csharp
interface IAnimal
{
    void EmitirSom();
}
```

```csharp
class Cachorro : IAnimal
{
    public void EmitirSom() => Console.WriteLine("Au au!");
}

class Robo : IAnimal
{
    public void EmitirSom() => Console.WriteLine("Bip bip!");
}
```

👉 `Cachorro` e `Robo` não têm nenhuma relação de herança entre si — mas ambos **cumprem o contrato** `IAnimal`, comprometendo-se a fornecer uma implementação para o método EmitirSom().

Note a convenção: interfaces em C# começam com `I` maiúsculo. Isso não é obrigatório, mas é praticamente universal no ecossistema .NET.

---

# 🔀 Interface vs Classe Abstrata: a dúvida mais comum

Essa é a pergunta que todo mundo faz ao aprender interfaces. As duas parecem "contratos", mas resolvem problemas diferentes.

## 🔹 Classe abstrata
- Pode ter **implementação** e **estado** (campos, propriedades já prontas)  
- Uma classe só pode herdar de **uma** classe abstrata  
- Faz sentido quando existe uma relação forte de "é um" com comportamento compartilhado  

## 🔹 Interface
- Só define **o que** deve existir (por padrão, sem implementação própria)  
- Uma classe pode implementar **várias** interfaces  
- Faz sentido quando classes não relacionadas precisam seguir a mesma regra  

```csharp
// Classe abstrata: FUNCIONÁRIO é a base de um relacionamento "é um"
abstract class Funcionario
{
    public string Nome { get; set; }
    public abstract decimal CalcularSalario();
}

// Interface: capacidades que não têm relação hierárquica
interface IAuditavel
{
    void RegistrarAuditoria();
}

interface INotificavel
{
    void EnviarNotificacao(string mensagem);
}

class Gerente : Funcionario, IAuditavel, INotificavel
{
    public override decimal CalcularSalario() => 8000;
    public void RegistrarAuditoria() => Console.WriteLine("Auditoria registrada");
    public void EnviarNotificacao(string mensagem) => Console.WriteLine(mensagem);
}
```

👉 `Gerente` herda de **uma** classe (`Funcionario`) e implementa **várias** interfaces (`IAuditavel`, `INotificavel`) — isso não seria possível só com herança

---

# 🧩 Múltiplas interfaces: contornando a ausência de herança múltipla

O C# não permite que uma classe herde de duas classes ao mesmo tempo, mas permite implementar quantas interfaces forem necessárias:

```csharp
interface IVoador
{
    void Voar();
}

interface INadador
{
    void Nadar();
}

class Pato : IVoador, INadador
{
    public void Voar() => Console.WriteLine("Pato voando");
    public void Nadar() => Console.WriteLine("Pato nadando");
}
```

👉 Isso resolve um problema real: e se um objeto precisasse se comportar como duas coisas ao mesmo tempo, sem que essas coisas tivessem uma relação hierárquica entre si?

---

# 📐 Interfaces como contrato: desacoplando código

O maior valor de uma interface aparece quando você programa **para o contrato**, não para a implementação concreta:

```csharp
interface IPagamento
{
    bool Processar(decimal valor);
}

class PagamentoCartao : IPagamento
{
    public bool Processar(decimal valor)
    {
        Console.WriteLine($"Processando R$ {valor} no cartão");
        return true;
    }
}

class PagamentoPix : IPagamento
{
    public bool Processar(decimal valor)
    {
        Console.WriteLine($"Processando R$ {valor} via Pix");
        return true;
    }
}

class Checkout
{
    private readonly IPagamento _pagamento;

    public Checkout(IPagamento pagamento)
    {
        _pagamento = pagamento;
    }

    public void Finalizar(decimal total)
    {
        _pagamento.Processar(total);
    }
}
```

```csharp
var checkoutCartao = new Checkout(new PagamentoCartao());
checkoutCartao.Finalizar(150);

var checkoutPix = new Checkout(new PagamentoPix());
checkoutPix.Finalizar(150);
```

👉 `Checkout` não sabe (e não precisa saber) se está lidando com cartão, Pix ou boleto — ele só conhece o contrato `IPagamento`

Isso é a base de um princípio muito usado em times profissionais: **programe para interfaces, não para implementações**. É esse desacoplamento que torna o código mais fácil de testar e de estender sem quebrar o que já existe.

---

# 🔌 Métodos padrão em interfaces (C# 8+)

Versões modernas do C# permitem que uma interface tenha uma implementação padrão:

```csharp
interface INotificavel
{
    void EnviarNotificacao(string mensagem);

    void EnviarAlerta() // método com implementação padrão
    {
        EnviarNotificacao("⚠️ Alerta padrão");
    }
}
```

👉 Útil para evoluir interfaces sem quebrar classes que já as implementam — mas use com moderação, o objetivo principal de uma interface continua sendo definir contrato, não comportamento

---

# ⚠️ Erros comuns

- Criar interfaces gigantes, com métodos demais ("interface Deus") — prefira interfaces pequenas e focadas  
- Confundir interface com classe abstrata e usar a errada para o problema  
- Esquecer de implementar **todos** os membros da interface (o código simplesmente não compila)  
- Depender da implementação concreta em vez da interface, perdendo o desacoplamento  

---

# 📌 Conclusão

- Interface define **o que** uma classe deve fazer, sem impor **como**  
- Uma classe pode implementar várias interfaces, mas herdar de apenas uma classe  
- Interfaces desacoplam código: quem consome só precisa conhecer o contrato  
- Classe abstrata compartilha implementação numa relação "é um"; interface garante comportamento entre classes não relacionadas  

👉 Com interfaces, seu código fica pronto para crescer e mudar sem quebrar o que já funciona

---

# 🔥 Próximo passo

Agora que você entende contratos entre classes, o próximo nível é:

👉 **Fundamentos do C#: Tratamento de Exceções (try, catch, finally e Exceções Customizadas)**

Aqui você vai aprender a lidar com erros de forma segura e profissional.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
