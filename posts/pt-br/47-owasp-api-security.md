# 🧠 Fundamentos do C#: Segurança Avançada em APIs (OWASP Top 10 na Prática)

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Autenticação e autorização com JWT  
- Performance e otimização de código  

Você já sabe **quem** pode acessar cada endpoint. Mas autenticação não protege contra tudo — existem vulnerabilidades específicas que atacam APIs mesmo quando a autenticação está funcionando perfeitamente.

👉 **Vamos revisar o OWASP Top 10 aplicado à realidade de uma API em C#**

---

# 💡 O que é o OWASP Top 10?

👉 **Uma lista, mantida pela comunidade OWASP, das vulnerabilidades mais críticas e mais comuns em aplicações web**

Não é uma lista teórica — é baseada em dados reais de incidentes de segurança. Vamos passar pelas que mais afetam APIs .NET no dia a dia.

---

# 💉 1. Injeção (SQL Injection)

```csharp
// ❌ Vulnerável: concatenar entrada do usuário direto na query
var sql = $"SELECT * FROM Produtos WHERE Nome = '{nomeDigitadoPeloUsuario}'";
```

```csharp
// ✅ Seguro: EF Core parametriza automaticamente
var produtos = await _contexto.Produtos
    .Where(p => p.Nome == nomeDigitadoPeloUsuario)
    .ToListAsync();
```

👉 Lembra do EF Core? Usar LINQ em vez de SQL concatenado manualmente **já elimina** a maior parte do risco de SQL Injection — o EF Core sempre parametriza as consultas por baixo dos panos

---

# 🔓 2. Quebra de Autenticação

```csharp
// ❌ Sessão nunca expira, senha sem política mínima
services.AddAuthentication().AddCookie(options =>
{
    options.ExpireTimeSpan = TimeSpan.FromDays(365); // muito tempo
});
```

```csharp
// ✅ Expiração curta + renovação, senhas com política forte
builder.Services.Configure<IdentityOptions>(options =>
{
    options.Password.RequiredLength = 12;
    options.Password.RequireNonAlphanumeric = true;
    options.Lockout.MaxFailedAccessAttempts = 5;
});
```

👉 Lembra do post sobre JWT? Tokens de vida curta, com `ValidateLifetime = true`, reduzem a janela de exposição se um token vazar

---

# 📤 3. Exposição de Dados Sensíveis

```csharp
// ❌ Devolvendo a entidade de domínio inteira, incluindo o hash da senha
return Ok(usuario);
```

```csharp
// ✅ Um DTO explícito controla exatamente o que sai
public record UsuarioResponse(int Id, string Nome, string Email);

return Ok(new UsuarioResponse(usuario.Id, usuario.Nome, usuario.Email));
```

👉 Esse é exatamente o erro comum que você viu no post sobre ASP.NET Core: expor a entidade de domínio direto na resposta. Um `record` de resposta garante que campos sensíveis nunca vazam por acidente

---

# 🌐 4. CORS mal configurado

```csharp
// ❌ Libera qualquer origem, com credenciais
app.UseCors(policy => policy.AllowAnyOrigin().AllowCredentials());
```

```csharp
// ✅ Lista explícita de origens confiáveis
app.UseCors(policy => policy
    .WithOrigins("https://meuapp.com")
    .AllowCredentials()
    .WithMethods("GET", "POST"));
```

👉 `AllowAnyOrigin` combinado com `AllowCredentials` é uma combinação perigosa — na prática, o navegador nem permite essa combinação, mas configurações mal pensadas de CORS continuam sendo uma causa comum de vulnerabilidades

---

# ⚠️ 5. Falta de Validação de Entrada

```csharp
public record CriarProdutoRequest(string Nome, decimal Preco);

public class CriarProdutoValidator : AbstractValidator<CriarProdutoRequest>
{
    public CriarProdutoValidator()
    {
        RuleFor(x => x.Nome).NotEmpty().MaximumLength(100);
        RuleFor(x => x.Preco).GreaterThan(0);
    }
}
```

👉 Usando a biblioteca **FluentValidation**, você garante que dados inválidos ou maliciosos nunca alcançam as regras de negócio da sua entidade — a mesma proteção que você viu no post sobre classes e objetos, agora aplicada na borda da API, antes mesmo dos dados entrarem no domínio

---

# 🔑 6. Controle de Acesso Quebrado

```csharp
// ❌ Verifica só se está autenticado, não se é o dono do recurso
[HttpGet("{pedidoId}")]
[Authorize]
public IActionResult ObterPedido(int pedidoId)
{
    return Ok(_repositorio.ObterPorId(pedidoId)); // qualquer usuário logado vê o pedido de qualquer outro
}
```

```csharp
// ✅ Verifica se o pedido pertence ao usuário autenticado
[HttpGet("{pedidoId}")]
[Authorize]
public IActionResult ObterPedido(int pedidoId)
{
    var pedido = _repositorio.ObterPorId(pedidoId);
    var usuarioId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

    if (pedido.ClienteId.ToString() != usuarioId)
        return Forbid();

    return Ok(pedido);
}
```

👉 `[Authorize]` sozinho só verifica **autenticação**. Verificar se o usuário tem permissão sobre **aquele recurso específico** é uma responsabilidade adicional que o código precisa implementar explicitamente — esse é provavelmente o erro mais comum e mais explorado em APIs reais

---

# 🧾 7. Registro e Monitoramento Insuficientes

```csharp
_logger.LogWarning("Tentativa de acesso não autorizado ao pedido {PedidoId} por {UsuarioId}", pedidoId, usuarioId);
```

👉 Lembra do post sobre Serilog? Logar tentativas de acesso negado, falhas de autenticação e ações sensíveis é o que permite **detectar** um ataque em andamento — sem isso, uma invasão pode passar despercebida por meses

---

# ⚠️ Erros comuns

- Confiar cegamente em dados vindos do cliente, mesmo depois de autenticado  
- Devolver mensagens de erro detalhadas demais (stack trace completo) em produção, revelando detalhes da implementação  
- Esquecer HTTPS em qualquer ambiente, mesmo interno  
- Validar só no front-end, sem repetir a validação no back-end (o front-end é opcional para um atacante)  

---

# 📌 Conclusão

- LINQ/EF Core já protege contra a maior parte do SQL Injection  
- DTOs de resposta evitam vazar dados sensíveis por acidente  
- CORS deve listar origens explícitas, nunca liberar tudo com credenciais  
- Validação de entrada (FluentValidation) barra dados inválidos antes de chegarem ao domínio  
- `[Authorize]` verifica autenticação; verificar posse do recurso é responsabilidade explícita do código  

👉 Segurança não é uma etapa final — é uma camada que atravessa cada decisão que você tomou ao longo de toda essa trilha, desde validação até logging

---

# 🔥 Próximo passo

Agora que sua API está protegida contra as vulnerabilidades mais comuns, o próximo nível é:

👉 **Fundamentos do C#: Versionamento de API**

Aqui você vai aprender a evoluir sua API sem quebrar os clientes que já dependem dela.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
