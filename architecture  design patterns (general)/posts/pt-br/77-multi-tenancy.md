# 🧠 Fundamentos do C#: Multi-tenancy

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Options Pattern para configuração fortemente tipada  
- Entity Framework Core e autenticação JWT  

Imagine um SaaS onde cada empresa cliente precisa ver só os próprios dados, mesma aplicação, mesmo deploy. Isso é multi-tenancy — um problema que aparece assim que seu produto passa de "uma empresa usa" para "várias empresas usam a mesma instância".

👉 **Vamos aprender Multi-tenancy**

---

# 💡 O que é um tenant?

👉 **Tenant = um cliente isolado logicamente dentro da mesma aplicação**

Se você constrói um sistema de gestão de pedidos vendido para várias lojas, cada loja é um tenant — a loja A nunca deveria ver pedidos da loja B, mesmo compartilhando a mesma aplicação e (possivelmente) o mesmo banco

---

# 🏗️ Estratégias de isolamento de dados

## 🔹 Banco de dados por tenant (isolamento total)

```csharp
public string ObterConnectionString(string tenantId) =>
    $"Server=meuservidor;Database=Empresa_{tenantId};...";
```

👉 Isolamento máximo — cada tenant tem seu próprio banco. Mais seguro, mas mais caro operacionalmente: migrações (post 32) precisam rodar em N bancos

## 🔹 Schema por tenant

```sql
CREATE SCHEMA Empresa_123;
CREATE TABLE Empresa_123.Pedidos (...);
```

👉 Meio-termo: mesmo servidor de banco, schemas separados por tenant

## 🔹 Coluna discriminadora (shared database)

```csharp
public class Pedido
{
    public int Id { get; set; }
    public Guid TenantId { get; set; } // toda tabela carrega essa coluna
    public decimal Valor { get; set; }
}
```

👉 Mais barato operacionalmente, mas exige disciplina total: **toda** query precisa filtrar por `TenantId`, sem exceção

---

# 🎯 Aplicando o filtro automaticamente com EF Core

```csharp
public class AppDbContext : DbContext
{
    private readonly string _tenantIdAtual;

    public AppDbContext(DbContextOptions options, ITenantProvider tenantProvider)
        : base(options)
    {
        _tenantIdAtual = tenantProvider.ObterTenantId();
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.Entity<Pedido>().HasQueryFilter(p => p.TenantId == _tenantIdAtual);
    }
}
```

👉 `HasQueryFilter` aplica o filtro de tenant **automaticamente** em toda consulta LINQ (post 19) — ninguém no time precisa lembrar de escrever `.Where(p => p.TenantId == tenantAtual)` manualmente em cada query, eliminando a chance de vazamento de dados entre tenants por esquecimento

---

# 🔍 Identificando o tenant na requisição

```csharp
public class TenantMiddleware
{
    private readonly RequestDelegate _proximo;

    public async Task InvokeAsync(HttpContext contexto, ITenantProvider tenantProvider)
    {
        var subdominio = contexto.Request.Host.Host.Split('.')[0]; // ex: empresa123.meuapp.com

        tenantProvider.DefinirTenantId(subdominio);

        await _proximo(contexto);
    }
}
```

👉 Lembra do middleware pipeline do post sobre ASP.NET Core? O tenant costuma ser identificado por subdomínio, header customizado, ou claim no JWT (post 34) — resolvido uma vez no início da requisição, disponível para todo o resto do pipeline

---

# ⚠️ Erros comuns

- Esquecer o `HasQueryFilter` em uma entidade nova, criando um vazamento silencioso de dados entre tenants  
- Misturar lógica de identificação de tenant espalhada pelo código, em vez de centralizá-la em um middleware  
- Escolher isolamento total (banco por tenant) sem necessidade real, aumentando complexidade operacional desnecessariamente  
- Não testar especificamente cenários de isolamento entre tenants (lembra dos testes de integração do post 59?) — esse é o tipo de bug que só aparece em produção, com dados reais de dois clientes diferentes  

---

# 📌 Conclusão

- Multi-tenancy isola clientes logicamente dentro da mesma aplicação  
- As estratégias variam de isolamento total (banco por tenant) a compartilhado (coluna discriminadora)  
- `HasQueryFilter` do EF Core aplica o filtro de tenant automaticamente, reduzindo risco de vazamento  
- O tenant é identificado uma vez, cedo no pipeline, e propagado para o resto da requisição  

👉 Com multi-tenancy bem implementado, uma única aplicação atende múltiplos clientes com a mesma confiança de isolamento que instâncias separadas dariam

---

# 🔥 Próximo passo

Agora que você sabe isolar dados entre clientes, o próximo nível é:

👉 **Fundamentos do C#: Kafka**

Aqui você vai aprender mensageria orientada a streams, uma alternativa ao RabbitMQ para cenários de altíssimo volume.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
