# 🧠 Fundamentos do C#: Clean Architecture na Prática

⏱️ Tempo de leitura: 9 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- SOLID e design patterns  
- Como construir uma API com ASP.NET Core  
- Como persistir dados de verdade com EF Core  

Você já tem todas as peças. Este post é sobre como **organizá-las** de um jeito que sobrevive ao crescimento do projeto — o tipo de estrutura que você vai encontrar em praticamente todo projeto sério de C# no mercado.

👉 **Vamos juntar tudo em Clean Architecture**

---

# 💡 O que é Clean Architecture?

👉 **Clean Architecture = uma forma de organizar o código em camadas, onde as regras de negócio nunca dependem de detalhes técnicos como banco de dados ou framework web**

A ideia foi popularizada por Robert C. Martin — o mesmo "Uncle Bob" do post sobre SOLID. Não é coincidência: Clean Architecture é, em grande parte, o **Dependency Inversion Principle** aplicado à estrutura inteira do projeto, não só a uma classe.

## 🔹 A Regra da Dependência

👉 **Dependências sempre apontam para dentro, em direção às regras de negócio — nunca o contrário**

```
API  →  Application  →  Domain
             ↑
      Infrastructure
```

O `Domain` (o centro) não sabe que existe um banco de dados, uma API ou um framework. Tudo o que é **técnico** (EF Core, ASP.NET Core, provedores de e-mail) fica nas bordas, e depende do centro — nunca o contrário.

---

# 🧱 As quatro camadas

## 🔹 1. Domain — o coração da aplicação

```csharp
// MinhaApi.Domain
public class Produto
{
    public int Id { get; set; }
    public string Nome { get; private set; }
    public decimal Preco { get; private set; }

    public Produto(string nome, decimal preco)
    {
        if (preco <= 0)
            throw new ArgumentException("Preço deve ser maior que zero");

        Nome = nome;
        Preco = preco;
    }

    public void AplicarDesconto(decimal percentual)
    {
        Preco -= Preco * percentual;
    }
}
```

👉 O `Domain` contém as entidades e as regras de negócio (você reconhece isso do post sobre classes e objetos: encapsulamento protegendo o estado). **Nenhuma referência a EF Core, ASP.NET ou qualquer coisa externa**

## 🔹 2. Application — os casos de uso

```csharp
// MinhaApi.Application
public interface IProdutoRepositorio
{
    void Adicionar(Produto produto);
    Produto ObterPorId(int id);
    IEnumerable<Produto> ListarTodos();
}

public class CriarProdutoUseCase
{
    private readonly IProdutoRepositorio _repositorio;

    public CriarProdutoUseCase(IProdutoRepositorio repositorio)
    {
        _repositorio = repositorio;
    }

    public Produto Executar(string nome, decimal preco)
    {
        var produto = new Produto(nome, preco);
        _repositorio.Adicionar(produto);
        return produto;
    }
}
```

👉 O `Application` orquestra os casos de uso do sistema ("criar produto", "aplicar desconto") e define **interfaces** para tudo que é externo — como `IProdutoRepositorio`. Repare: a interface está aqui, não na infraestrutura. Isso é a Regra da Dependência na prática

## 🔹 3. Infrastructure — os detalhes técnicos

```csharp
// MinhaApi.Infrastructure
public class ProdutoRepositorioEfCore : IProdutoRepositorio
{
    private readonly AppDbContext _contexto;

    public ProdutoRepositorioEfCore(AppDbContext contexto)
    {
        _contexto = contexto;
    }

    public void Adicionar(Produto produto) => _contexto.Produtos.Add(produto);
    public Produto ObterPorId(int id) => _contexto.Produtos.Find(id);
    public IEnumerable<Produto> ListarTodos() => _contexto.Produtos.ToList();
}
```

👉 `Infrastructure` implementa as interfaces definidas no `Application`, usando EF Core, APIs externas, sistemas de arquivo — tudo que é "detalhe". Esta camada **depende** do `Application`, nunca o contrário

## 🔹 4. API (Presentation) — a porta de entrada

```csharp
// MinhaApi.Api
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    private readonly CriarProdutoUseCase _criarProduto;

    public ProdutosController(CriarProdutoUseCase criarProduto)
    {
        _criarProduto = criarProduto;
    }

    [HttpPost]
    public IActionResult Criar(CriarProdutoRequest request)
    {
        var produto = _criarProduto.Executar(request.Nome, request.Preco);
        return CreatedAtAction(nameof(Criar), new { id = produto.Id }, produto);
    }
}
```

👉 O controller não conhece EF Core, nem sabe como o produto é salvo — ele só chama o caso de uso. Comparado ao post sobre ASP.NET Core, onde o controller falava direto com o repositório, agora existe uma camada de orquestração de negócio entre os dois

---

# 🔌 Conectando tudo: injeção de dependência

```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options => options.UseSqlServer(connectionString));

builder.Services.AddScoped<IProdutoRepositorio, ProdutoRepositorioEfCore>();
builder.Services.AddScoped<CriarProdutoUseCase>();
```

👉 O ASP.NET Core monta a cadeia inteira: quando o controller pede um `CriarProdutoUseCase`, o container entrega um já com `IProdutoRepositorio` injetado — que, por sua vez, já vem com o `AppDbContext` pronto. Tudo isso é a injeção de dependência que você aprendeu, só que orquestrando quatro camadas em vez de duas classes soltas

---

# 🗂️ Estrutura de pastas / projetos

```
MinhaApi.sln
├── MinhaApi.Domain          (sem dependências)
├── MinhaApi.Application     (depende de Domain)
├── MinhaApi.Infrastructure  (depende de Application e Domain)
└── MinhaApi.Api             (depende de Application, Infrastructure e Domain)
```

👉 Em projetos reais, cada camada costuma ser um **projeto .NET separado**, e o próprio compilador impede referências na direção errada — se `Domain` tentar referenciar `Infrastructure`, o build falha. A arquitetura vira uma regra imposta pela ferramenta, não só uma convenção de time

---

# 🔗 Como tudo se conecta com o que você já aprendeu

| Camada | Conceitos que você já viu |
|---|---|
| **Domain** | Classes, encapsulamento, construtores validando estado |
| **Application** | Interfaces, Dependency Inversion, casos de uso como Strategy |
| **Infrastructure** | EF Core, implementação concreta do Repository pattern |
| **API** | ASP.NET Core, controllers, model binding, respostas HTTP |

👉 Clean Architecture não introduz conceitos novos — ela é a **organização** de tudo que você já domina, cada peça no lugar certo

---

# ⚠️ Erros comuns

- Aplicar Clean Architecture em um projeto pequeno demais, criando quatro projetos para uma aplicação que caberia em um único arquivo  
- Deixar o `Domain` referenciar EF Core (ex: usando atributos do Entity Framework diretamente nas entidades de domínio)  
- Colocar lógica de negócio no controller "porque é mais rápido", esvaziando o propósito da camada `Application`  
- Achar que Clean Architecture é uma estrutura de pastas fixa — o que importa é a **direção das dependências**, não os nomes exatos das camadas  

---

# 📌 Conclusão

- A Regra da Dependência: tudo aponta para dentro, em direção às regras de negócio  
- `Domain` não depende de nada; `Infrastructure` implementa o que `Application` define  
- Interfaces vivem perto do centro; implementações concretas vivem nas bordas  
- Nenhum conceito aqui é novo — é tudo o que você já aprendeu, organizado com intenção  

👉 Com Clean Architecture, seu projeto para de ser "só uma API que salva no banco" e passa a ser um sistema onde trocar o banco, o framework web ou até a forma de notificar usuários nunca ameaça a regra de negócio central

---

# 🔥 Próximo passo

Agora que você sabe estruturar uma aplicação profissional de ponta a ponta, o próximo nível é:

👉 **Fundamentos do C#: Autenticação e Autorização com JWT**

Aqui você vai aprender a proteger sua API, garantindo que só usuários autenticados (e autorizados) acessem cada endpoint.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
