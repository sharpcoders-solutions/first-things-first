# 🧠 Fundamentos do C#: Testes de Integração com WebApplicationFactory

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Testes unitários com xUnit, isolando uma classe por vez  
- Recursos avançados de tipos: iteradores, operadores customizados, indexers e static abstract members  

Testes unitários validam uma classe isolada. Mas será que o roteamento HTTP, a autenticação JWT, a injeção de dependência e o banco de dados realmente funcionam **juntos**? É isso que testes de integração respondem.

👉 **Vamos aprender testes de integração com WebApplicationFactory**

---

# 💡 Unitário vs Integração

## 🔹 Teste unitário (post 33)

```csharp
[Fact]
public void CalcularDesconto_DeveAplicar10PorCento()
{
    var resultado = _calculadora.Calcular(100m);
    Assert.Equal(90m, resultado);
}
```

👉 Isola uma classe, usando mocks (Moq) para tudo à volta

## 🔹 Teste de integração

```csharp
[Fact]
public async Task PostPedido_DeveRetornar201()
{
    var resposta = await _cliente.PostAsJsonAsync("/pedidos", novoPedido);
    Assert.Equal(HttpStatusCode.Created, resposta.StatusCode);
}
```

👉 Sobe a aplicação inteira, real, e testa via HTTP de verdade — roteamento, model binding, DI, tudo funcionando junto, como você aprendeu no post sobre ASP.NET Core

---

# 🏗️ Configurando o WebApplicationFactory

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

```csharp
public class FabricaTestes : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(servicos =>
        {
            var descritor = servicos.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));

            if (descritor != null)
                servicos.Remove(descritor);

            servicos.AddDbContext<AppDbContext>(opcoes =>
                opcoes.UseInMemoryDatabase("BancoDeTestes"));
        });
    }
}
```

👉 `Program` é a classe do seu `Program.cs` (lembra do post 14, com top-level statements?) — o `WebApplicationFactory` sobe a aplicação real inteira em memória, trocando só o banco por um in-memory

---

# 🎯 Escrevendo o teste

```csharp
public class PedidosControllerTestes : IClassFixture<FabricaTestes>
{
    private readonly HttpClient _cliente;

    public PedidosControllerTestes(FabricaTestes fabrica)
    {
        _cliente = fabrica.CreateClient();
    }

    [Fact]
    public async Task PostPedido_ComDadosValidos_DeveRetornar201()
    {
        var novoPedido = new { ClienteId = 1, Valor = 150.00m };

        var resposta = await _cliente.PostAsJsonAsync("/pedidos", novoPedido);

        Assert.Equal(HttpStatusCode.Created, resposta.StatusCode);

        var pedidoCriado = await resposta.Content.ReadFromJsonAsync<PedidoDto>();
        Assert.NotNull(pedidoCriado);
        Assert.Equal(150.00m, pedidoCriado!.Valor);
    }

    [Fact]
    public async Task GetPedido_ComIdInexistente_DeveRetornar404()
    {
        var resposta = await _cliente.GetAsync("/pedidos/99999");

        Assert.Equal(HttpStatusCode.NotFound, resposta.StatusCode);
    }
}
```

👉 `IClassFixture<FabricaTestes>` compartilha a mesma instância da aplicação entre todos os testes da classe, evitando o custo de subir a aplicação a cada teste — cada `[Fact]` continua isolado (post 33), mas a infraestrutura é reutilizada

---

# 🔐 Testando endpoints autenticados

```csharp
[Fact]
public async Task GetPedidos_SemToken_DeveRetornar401()
{
    var resposta = await _cliente.GetAsync("/pedidos");
    Assert.Equal(HttpStatusCode.Unauthorized, resposta.StatusCode);
}

[Fact]
public async Task GetPedidos_ComTokenValido_DeveRetornar200()
{
    var token = GerarTokenDeTeste();
    _cliente.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

    var resposta = await _cliente.GetAsync("/pedidos");

    Assert.Equal(HttpStatusCode.OK, resposta.StatusCode);
}
```

👉 Combinado com o post sobre JWT, você valida não só que a lógica de negócio funciona, mas que o `[Authorize]` realmente bloqueia requisições sem token válido — o mesmo 401 vs 403 que discutimos naquele post

---

# ⚠️ Erros comuns

- Usar o banco de produção (ou até de desenvolvimento) nos testes de integração, contaminando dados reais  
- Não isolar o estado entre testes, fazendo um teste passar ou falhar dependendo da ordem de execução  
- Escrever só testes de integração, perdendo a velocidade e a granularidade dos testes unitários  
- Testar todos os cenários de borda via integração, quando validação de regras específicas deveria estar em testes unitários mais rápidos  

---

# 📌 Conclusão

- Testes de integração validam que roteamento, DI, autenticação e banco funcionam juntos  
- `WebApplicationFactory` sobe a aplicação real inteira, em memória, para os testes  
- Trocar o banco real por um in-memory database isola os testes sem sacrificar realismo  
- Testes unitários e de integração são complementares, não substitutos um do outro  

👉 Com WebApplicationFactory, você testa sua API como ela realmente vai ser usada: via HTTP, de ponta a ponta

---

# 🔥 Próximo passo

Agora que você testa sua API inteira em memória, o próximo nível é:

👉 **Fundamentos do C#: Coleções Concorrentes (ConcurrentDictionary e Cia.)**

Aqui você vai aprender a lidar com múltiplas threads acessando a mesma coleção ao mesmo tempo, sem corromper dados nem travar sua aplicação.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
