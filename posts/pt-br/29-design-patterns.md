# 🧠 Fundamentos do C#: Design Patterns Essenciais (Singleton, Factory, Strategy e Repository)

⏱️ Tempo de leitura: 9 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Os cinco princípios SOLID  
- Como interfaces e polimorfismo desacoplam código  

SOLID te dá os **princípios**. Design patterns são as **soluções prontas** que o mercado já testou e aprovou para problemas que se repetem em praticamente todo sistema.

👉 **Vamos conhecer quatro padrões que você vai encontrar em praticamente todo projeto C# profissional**

---

# 💡 O que é um Design Pattern?

👉 **Design pattern = uma solução reutilizável para um problema recorrente de design de software**

Não é código pronto para copiar e colar — é um **modelo**, uma forma comprovada de resolver um tipo de problema, catalogada originalmente pelo livro do "Gang of Four" (GoF) nos anos 90.

---

# 🔒 Singleton

👉 **Garante que uma classe tenha apenas uma instância durante toda a execução do programa**

```csharp
class ConfiguracaoApp
{
    private static readonly ConfiguracaoApp _instancia = new();

    public static ConfiguracaoApp Instancia => _instancia;

    public string AmbienteAtual { get; set; } = "Produção";

    private ConfiguracaoApp() { } // construtor privado — ninguém cria de fora
}
```

```csharp
ConfiguracaoApp.Instancia.AmbienteAtual = "Homologação";
Console.WriteLine(ConfiguracaoApp.Instancia.AmbienteAtual); // Homologação
```

## 🔹 Como funciona

- Construtor `private` → impede `new ConfiguracaoApp()` de fora da classe  
- Campo `static readonly` → garante que só existe uma instância, criada uma única vez  
- Propriedade estática pública → ponto único de acesso  

## ⚠️ Use com cautela

👉 Singleton é útil para configuração global, cache ou logging — mas é fácil abusar dele e transformá-lo em **estado global disfarçado**, dificultando testes automatizados (você não consegue trocar a instância por um mock facilmente)

**Regra prática:** em aplicações ASP.NET Core modernas, prefira registrar a dependência como singleton no container de injeção de dependência em vez de codificar o padrão manualmente — você ganha o mesmo comportamento sem perder testabilidade.

---

# 🏭 Factory

👉 **Centraliza a lógica de criação de objetos, escondendo os detalhes de "como" e "qual" instanciar**

Lembra do exemplo de `IDesconto` no post sobre SOLID? Vamos usar uma factory para decidir qual implementação criar:

```csharp
interface IDesconto
{
    decimal Aplicar(decimal valor);
}

class DescontoRegular : IDesconto
{
    public decimal Aplicar(decimal valor) => valor * 0.95m;
}

class DescontoVip : IDesconto
{
    public decimal Aplicar(decimal valor) => valor * 0.80m;
}

class DescontoFactory
{
    public static IDesconto Criar(string tipoCliente) => tipoCliente switch
    {
        "VIP" => new DescontoVip(),
        _ => new DescontoRegular()
    };
}
```

```csharp
IDesconto desconto = DescontoFactory.Criar("VIP");
Console.WriteLine(desconto.Aplicar(1000)); // 800
```

👉 Quem chama `DescontoFactory.Criar` não precisa saber quais classes existem nem como elas são construídas — só pede o resultado e recebe algo que cumpre o contrato `IDesconto`

Isso reforça o **Open/Closed Principle**: novos tipos de desconto viram novas classes + uma linha na factory, sem alterar quem consome `IDesconto`.

---

# 🎯 Strategy

👉 **Encapsula algoritmos intercambiáveis atrás de uma interface comum, permitindo trocar o comportamento em tempo de execução**

Aqui vai uma revelação: você **já viu** o padrão Strategy no post sobre SOLID, só que sem o nome.

```csharp
interface IDesconto
{
    decimal Aplicar(decimal valor);
}

class CalculadoraDesconto
{
    private readonly IDesconto _estrategia;

    public CalculadoraDesconto(IDesconto estrategia)
    {
        _estrategia = estrategia;
    }

    public decimal Calcular(decimal valor) => _estrategia.Aplicar(valor);
}
```

```csharp
var calculadora = new CalculadoraDesconto(new DescontoVip());
Console.WriteLine(calculadora.Calcular(1000)); // 800
```

👉 `CalculadoraDesconto` não sabe qual algoritmo está usando — ela só executa a **estratégia** que foi injetada. Trocar o comportamento é só trocar qual implementação de `IDesconto` você passa

**A diferença entre Factory e Strategy:** Factory resolve **quem criar**; Strategy resolve **qual comportamento usar**. Muitas vezes eles trabalham juntos — como no exemplo acima, onde a factory decide qual estratégia instanciar.

---

# 🗄️ Repository

👉 **Abstrai o acesso a dados atrás de uma interface, isolando o resto da aplicação dos detalhes de como os dados são armazenados**

```csharp
interface IRepositorio<T>
{
    void Adicionar(T item);
    T ObterPorId(int id);
    IEnumerable<T> ListarTodos();
}
```

```csharp
class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; }
}

class RepositorioEmMemoria<T> : IRepositorio<T> where T : class
{
    private readonly List<T> _itens = new();

    public void Adicionar(T item) => _itens.Add(item);
    public T ObterPorId(int id) => _itens.FirstOrDefault(); // simplificado
    public IEnumerable<T> ListarTodos() => _itens;
}
```

```csharp
IRepositorio<Produto> repositorio = new RepositorioEmMemoria<Produto>();
repositorio.Adicionar(new Produto { Id = 1, Nome = "Notebook" });

foreach (var produto in repositorio.ListarTodos())
{
    Console.WriteLine(produto.Nome);
}
```

👉 Se amanhã você trocar a lista em memória por Entity Framework, Dapper ou uma API externa, quem consome `IRepositorio<T>` **não muda uma linha** — puro Dependency Inversion Principle aplicado a acesso de dados

Esse padrão usa exatamente o que você aprendeu em [Generics](24-generics.md) (`IRepositorio<T>`) e em [DIP](28-solid-principles.md) (depender da interface, não da implementação concreta).

---

# 🔗 Como os quatro se conectam

| Padrão | Problema que resolve | Princípio SOLID relacionado |
|---|---|---|
| **Singleton** | Garantir uma única instância compartilhada | — (usar com cautela) |
| **Factory** | Centralizar e esconder a lógica de criação | Open/Closed |
| **Strategy** | Trocar algoritmos em tempo de execução | Open/Closed, Dependency Inversion |
| **Repository** | Isolar o resto do sistema dos detalhes de persistência | Dependency Inversion |

👉 Repare que nenhum desses padrões é mágica nova — todos são o SOLID aplicado a um problema concreto e recorrente

---

# ⚠️ Erros comuns

- Usar Singleton para tudo, criando estado global escondido e código difícil de testar  
- Criar uma Factory para um único tipo, quando `new` direto já resolveria sem complicação  
- Confundir Strategy com Factory — Strategy troca comportamento, Factory decide qual objeto criar  
- Implementar Repository só para bater ponto de "boas práticas", sem que a aplicação realmente precise trocar a fonte de dados  

---

# 📌 Conclusão

- **Singleton** garante uma única instância — poderoso, mas fácil de abusar  
- **Factory** centraliza e esconde a lógica de criação de objetos  
- **Strategy** permite trocar algoritmos em tempo de execução através de uma interface comum  
- **Repository** isola o acesso a dados, aplicando Dependency Inversion na prática  

👉 Design patterns não são regras — são vocabulário compartilhado. Quando um colega de time disser "isso aqui é um Strategy", agora vocês dois sabem exatamente do que se trata.

---

# 🔥 Próximo passo

Agora que você conhece os padrões mais usados no dia a dia, o próximo nível é:

👉 **Fundamentos do C#: Testes Unitários com xUnit**

Aqui você vai aprender a garantir, de forma automatizada, que todo esse código bem projetado continua funcionando conforme o sistema cresce.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
