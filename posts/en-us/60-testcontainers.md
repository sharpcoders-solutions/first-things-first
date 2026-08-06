# 🧠 C# Fundamentals: Testcontainers

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Integration testing with an in-memory database  
- Docker for packaging and running applications in containers  

The in-memory database from the previous post is fast, but it lies: it doesn't validate real SQL Server constraints, doesn't reproduce provider-specific behaviors, and some LINQ queries that work in-memory fail against the production database. Testcontainers fixes this.

👉 **Let's learn Testcontainers**

---

# 💡 The problem with in-memory databases

```csharp
// This "passes" in-memory, but may behave differently on real SQL Server
var orders = await _context.Orders
    .Where(o => EF.Functions.DateDiffDay(o.CreatedAt, DateTime.Now) > 30)
    .ToListAsync();
```

👉 `EF.Functions.DateDiffDay` translates to real SQL on SQL Server, but the in-memory provider doesn't understand the same translation — the test can pass while production breaks

---

# 🏗️ Setting up Testcontainers

```bash
dotnet add package Testcontainers.MsSql
```

```csharp
public class DatabaseTestFactory : WebApplicationFactory<Program>, IAsyncLifetime
{
    private readonly MsSqlContainer _container = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .Build();

    public async Task InitializeAsync() => await _container.StartAsync();

    public new async Task DisposeAsync() => await _container.DisposeAsync();

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));

            if (descriptor != null)
                services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
                options.UseSqlServer(_container.GetConnectionString()));
        });
    }
}
```

👉 Remember the Docker post? Testcontainers uses the same Docker engine under the hood, but manages the container's lifecycle automatically — spins up before the tests, tears down after

---

# 🎯 Using it in tests

```csharp
public class OrdersWithRealDatabaseTests : IClassFixture<DatabaseTestFactory>
{
    private readonly HttpClient _client;

    public OrdersWithRealDatabaseTests(DatabaseTestFactory factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetOldOrders_ShouldUseRealQuery()
    {
        var response = await _client.GetAsync("/orders/old");

        Assert.Equal(HttpStatusCode.OK, response.StatusCode);
        // the DateDiffDay query actually runs against SQL Server here
    }
}
```

👉 The test runs against the exact same database engine as production — no behavior gap between "passed the test" and "broke in production"

---

# 🐰 Beyond databases: any containerizable dependency

```csharp
private readonly RabbitMqContainer _rabbit = new RabbitMqBuilder()
    .WithImage("rabbitmq:3-management")
    .Build();

private readonly RedisContainer _redis = new RedisBuilder()
    .WithImage("redis:7")
    .Build();
```

👉 Remember RabbitMQ (post 41) and Redis caching (post 39)? Testcontainers covers any dependency that runs in Docker — database, queue, cache, all testable against the real implementation, not a simplified stand-in

---

# ⚠️ Common Mistakes

- Sharing the same container across tests without cleaning up state, causing tests that depend on execution order  
- Not setting a timeout for the container to start, hanging the CI pipeline if Docker takes too long  
- Using Testcontainers for everything, even when in-memory is already enough for a simple business rule — slower with no real benefit  
- Forgetting that the CI pipeline needs Docker available to run these tests  

---

# 📌 Conclusion

- In-memory databases can lie about provider-specific behaviors  
- Testcontainers spins up a real instance (SQL Server, RabbitMQ, Redis) in a container, just for the tests  
- The container's lifecycle is managed automatically by the test itself  
- The payoff is fidelity: the test validates against the same engine that runs in production  

👉 With Testcontainers, your integration tests stop trusting stand-ins and start validating against the real thing

---

# 🔥 Next Step

Now that your tests validate against real dependencies, the next level is:

👉 **C# Fundamentals: Mutation Testing**

Here you'll learn to test whether your own tests actually catch bugs, not just whether they pass.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
