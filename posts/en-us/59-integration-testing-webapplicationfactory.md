# 🧠 C# Fundamentals: Integration Testing with WebApplicationFactory

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Unit testing with xUnit, isolating one class at a time  
- Advanced type features: iterators, custom operators, indexers, and static abstract members  

Unit tests validate an isolated class. But do HTTP routing, JWT authentication, dependency injection, and the database actually work **together**? That's what integration tests answer.

👉 **Let's learn integration testing with WebApplicationFactory**

---

# 💡 Unit vs Integration

## 🔹 Unit test (post 30)

```csharp
[Fact]
public void CalculateDiscount_ShouldApply10Percent()
{
    var result = _calculator.Calculate(100m);
    Assert.Equal(90m, result);
}
```

👉 Isolates a single class, using mocks (Moq) for everything around it

## 🔹 Integration test

```csharp
[Fact]
public async Task PostOrder_ShouldReturn201()
{
    var response = await _client.PostAsJsonAsync("/orders", newOrder);
    Assert.Equal(HttpStatusCode.Created, response.StatusCode);
}
```

👉 Spins up the whole real application and tests it through real HTTP — routing, model binding, DI, everything working together, like you learned in the ASP.NET Core post

---

# 🏗️ Setting up WebApplicationFactory

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

```csharp
public class TestFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));

            if (descriptor != null)
                services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
                options.UseInMemoryDatabase("TestDatabase"));
        });
    }
}
```

👉 `Program` is the class from your `Program.cs` (remember post 14, with top-level statements?) — `WebApplicationFactory` spins up the entire real application in memory, swapping only the database for an in-memory one

---

# 🎯 Writing the test

```csharp
public class OrdersControllerTests : IClassFixture<TestFactory>
{
    private readonly HttpClient _client;

    public OrdersControllerTests(TestFactory factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task PostOrder_WithValidData_ShouldReturn201()
    {
        var newOrder = new { CustomerId = 1, Amount = 150.00m };

        var response = await _client.PostAsJsonAsync("/orders", newOrder);

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);

        var createdOrder = await response.Content.ReadFromJsonAsync<OrderDto>();
        Assert.NotNull(createdOrder);
        Assert.Equal(150.00m, createdOrder!.Amount);
    }

    [Fact]
    public async Task GetOrder_WithNonexistentId_ShouldReturn404()
    {
        var response = await _client.GetAsync("/orders/99999");

        Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
    }
}
```

👉 `IClassFixture<TestFactory>` shares the same application instance across all tests in the class, avoiding the cost of spinning up the application for every test — each `[Fact]` remains isolated (post 30), but the infrastructure is reused

---

# 🔐 Testing authenticated endpoints

```csharp
[Fact]
public async Task GetOrders_WithoutToken_ShouldReturn401()
{
    var response = await _client.GetAsync("/orders");
    Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
}

[Fact]
public async Task GetOrders_WithValidToken_ShouldReturn200()
{
    var token = GenerateTestToken();
    _client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

    var response = await _client.GetAsync("/orders");

    Assert.Equal(HttpStatusCode.OK, response.StatusCode);
}
```

👉 Combined with the JWT post, you validate not just that the business logic works, but that `[Authorize]` actually blocks requests without a valid token — the same 401 vs 403 distinction we discussed in that post

---

# ⚠️ Common Mistakes

- Using the production (or even development) database in integration tests, contaminating real data  
- Not isolating state between tests, making a test pass or fail depending on execution order  
- Writing only integration tests, losing the speed and granularity of unit tests  
- Testing every edge case through integration, when specific rule validation should live in faster unit tests  

---

# 📌 Conclusion

- Integration tests validate that routing, DI, authentication, and the database work together  
- `WebApplicationFactory` spins up the entire real application, in memory, for tests  
- Swapping the real database for an in-memory one isolates tests without sacrificing realism  
- Unit and integration tests are complementary, not substitutes for each other  

👉 With WebApplicationFactory, you test your API the way it's actually going to be used: over HTTP, end to end

---

# 🔥 Next Step

Now that you can test your entire API in memory, the next level is:

👉 **C# Fundamentals: Concurrent Collections (ConcurrentDictionary and Friends)**

Here you'll learn to handle multiple threads accessing the same collection at once, without corrupting data or locking up your application.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
