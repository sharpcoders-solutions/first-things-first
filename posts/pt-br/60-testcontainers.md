# 🧠 Fundamentos do C#: Testcontainers

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Testes de integração com banco in-memory  
- Docker para empacotar e rodar aplicações em containers  

O banco in-memory do post anterior é rápido, mas mente: ele não valida constraints reais de SQL Server, não reproduz comportamentos específicos do provider, e algumas queries LINQ que funcionam no in-memory falham no banco de produção. Testcontainers resolve isso.

👉 **Vamos aprender Testcontainers**

---

# 💡 O problema do banco in-memory

```csharp
// Isso "passa" no in-memory, mas pode se comportar diferente no SQL Server real
var pedidos = await _contexto.Pedidos
    .Where(p => EF.Functions.DateDiffDay(p.CriadoEm, DateTime.Now) > 30)
    .ToListAsync();
```

👉 `EF.Functions.DateDiffDay` é traduzido para SQL real no SQL Server, mas o provider in-memory não entende a mesma tradução — o teste pode passar e a produção quebrar

---

# 🏗️ Configurando o Testcontainers

```bash
dotnet add package Testcontainers.MsSql
```

```csharp
public class FabricaTestesComBanco : WebApplicationFactory<Program>, IAsyncLifetime
{
    private readonly MsSqlContainer _container = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .Build();

    public async Task InitializeAsync() => await _container.StartAsync();

    public new async Task DisposeAsync() => await _container.DisposeAsync();

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(servicos =>
        {
            var descritor = servicos.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));

            if (descritor != null)
                servicos.Remove(descritor);

            servicos.AddDbContext<AppDbContext>(opcoes =>
                opcoes.UseSqlServer(_container.GetConnectionString()));
        });
    }
}
```

👉 Lembra do post sobre Docker? O Testcontainers usa a mesma engine do Docker por baixo dos panos, mas gerencia o ciclo de vida do container automaticamente — sobe antes dos testes, derruba depois

---

# 🎯 Usando nos testes

```csharp
public class PedidosComBancoReal : IClassFixture<FabricaTestesComBanco>
{
    private readonly HttpClient _cliente;

    public PedidosComBancoReal(FabricaTestesComBanco fabrica)
    {
        _cliente = fabrica.CreateClient();
    }

    [Fact]
    public async Task BuscarPedidosAntigos_DeveUsarQueryReal()
    {
        var resposta = await _cliente.GetAsync("/pedidos/antigos");

        Assert.Equal(HttpStatusCode.OK, resposta.StatusCode);
        // a query DateDiffDay realmente executa contra o SQL Server aqui
    }
}
```

👉 O teste roda exatamente contra o mesmo motor de banco de produção — sem gap de comportamento entre "passou no teste" e "quebrou em produção"

---

# 🐰 Além de banco: qualquer dependência containerizável

```csharp
private readonly RabbitMqContainer _rabbit = new RabbitMqBuilder()
    .WithImage("rabbitmq:3-management")
    .Build();

private readonly RedisContainer _redis = new RedisBuilder()
    .WithImage("redis:7")
    .Build();
```

👉 Lembra do RabbitMQ (post 41) e do cache com Redis (post 39)? O Testcontainers cobre qualquer dependência que rode em Docker — banco, fila, cache, tudo testável contra a implementação real, não um substituto simplificado

---

# ⚠️ Erros comuns

- Compartilhar o mesmo container entre testes sem limpar o estado, causando testes que dependem da ordem de execução  
- Não definir timeout para o container subir, travando o pipeline de CI se o Docker demorar  
- Usar Testcontainers para tudo, mesmo quando in-memory já é suficiente para uma regra de negócio simples — mais lento sem necessidade real  
- Esquecer que o pipeline de CI precisa ter Docker disponível para rodar esses testes  

---

# 📌 Conclusão

- Bancos in-memory podem mentir sobre comportamentos específicos do provider real  
- Testcontainers sobe uma instância real (SQL Server, RabbitMQ, Redis) em container, só para os testes  
- O ciclo de vida do container é gerenciado automaticamente pelo próprio teste  
- O ganho é fidelidade: o teste valida contra o mesmo motor que roda em produção  

👉 Com Testcontainers, seus testes de integração param de confiar em substitutos e passam a validar contra a coisa real

---

# 🔥 Próximo passo

Agora que seus testes validam contra dependências reais, o próximo nível é:

👉 **Fundamentos do C#: Mutation Testing**

Aqui você vai aprender a testar se seus próprios testes realmente pegam bugs, não só se eles passam.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
