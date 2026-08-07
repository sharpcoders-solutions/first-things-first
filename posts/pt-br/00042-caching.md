# 🧠 Fundamentos do C#: Cache em C# (In-Memory e Distribuído com Redis)

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Health checks e monitoramento  
- Persistência de dados com EF Core  

Toda consulta ao banco de dados tem um custo. Se a mesma informação é pedida cem vezes por minuto e muda raramente, buscar do banco toda vez é desperdício puro.

👉 **É para isso que existe cache**

---

# 💡 O que é cache?

👉 **Cache = guardar temporariamente um resultado já calculado, para não precisar refazer o trabalho**

```csharp
// Sem cache: toda requisição bate no banco
var produtos = await _contexto.Produtos.ToListAsync();

// Com cache: só bate no banco na primeira vez
```

O ganho é direto: menos carga no banco, respostas mais rápidas. O custo: dados em cache podem ficar **desatualizados** por um tempo — cache é sempre uma troca entre performance e atualidade.

---

# 🏗️ `IMemoryCache`: cache em memória

```csharp
builder.Services.AddMemoryCache();
```

```csharp
public class ProdutosController : ControllerBase
{
    private readonly IMemoryCache _cache;
    private readonly IProdutoRepositorio _repositorio;

    public ProdutosController(IMemoryCache cache, IProdutoRepositorio repositorio)
    {
        _cache = cache;
        _repositorio = repositorio;
    }

    [HttpGet]
    public IActionResult ObterTodos()
    {
        var produtos = _cache.GetOrCreate("produtos-todos", entrada =>
        {
            entrada.SetAbsoluteExpiration(TimeSpan.FromMinutes(5));
            return _repositorio.ListarTodos();
        });

        return Ok(produtos);
    }
}
```

👉 `GetOrCreate` verifica se a chave já existe no cache; se não existir, executa a função e guarda o resultado pelo tempo definido. Nas próximas chamadas, dentro dos 5 minutos, o repositório **nem é consultado**

## 🔹 Quando `IMemoryCache` é suficiente

👉 Ótimo para aplicações com uma única instância — o cache vive na memória do processo, então cada instância teria seu próprio cache separado

---

# 🌐 Cache distribuído: o problema de múltiplas instâncias

Quando sua API roda em várias instâncias (múltiplos containers ou máquinas atrás de um load balancer), `IMemoryCache` vira um problema: cada instância tem seu próprio cache, isolado das outras.

```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
});
```

```csharp
public class ProdutosController : ControllerBase
{
    private readonly IDistributedCache _cache;

    public ProdutosController(IDistributedCache cache)
    {
        _cache = cache;
    }

    [HttpGet]
    public async Task<IActionResult> ObterTodos()
    {
        var cacheado = await _cache.GetStringAsync("produtos-todos");

        if (cacheado is not null)
        {
            return Ok(JsonSerializer.Deserialize<List<Produto>>(cacheado));
        }

        var produtos = await _repositorio.ListarTodosAsync();

        await _cache.SetStringAsync(
            "produtos-todos",
            JsonSerializer.Serialize(produtos),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            });

        return Ok(produtos);
    }
}
```

👉 `IDistributedCache` guarda os dados em um serviço externo compartilhado (Redis) — todas as instâncias da sua API leem e escrevem no **mesmo** cache, resolvendo o problema da memória isolada

---

# ⏳ Estratégias de expiração

```csharp
// Expira em um horário fixo, não importa o uso
new DistributedCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
};

// Expira só se ficar 2 minutos sem ser acessado
new DistributedCacheEntryOptions
{
    SlidingExpiration = TimeSpan.FromMinutes(2)
};
```

## 🔹 Qual escolher

- **Absoluta** → dados que mudam em intervalos previsíveis (ex: cotação atualizada de hora em hora)  
- **Deslizante (sliding)** → dados acessados com frequência, mas sem horário fixo de mudança  

👉 Combinar as duas é comum: expiração absoluta como limite máximo, deslizante para renovar enquanto o dado continua sendo usado

---

# ♻️ Invalidando o cache

```csharp
[HttpPost]
public async Task<IActionResult> Criar(CriarProdutoRequest request)
{
    var produto = new Produto(request.Nome, request.Preco);
    await _repositorio.AdicionarAsync(produto);

    await _cache.RemoveAsync("produtos-todos"); // invalida o cache desatualizado

    return CreatedAtAction(nameof(ObterTodos), produto);
}
```

👉 Sempre que os dados mudam, o cache correspondente precisa ser invalidado — esquecer isso é a causa mais comum de bugs relacionados a cache ("por que os dados não atualizam?")

---

# ⚠️ Erros comuns

- Usar `IMemoryCache` em uma aplicação com múltiplas instâncias, gerando dados inconsistentes entre elas  
- Cachear dados sensíveis ou específicos de usuário em uma chave compartilhada por engano  
- Esquecer de invalidar o cache depois de uma escrita, servindo dados desatualizados indefinidamente  
- Cachear tudo, mesmo dados que mudam a cada segundo, sem ganho real de performance  

---

# 📌 Conclusão

- Cache troca atualidade por performance, guardando resultados já calculados  
- `IMemoryCache` funciona bem para uma única instância; `IDistributedCache` (Redis) resolve múltiplas instâncias  
- Expiração absoluta e deslizante controlam por quanto tempo um dado fica em cache  
- Invalidar o cache após uma escrita é essencial para evitar dados desatualizados  

👉 Com cache bem aplicado, sua aplicação responde mais rápido e reduz a carga no banco de dados, sem comprometer a confiabilidade dos dados

---

# 🔥 Próximo passo

Agora que você sabe acelerar sua aplicação com cache, o próximo nível é:

👉 **Fundamentos do C#: Resiliência com Polly**

Aqui você vai aprender a fazer sua aplicação lidar graciosamente com falhas temporárias em dependências externas.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
