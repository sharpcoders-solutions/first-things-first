# 🧠 Fundamentos do C#: Identity Server e OAuth2

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Observabilidade completa, consolidando métricas, traces e logs  
- JWT (post 34) — você já emite e valida tokens, mas sempre dentro da própria API  

No post 34, sua própria API criava e validava o JWT. Mas e quando você precisa que o usuário faça login com Google, ou que múltiplas aplicações compartilhem um único provedor de identidade? Isso é OAuth2 e OpenID Connect.

👉 **Vamos aprofundar em Identity Server e OAuth2**

---

# 💡 Revisitando o JWT do post 34 com mais contexto

```csharp
// Post 34: sua própria API emitia o token
var token = new JwtSecurityToken(
    issuer: "minhaapi.com",
    claims: claims,
    expires: DateTime.UtcNow.AddHours(1));
```

👉 Isso funciona bem quando só existe uma API. Mas se você tem 5 aplicações diferentes, cada uma reimplementando login, ou quer permitir "Entrar com Google" — você precisa de um provedor de identidade centralizado

---

# 🏗️ OAuth2: papéis, não implementação

👉 **OAuth2 é um protocolo de autorização — define papéis e fluxos, não uma biblioteca específica**

```
Resource Owner: o usuário
Client: a aplicação que quer acessar dados em nome do usuário
Authorization Server: emite tokens (ex: Identity Server, Auth0, Azure AD)
Resource Server: sua API, que valida o token (o que você já faz desde o post 34)
```

---

# 🎯 O fluxo Authorization Code (o mais comum)

```
1. Usuário clica em "Entrar" no seu app
2. App redireciona para o Authorization Server
3. Usuário faz login lá (não na sua aplicação)
4. Authorization Server redireciona de volta com um "código"
5. Sua aplicação troca esse código por um Access Token + Refresh Token
6. Sua API valida o Access Token — mesmo mecanismo do post 34
```

```csharp
// Program.cs — configurando sua API como Resource Server
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opcoes =>
    {
        opcoes.Authority = "https://meu-identity-server.com";
        opcoes.Audience = "minha-api";
    });
```

👉 A diferença central: sua API não emite mais o token, só **confia** em um emissor externo — o `[Authorize]` que você já usa desde o post 34 continua funcionando exatamente igual

---

# 🔐 OpenID Connect: identidade em cima do OAuth2

```
OAuth2 sozinho: "esse cliente tem permissão para acessar este recurso"
OpenID Connect (OIDC): OAuth2 + "e esse é o usuário: nome, e-mail, foto"
```

```csharp
var claims = User.Claims;
var nome = claims.First(c => c.Type == "name").Value;
var email = claims.First(c => c.Type == "email").Value;
```

👉 OAuth2 puro resolve autorização; OIDC adiciona identidade — o `id_token` (diferente do `access_token`) carrega essas claims sobre quem é o usuário, no mesmo formato JWT que você já conhece

---

# 🏢 Duas.NET Options: Identity Server vs provedores gerenciados

## 🔹 Duende IdentityServer (self-hosted)

```csharp
builder.Services.AddIdentityServer()
    .AddInMemoryClients(Config.Clients)
    .AddInMemoryApiScopes(Config.ApiScopes)
    .AddDeveloperSigningCredential();
```

👉 Você hospeda e controla o Authorization Server inteiro — máximo controle, máxima responsabilidade operacional

## 🔹 Provedores gerenciados (Azure AD, Auth0, Okta)

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opcoes =>
    {
        opcoes.Authority = "https://minhaempresa.b2clogin.com/...";
    });
```

👉 Terceiriza a complexidade de segurança de identidade — geralmente a escolha certa, a menos que você tenha requisitos muito específicos de customização

---

# ⚠️ Erros comuns

- Implementar seu próprio Authorization Server do zero sem necessidade real, quando um provedor gerenciado resolveria com muito menos risco  
- Confundir `access_token` com `id_token` — um autoriza acesso a recursos, o outro carrega identidade  
- Não validar o `Audience` do token, aceitando tokens emitidos para outra aplicação  
- Armazenar tokens de forma insegura no cliente (localStorage é vulnerável a XSS) em vez de cookies HttpOnly  

---

# 📌 Conclusão

- OAuth2 define papéis e fluxos de autorização; OpenID Connect adiciona identidade em cima dele  
- O fluxo Authorization Code é o padrão mais comum para aplicações web  
- Sua API continua validando tokens com `[Authorize]`, só que confiando em um emissor externo  
- Provedores gerenciados geralmente são a escolha certa sobre hospedar seu próprio Identity Server  

👉 Com OAuth2 e OpenID Connect, autenticação deixa de ser reinventada em cada aplicação e se torna um serviço centralizado e confiável

---

# 🔥 Próximo passo

Agora que você domina autenticação federada, o próximo nível é:

👉 **Fundamentos do C#: Criptografia e Proteção de Dados**

Aqui você vai aprender a proteger dados sensíveis em repouso e em trânsito, além da autenticação.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
