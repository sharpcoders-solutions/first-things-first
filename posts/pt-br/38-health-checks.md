# 🧠 Fundamentos do C#: Health Checks e Monitoramento

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Logging estruturado com Serilog  
- CI/CD e deploy automatizado  

Seus logs contam a história depois que algo já aconteceu. Mas como um orquestrador (Kubernetes, Azure, um load balancer) sabe, **agora mesmo**, se sua aplicação está saudável o suficiente para receber tráfego?

👉 **É para isso que existem os health checks**

---

# 💡 O que é um Health Check?

👉 **Health check = um endpoint que responde se a aplicação (e suas dependências) estão funcionando**

Sem ele, um orquestrador só sabe que o processo está "rodando" — não sabe se o banco de dados está acessível, se uma API externa está fora do ar, ou se a aplicação está travada silenciosamente.

---

# 🏗️ Configurando o básico

```csharp
// Program.cs
builder.Services.AddHealthChecks();

// ...

app.MapHealthChecks("/health");
```

```bash
curl http://localhost:8080/health
# Healthy
```

👉 Só essa configuração já expõe um endpoint que responde `200 OK` com `Healthy` enquanto a aplicação estiver de pé — simples, mas ainda não verifica nada além do próprio processo estar rodando

---

# 🗄️ Verificando dependências reais

O valor de verdade aparece quando você verifica as dependências externas:

```bash
dotnet add package AspNetCore.HealthChecks.SqlServer
dotnet add package AspNetCore.HealthChecks.Redis
```

```csharp
builder.Services.AddHealthChecks()
    .AddSqlServer(connectionString, name: "banco-de-dados")
    .AddRedis(redisConnectionString, name: "cache")
    .AddUrlGroup(new Uri("https://api-externa.com/status"), name: "api-externa");
```

👉 Agora o health check reflete a saúde real do sistema: se o banco de dados cair, o endpoint `/health` reporta isso automaticamente — sem você escrever nenhuma lógica de verificação manual

---

# 📊 Respostas detalhadas em JSON

```csharp
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (contexto, resultado) =>
    {
        contexto.Response.ContentType = "application/json";

        var resposta = new
        {
            status = resultado.Status.ToString(),
            checks = resultado.Entries.Select(e => new
            {
                nome = e.Key,
                status = e.Value.Status.ToString(),
                descricao = e.Value.Description
            })
        };

        await contexto.Response.WriteAsync(JsonSerializer.Serialize(resposta));
    }
});
```

```json
{
  "status": "Unhealthy",
  "checks": [
    { "nome": "banco-de-dados", "status": "Healthy", "descricao": null },
    { "nome": "cache", "status": "Unhealthy", "descricao": "Timeout ao conectar" }
  ]
}
```

👉 Em vez de um simples "funciona ou não funciona", agora você sabe **exatamente qual** dependência está falhando

---

# 🔀 Liveness vs Readiness: a distinção que ambientes como Kubernetes exigem

## 🔹 Liveness — "a aplicação está viva?"

```csharp
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false // não verifica dependências externas
});
```

👉 Se falhar, o orquestrador **reinicia** o container — usado para detectar travamentos internos

## 🔹 Readiness — "a aplicação está pronta para receber tráfego?"

```csharp
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});
```

👉 Se falhar, o orquestrador **para de enviar tráfego** para esse container, sem necessariamente reiniciá-lo — usado quando uma dependência externa (banco, cache) está temporariamente indisponível

**Regra prática:** liveness verifica "o processo trava?"; readiness verifica "estou pronto para atender pedidos agora?". Confundir os dois pode fazer o orquestrador reiniciar containers desnecessariamente durante uma instabilidade momentânea de uma dependência.

---

# ⚠️ Erros comuns

- Fazer o liveness check verificar dependências externas, causando reinícios em cascata quando o banco de dados fica lento  
- Não configurar health checks e descobrir problemas de infraestrutura só quando o usuário reclama  
- Expor o endpoint `/health` publicamente com detalhes sensíveis sobre a infraestrutura interna  
- Ignorar o resultado do health check no pipeline de CI/CD, publicando uma versão que já nasce com uma dependência quebrada  

---

# 📌 Conclusão

- Health checks respondem, em tempo real, se a aplicação e suas dependências estão saudáveis  
- Verificar dependências reais (banco, cache, APIs externas) é o que dá valor de verdade ao endpoint  
- Liveness detecta travamentos internos; Readiness detecta se a aplicação pode receber tráfego agora  
- Respostas detalhadas em JSON facilitam diagnosticar exatamente o que está falhando  

👉 Com health checks, sua infraestrutura para de adivinhar se a aplicação está bem e passa a **saber**

---

# 🔥 Próximo passo

Agora que sua aplicação relata sua própria saúde, o próximo nível é:

👉 **Fundamentos do C#: Cache em C# (In-Memory e Distribuído com Redis)**

Aqui você vai aprender a reduzir carga no banco de dados e acelerar respostas guardando dados frequentemente acessados.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
