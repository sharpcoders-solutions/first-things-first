# 🧠 Fundamentos do C#: Autenticação e Autorização com JWT

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Extension Methods e como o LINQ é construído por baixo dos panos  
- Como persistir dados de verdade com EF Core  

Sua API está funcional, mas hoje ela aceita **qualquer requisição de qualquer pessoa**. Chegou a hora de fechar essa porta.

👉 **Vamos proteger a API com autenticação e autorização usando JWT**

---

# 💡 Autenticação vs Autorização: a distinção que todo mundo confunde

👉 **Autenticação = quem você é. Autorização = o que você pode fazer**

- **Autenticação**: o usuário prova sua identidade (login com usuário e senha)  
- **Autorização**: depois de identificado, o sistema decide se ele tem permissão para aquela ação específica  

👉 Um usuário pode estar **autenticado** (logado) e ainda assim **não autorizado** a acessar um endpoint de administrador

---

# 🔑 O que é JWT?

👉 **JWT (JSON Web Token) = um token compacto e autocontido que carrega informações verificáveis sobre o usuário**

Um JWT tem três partes, separadas por pontos:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMiLCJyb2xlIjoiQWRtaW4ifQ.4f8a...
   Header              Payload                              Signature
```

- **Header** → algoritmo de assinatura usado  
- **Payload** → os **claims** (dados sobre o usuário: id, nome, papel/role)  
- **Signature** → garante que o token não foi alterado depois de emitido  

👉 O servidor não precisa consultar o banco a cada requisição para saber quem é o usuário — as informações já vêm dentro do próprio token, assinadas de forma que qualquer alteração seja detectável

---

# 🏗️ Configurando a autenticação JWT

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

```csharp
// Program.cs
var chave = builder.Configuration["Jwt:Chave"];

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Emissor"],
            ValidAudience = builder.Configuration["Jwt:Audiencia"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(chave))
        };
    });

builder.Services.AddAuthorization();

// ...

app.UseAuthentication(); // sempre antes de UseAuthorization
app.UseAuthorization();
```

👉 `UseAuthentication` identifica **quem** está fazendo a requisição; `UseAuthorization` decide **se** essa pessoa pode acessar o recurso — a ordem entre os dois importa

---

# ✍️ Gerando um token no login

```csharp
[HttpPost("login")]
public IActionResult Login(LoginRequest request)
{
    // validação de usuário e senha (simplificada)
    if (request.Usuario != "admin" || request.Senha != "123456")
        return Unauthorized();

    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, request.Usuario),
        new Claim(ClaimTypes.Role, "Admin")
    };

    var chave = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuracao["Jwt:Chave"]));
    var credenciais = new SigningCredentials(chave, SecurityAlgorithms.HmacSha256);

    var token = new JwtSecurityToken(
        issuer: _configuracao["Jwt:Emissor"],
        audience: _configuracao["Jwt:Audiencia"],
        claims: claims,
        expires: DateTime.UtcNow.AddHours(2),
        signingCredentials: credenciais
    );

    return Ok(new { token = new JwtSecurityTokenHandler().WriteToken(token) });
}
```

👉 O `Claim` é a unidade básica de informação dentro do token — aqui carregamos o nome do usuário e seu papel (`Role`), que serão lidos depois para decisões de autorização

---

# 🔒 Protegendo endpoints com `[Authorize]`

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    [HttpGet]
    [Authorize] // exige qualquer usuário autenticado
    public IActionResult ObterTodos() => Ok(/* ... */);

    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")] // exige usuário autenticado E com papel Admin
    public IActionResult Deletar(int id) => Ok(/* ... */);
}
```

👉 Sem um token válido no header `Authorization: Bearer {token}`, a requisição recebe `401 Unauthorized`. Com token válido mas sem o papel exigido, recebe `403 Forbidden` — dois códigos HTTP diferentes para dois problemas diferentes

---

# 👤 Lendo os claims do usuário autenticado

```csharp
[HttpGet("meu-perfil")]
[Authorize]
public IActionResult MeuPerfil()
{
    string nome = User.Identity.Name;
    string papel = User.FindFirst(ClaimTypes.Role)?.Value;

    return Ok(new { nome, papel });
}
```

👉 Depois que o middleware de autenticação valida o token, o ASP.NET Core preenche `User` automaticamente com os claims — você nunca precisa decodificar o token manualmente

---

# ⚠️ Erros comuns

- Guardar a chave de assinatura (`Jwt:Chave`) direto no código-fonte, em vez de em configuração segura (variáveis de ambiente, secrets manager)  
- Esquecer `ValidateLifetime`, permitindo que tokens expirados continuem sendo aceitos  
- Confundir `401` (não autenticado) com `403` (autenticado, mas sem permissão) ao depurar problemas de acesso  
- Colocar dados sensíveis no payload do JWT — ele é **assinado**, não **criptografado**, então qualquer pessoa consegue ler o conteúdo, só não consegue alterá-lo sem invalidar a assinatura  

---

# 📌 Conclusão

- Autenticação prova identidade; autorização decide permissões  
- JWT carrega claims verificáveis, sem exigir consulta ao banco a cada requisição  
- `[Authorize]` protege endpoints; `[Authorize(Roles = "...")]` adiciona uma camada extra de controle  
- `401` significa "quem é você?"; `403` significa "eu sei quem você é, mas não pode"  

👉 Com autenticação e autorização, sua API deixa de confiar em quem está do outro lado da requisição e passa a **verificar**

---

# 🔥 Próximo passo

Agora que sua API está seguramente protegida, o próximo nível é:

👉 **Fundamentos do C#: IDisposable e o Padrão Dispose**

Aqui você vai aprender a liberar recursos não gerenciados corretamente, como conexões de banco e arquivos abertos.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
