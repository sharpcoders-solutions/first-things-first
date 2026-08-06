# 🧠 Fundamentos do C#: Rate Limiting em ASP.NET Core

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Background jobs para tarefas fora do ciclo da requisição  
- Segurança seguindo o OWASP Top 10  

Sua API está protegida contra vulnerabilidades clássicas, mas ainda existe um problema: nada impede que um único cliente (por erro ou má-fé) faça milhares de requisições por segundo e derrube o serviço para todo mundo.

👉 **Vamos aprender a limitar taxa de requisições com Rate Limiting**

---

# 💡 O que é Rate Limiting?

👉 **Rate Limiting = restringir quantas requisições um cliente pode fazer em um intervalo de tempo**

Sem isso, sua API fica exposta a:

- Bugs em clientes que geram loops de requisições sem querer  
- Ataques de força bruta contra login (lembra do post sobre JWT?)  
- Scraping agressivo consumindo todos os seus recursos de servidor  

---

# 🏗️ Configurando Rate Limiting nativo do ASP.NET Core

```csharp
// Program.cs
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("padrao", opcoes =>
    {
        opcoes.PermitLimit = 100;
        opcoes.Window = TimeSpan.FromMinutes(1);
        opcoes.QueueLimit = 0;
    });

    options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
});

// ...

app.UseRateLimiter();
```

```csharp
[HttpGet]
[EnableRateLimiting("padrao")]
public IActionResult ObterTodos() => Ok(/* ... */);
```

👉 `429 Too Many Requests` é o código HTTP correto para sinalizar esse limite — lembra do post sobre respostas HTTP corretas? Aqui é mais um código que faz parte do contrato da sua API

---

# 🪟 Estratégias de limitação

## 🔹 Fixed Window (janela fixa)

```csharp
options.AddFixedWindowLimiter("padrao", opcoes =>
{
    opcoes.PermitLimit = 100;
    opcoes.Window = TimeSpan.FromMinutes(1);
});
```

👉 100 requisições por minuto, resetando no início de cada janela — simples, mas pode permitir picos nas bordas da janela (100 no fim de um minuto + 100 no início do próximo)

## 🔹 Sliding Window (janela deslizante)

```csharp
options.AddSlidingWindowLimiter("padrao", opcoes =>
{
    opcoes.PermitLimit = 100;
    opcoes.Window = TimeSpan.FromMinutes(1);
    opcoes.SegmentsPerWindow = 4;
});
```

👉 Divide a janela em segmentos, suavizando os picos que o Fixed Window permite

## 🔹 Token Bucket

```csharp
options.AddTokenBucketLimiter("padrao", opcoes =>
{
    opcoes.TokenLimit = 100;
    opcoes.TokensPerPeriod = 20;
    opcoes.ReplenishmentPeriod = TimeSpan.FromSeconds(10);
});
```

👉 Permite rajadas controladas (usando tokens acumulados), reabastecendo gradualmente — ideal para tráfego naturalmente "em rajadas", como usuários abrindo várias abas ao mesmo tempo

## 🔹 Concurrency Limiter

```csharp
options.AddConcurrencyLimiter("uploads", opcoes =>
{
    opcoes.PermitLimit = 5;
});
```

👉 Não limita por tempo, mas por **quantidade simultânea** — ótimo para endpoints custosos, como upload de arquivos grandes

---

# 🎯 Limitando por usuário, não só globalmente

```csharp
options.AddPolicy("por-usuario", contexto =>
{
    var usuarioId = contexto.User.FindFirst(ClaimTypes.NameIdentifier)?.Value ?? "anonimo";

    return RateLimitPartition.GetFixedWindowLimiter(usuarioId, _ => new FixedWindowRateLimiterOptions
    {
        PermitLimit = 50,
        Window = TimeSpan.FromMinutes(1)
    });
});
```

👉 Combinando com o `User` que você já lê desde o post sobre JWT, cada usuário autenticado tem sua própria cota — um usuário abusivo não consome o limite de todos os outros

---

# ⚠️ Erros comuns

- Aplicar um único limite global, permitindo que um usuário abusivo prejudique todos os outros  
- Configurar limites tão baixos que usuários legítimos são bloqueados no uso normal  
- Não diferenciar endpoints críticos (login) de endpoints leves (listagem pública), aplicando a mesma regra para todos  
- Esquecer de logar quando um limite é atingido, perdendo visibilidade sobre possíveis ataques  

---

# 📌 Conclusão

- Rate Limiting protege a API contra uso excessivo, intencional ou acidental  
- `429 Too Many Requests` é o código HTTP correto para sinalizar o limite  
- Fixed Window, Sliding Window, Token Bucket e Concurrency Limiter cobrem diferentes padrões de tráfego  
- Limitar por usuário (não só globalmente) evita que um cliente prejudique todos os outros  

👉 Com Rate Limiting, sua API se protege do próprio sucesso — o tráfego alto deixa de ser um risco de derrubar o serviço

---

# 🔥 Próximo passo

Agora que sua API resiste a uso excessivo, o próximo nível é:

👉 **Fundamentos do C#: API Gateway com YARP**

Aqui você vai aprender a centralizar rate limiting, autenticação e roteamento na frente de múltiplos serviços.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
