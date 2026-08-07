# 🧠 Fundamentos do C#: Princípios SOLID (Introdução ao Design de Software)

⏱️ Tempo de leitura: 13 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Classes, herança, polimorfismo e interfaces  
- Generics, delegates e os recursos modernos da linguagem  

Você já sabe escrever C# que **funciona**. A partir de agora, a pergunta muda: como escrever C# que continua fácil de manter daqui a um ano, com outras dez pessoas mexendo no mesmo código?

👉 **É exatamente isso que o SOLID resolve — e é por isso que ele é, sem dúvida, um dos temas mais importantes desta trilha**

Vamos com calma, exemplo por exemplo. Esse post merece ser lido devagar.

---

# 💡 O que é SOLID?

👉 **SOLID = cinco princípios de design orientado a objetos, criados para tornar o software mais fácil de entender, estender e manter**

O termo foi popularizado por Robert C. Martin (o "Uncle Bob") e cada letra representa um princípio:

- **S** — Single Responsibility Principle  
- **O** — Open/Closed Principle  
- **L** — Liskov Substitution Principle  
- **I** — Interface Segregation Principle  
- **D** — Dependency Inversion Principle  

👉 Nenhum desses princípios é uma regra rígida gravada em pedra — são **heurísticas**. O objetivo final é sempre o mesmo: reduzir acoplamento e aumentar coesão.

---

# 🔤 S — Single Responsibility Principle (Princípio da Responsabilidade Única)

👉 **Uma classe deve ter um, e apenas um, motivo para mudar**

## ❌ Antes

```csharp
class Pedido
{
    public List<string> Itens { get; set; } = new();

    public decimal CalcularTotal()
    {
        // lógica de cálculo
        return Itens.Count * 100;
    }

    public void SalvarNoBanco()
    {
        // lógica de acesso a dados
        Console.WriteLine("Salvando pedido no banco...");
    }

    public void EnviarEmailConfirmacao()
    {
        // lógica de envio de e-mail
        Console.WriteLine("Enviando e-mail de confirmação...");
    }
}
```

👉 Essa classe muda se a regra de cálculo mudar, se o banco de dados mudar, ou se o provedor de e-mail mudar. **Três motivos diferentes para mudar dentro da mesma classe**

## ✅ Depois

```csharp
class Pedido
{
    public List<string> Itens { get; set; } = new();

    public decimal CalcularTotal() => Itens.Count * 100;
}

class PedidoRepositorio
{
    public void Salvar(Pedido pedido) => Console.WriteLine("Salvando pedido no banco...");
}

class NotificadorEmail
{
    public void EnviarConfirmacao(Pedido pedido) => Console.WriteLine("Enviando e-mail de confirmação...");
}
```

👉 Agora cada classe tem **um único motivo para mudar**: `Pedido` muda se a regra de negócio mudar; `PedidoRepositorio` muda se a forma de persistir mudar; `NotificadorEmail` muda se a forma de notificar mudar

**Sinal de alerta:** se você descreve o que uma classe faz usando a palavra "e" ("calcula o total **e** salva **e** envia e-mail"), ela provavelmente está violando o SRP.

---

# 🔓 O — Open/Closed Principle (Princípio Aberto/Fechado)

👉 **Classes devem estar abertas para extensão, mas fechadas para modificação**

## ❌ Antes

```csharp
class CalculadoraDesconto
{
    public decimal Calcular(string tipoCliente, decimal valor)
    {
        if (tipoCliente == "Regular")
            return valor * 0.95m;
        else if (tipoCliente == "Premium")
            return valor * 0.90m;
        else if (tipoCliente == "VIP")
            return valor * 0.80m;

        return valor;
    }
}
```

👉 Todo novo tipo de cliente exige **abrir** essa classe e adicionar mais um `else if`. Isso é frágil — qualquer erro de digitação em um `case` já existente pode quebrar uma regra que já estava funcionando

## ✅ Depois

```csharp
interface IDesconto
{
    decimal Aplicar(decimal valor);
}

class DescontoRegular : IDesconto
{
    public decimal Aplicar(decimal valor) => valor * 0.95m;
}

class DescontoPremium : IDesconto
{
    public decimal Aplicar(decimal valor) => valor * 0.90m;
}

class DescontoVip : IDesconto
{
    public decimal Aplicar(decimal valor) => valor * 0.80m;
}

class CalculadoraDesconto
{
    public decimal Calcular(IDesconto desconto, decimal valor) => desconto.Aplicar(valor);
}
```

```csharp
var calculadora = new CalculadoraDesconto();
Console.WriteLine(calculadora.Calcular(new DescontoVip(), 1000)); // 800
```

👉 Um novo tipo de cliente vira uma **nova classe** que implementa `IDesconto` — `CalculadoraDesconto` nunca precisa ser tocada de novo. Isso é `interface` e polimorfismo (posts 24 e 22) trabalhando junto com design de software

---

# 🔄 L — Liskov Substitution Principle (Princípio da Substituição de Liskov)

👉 **Um objeto de uma classe filha deve poder substituir um objeto da classe pai sem quebrar o comportamento esperado**

Esse é o mais sutil dos cinco — e o exemplo clássico prova por quê.

## ❌ Antes

```csharp
class Retangulo
{
    public virtual double Largura { get; set; }
    public virtual double Altura { get; set; }

    public double CalcularArea() => Largura * Altura;
}

class Quadrado : Retangulo
{
    public override double Largura
    {
        get => base.Largura;
        set { base.Largura = value; base.Altura = value; } // efeito colateral inesperado
    }
}
```

```csharp
void TestarArea(Retangulo retangulo)
{
    retangulo.Largura = 5;
    retangulo.Altura = 10;

    Console.WriteLine(retangulo.CalcularArea()); // esperado: 50
}

TestarArea(new Quadrado()); // resultado real: 100 — quebrou a expectativa!
```

👉 `Quadrado` **é um** `Retangulo` na teoria matemática, mas na prática do código ele quebra o contrato: alterar `Largura` também altera `Altura` de forma inesperada. Quem usa `Retangulo` não pode confiar mais no comportamento

## ✅ Depois

```csharp
abstract class Forma
{
    public abstract double CalcularArea();
}

class Retangulo : Forma
{
    public double Largura { get; set; }
    public double Altura { get; set; }

    public override double CalcularArea() => Largura * Altura;
}

class Quadrado : Forma
{
    public double Lado { get; set; }

    public override double CalcularArea() => Lado * Lado;
}
```

👉 Em vez de forçar uma relação de herança que não se sustenta, cada forma implementa seu próprio contrato através de `Forma` (você já viu esse padrão no post sobre herança e polimorfismo)

**Regra prática:** se sobrescrever um método na classe filha exige "enfraquecer" ou mudar o comportamento esperado da classe pai, a herança provavelmente é o modelo errado.

---

# 🧩 I — Interface Segregation Principle (Princípio da Segregação de Interfaces)

👉 **Nenhuma classe deve ser forçada a implementar métodos que não usa**

## ❌ Antes

```csharp
interface IFuncionario
{
    void Trabalhar();
    void CalcularFerias();
    void ReceberBonus();
}

class Estagiario : IFuncionario
{
    public void Trabalhar() => Console.WriteLine("Estagiário trabalhando");
    public void CalcularFerias() => throw new NotImplementedException(); // não se aplica
    public void ReceberBonus() => throw new NotImplementedException();   // não se aplica
}
```

👉 `Estagiario` é obrigado a "implementar" métodos que não fazem sentido para ele — o contrato é grande demais para quem consome só uma parte dele

## ✅ Depois

```csharp
interface ITrabalhador
{
    void Trabalhar();
}

interface IElegivelParaFerias
{
    void CalcularFerias();
}

interface IElegivelParaBonus
{
    void ReceberBonus();
}

class Estagiario : ITrabalhador
{
    public void Trabalhar() => Console.WriteLine("Estagiário trabalhando");
}

class Gerente : ITrabalhador, IElegivelParaFerias, IElegivelParaBonus
{
    public void Trabalhar() => Console.WriteLine("Gerente trabalhando");
    public void CalcularFerias() => Console.WriteLine("Calculando férias");
    public void ReceberBonus() => Console.WriteLine("Recebendo bônus");
}
```

👉 Cada classe implementa só os contratos que fazem sentido pra ela — interfaces pequenas e específicas, exatamente como você viu no post sobre interfaces

---

# 🔌 D — Dependency Inversion Principle (Princípio da Inversão de Dependência)

👉 **Módulos de alto nível não devem depender de módulos de baixo nível — ambos devem depender de abstrações**

## ❌ Antes

```csharp
class ServicoEmail
{
    public void Enviar(string mensagem) => Console.WriteLine($"E-mail: {mensagem}");
}

class ProcessadorPedido
{
    private readonly ServicoEmail _servicoEmail = new(); // dependência concreta, criada aqui dentro

    public void Processar()
    {
        _servicoEmail.Enviar("Pedido processado!");
    }
}
```

👉 `ProcessadorPedido` está "amarrado" a `ServicoEmail`. Se amanhã você quiser notificar por SMS em vez de e-mail, precisa **modificar** `ProcessadorPedido`

## ✅ Depois

```csharp
interface INotificador
{
    void Enviar(string mensagem);
}

class ServicoEmail : INotificador
{
    public void Enviar(string mensagem) => Console.WriteLine($"E-mail: {mensagem}");
}

class ServicoSms : INotificador
{
    public void Enviar(string mensagem) => Console.WriteLine($"SMS: {mensagem}");
}

class ProcessadorPedido
{
    private readonly INotificador _notificador;

    public ProcessadorPedido(INotificador notificador) // dependência injetada de fora
    {
        _notificador = notificador;
    }

    public void Processar()
    {
        _notificador.Enviar("Pedido processado!");
    }
}
```

```csharp
var processador = new ProcessadorPedido(new ServicoSms());
processador.Processar(); // SMS: Pedido processado!
```

👉 `ProcessadorPedido` depende só da **abstração** `INotificador` — trocar `ServicoEmail` por `ServicoSms` não exige tocar em uma linha do processador

Isso é exatamente o mecanismo por trás da **injeção de dependência**, um dos pilares de qualquer aplicação ASP.NET Core moderna: em vez de uma classe criar suas próprias dependências, elas chegam prontas de fora — geralmente via construtor, como acabamos de ver.

---

# 📋 Resumo rápido

| Letra | Princípio | Em uma frase |
|---|---|---|
| **S** | Single Responsibility | Uma classe, um motivo para mudar |
| **O** | Open/Closed | Estenda com novas classes, não editando as existentes |
| **L** | Liskov Substitution | A classe filha não pode quebrar o contrato da classe pai |
| **I** | Interface Segregation | Interfaces pequenas e específicas, não uma gigante |
| **D** | Dependency Inversion | Dependa de abstrações, não de implementações concretas |

---

# ⚠️ Erros comuns

- Aplicar SOLID de forma dogmática em projetos pequenos, criando abstração demais para um problema simples  
- Confundir Dependency Inversion com "usar um container de DI" — o princípio é sobre depender de abstrações, o container é só uma ferramenta que facilita isso  
- Achar que SRP significa "uma classe deve ter um único método" — o princípio é sobre **motivo para mudar**, não sobre quantidade de código  
- Ignorar SOLID completamente e só perceber o custo disso meses depois, quando qualquer mudança pequena quebra três outras partes do sistema  

---

# 📌 Conclusão

- **SRP**: cada classe deve ter um único motivo para mudar  
- **OCP**: estenda comportamento com novas classes, sem modificar o que já existe e funciona  
- **LSP**: uma classe filha nunca deve quebrar as expectativas de quem usa a classe pai  
- **ISP**: prefira várias interfaces pequenas a uma única interface gigante  
- **DIP**: dependa de abstrações (interfaces), não de implementações concretas  

👉 SOLID não é sobre decorar siglas — é sobre um objetivo único: código fácil de estender sem medo de quebrar o que já funciona. Depois de entender isso de verdade, você não escreve C# do mesmo jeito nunca mais.

---

# 🔥 Próximo passo

Agora que você domina os princípios de design mais importantes da carreira, o próximo nível é:

👉 **Fundamentos do C#: Design Patterns Essenciais (Singleton, Factory, Strategy e Repository)**

Aqui você vai ver como o SOLID se materializa em padrões de projeto usados todos os dias no mercado.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
