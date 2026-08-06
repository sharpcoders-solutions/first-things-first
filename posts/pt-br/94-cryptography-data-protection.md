# 🧠 Fundamentos do C#: Criptografia e Proteção de Dados

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- OAuth2 e OpenID Connect para autenticação federada  
- OWASP Top 10 (post 47), incluindo exposição de dados sensíveis como uma das vulnerabilidades mais críticas  

Autenticação garante que você sabe **quem** está acessando. Mas os dados em si — senhas, CPFs, números de cartão — precisam de proteção própria, tanto guardados no banco quanto trafegando pela rede.

👉 **Vamos aprender Criptografia e Proteção de Dados**

---

# 💡 Hashing vs Criptografia: a distinção que evita bugs graves

```csharp
// ❌ Nunca "criptografe" senhas — isso implica que podem ser desкриptografadas
var senhaCriptografada = Criptografar(senha);

// ✅ Senhas são "hasheadas" — processo de mão única, irreversível
var hashSenha = HashearSenha(senha);
```

👉 Criptografia é reversível (com a chave certa). Hashing não é — se alguém acessar seu banco, uma senha hasheada corretamente é inútil para eles, mesmo com acesso total ao banco de dados

---

# 🏗️ Hashing de senhas do jeito certo

```csharp
public class ServicoSenha
{
    public string GerarHash(string senha)
    {
        return BCrypt.Net.BCrypt.HashPassword(senha, workFactor: 12);
    }

    public bool Verificar(string senha, string hashArmazenado)
    {
        return BCrypt.Net.BCrypt.Verify(senha, hashArmazenado);
    }
}
```

```csharp
// ❌ NUNCA faça isso
var hash = SHA256.HashData(Encoding.UTF8.GetBytes(senha)); // rápido demais, vulnerável a força bruta
```

👉 BCrypt (ou Argon2) é **propositalmente lento** — isso é uma característica, não um defeito, porque dificulta ataques de força bruta. SHA256 é rápido demais e não deve ser usado para senhas, mesmo sendo um algoritmo de hash "seguro" para outros propósitos

---

# 🔐 Criptografando dados sensíveis em repouso

```csharp
public class ServicoCriptografia
{
    private readonly byte[] _chave;

    public string Criptografar(string textoPuro)
    {
        using var aes = Aes.Create();
        aes.Key = _chave;
        aes.GenerateIV();

        using var criptografador = aes.CreateEncryptor();
        var bytesTexto = Encoding.UTF8.GetBytes(textoPuro);
        var bytesCriptografados = criptografador.TransformFinalBlock(bytesTexto, 0, bytesTexto.Length);

        return Convert.ToBase64String(aes.IV.Concat(bytesCriptografados).ToArray());
    }
}
```

👉 Dados como número de cartão ou CPF (quando você precisa deles de volta, diferente de senha) usam AES — simétrico, reversível com a chave certa, e a chave nunca deve viver no mesmo lugar que os dados criptografados

---

# 🔑 Onde guardar as chaves: nunca no código

```csharp
// ❌ Nunca faça isso
private readonly byte[] _chave = Convert.FromBase64String("MinhaChaveSecreta123==");

// ✅ Use um cofre de segredos
var chave = await _clienteKeyVault.GetSecretAsync("chave-criptografia-pedidos");
```

👉 Lembra do post sobre configuração (76)? Chaves de criptografia seguem o mesmo princípio de segredos que discutimos lá, mas com ainda mais rigor — Azure Key Vault, AWS Secrets Manager ou HashiCorp Vault gerenciam rotação e acesso, algo que um `appsettings.json` nunca deveria fazer

---

# 🌐 Dados em trânsito: HTTPS não é opcional

```csharp
// Program.cs
app.UseHttpsRedirection();
app.UseHsts(); // força HTTPS em requisições futuras do navegador
```

👉 Mesmo com dados criptografados em repouso, uma requisição HTTP sem TLS expõe tudo em texto plano durante o trajeto pela rede — HTTPS protege o "em trânsito" da mesma forma que AES protege o "em repouso"

---

# 🛡️ ASP.NET Core Data Protection API

```csharp
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"/chaves-compartilhadas"))
    .SetApplicationName("MinhaEmpresa.Pedidos");
```

```csharp
public class ServicoTokenSeguro
{
    private readonly IDataProtector _protetor;

    public ServicoTokenSeguro(IDataProtectionProvider provedor)
    {
        _protetor = provedor.CreateProtector("TokensRecuperacaoSenha");
    }

    public string Proteger(string dado) => _protetor.Protect(dado);
    public string Desproteger(string dadoProtegido) => _protetor.Unprotect(dadoProtegido);
}
```

👉 O ASP.NET Core já vem com uma API de criptografia gerenciada — para casos comuns (tokens temporários, cookies), ela cuida de rotação de chaves e detalhes de implementação que você não precisaria reinventar manualmente

---

# ⚠️ Erros comuns

- Usar o mesmo algoritmo (hash) para senhas e para dados que precisam ser recuperados — são problemas diferentes, com soluções diferentes  
- Guardar chaves de criptografia no controle de versão, mesmo que "temporariamente"  
- Confiar só em HTTPS e esquecer de criptografar dados sensíveis também em repouso, no banco  
- Implementar seu próprio algoritmo de criptografia em vez de usar bibliotecas testadas e revisadas (`System.Security.Cryptography`)  

---

# 📌 Conclusão

- Hashing (senhas) e criptografia (dados recuperáveis) resolvem problemas diferentes, com ferramentas diferentes  
- BCrypt/Argon2 são propositalmente lentos para dificultar força bruta; nunca use hash rápido para senhas  
- Chaves de criptografia vivem em cofres de segredos, nunca no código ou no controle de versão  
- HTTPS protege dados em trânsito; criptografia protege dados em repouso — ambos são necessários  

👉 Com criptografia bem aplicada, mesmo um vazamento de banco de dados não expõe diretamente as informações mais sensíveis dos seus usuários

---

# 🔥 Próximo passo

Agora que você protege dados sensíveis corretamente, o próximo nível é:

👉 **Fundamentos do C#: Clean Code**

Aqui você vai consolidar princípios de código limpo que perpassaram esta trilha inteira, desde o primeiro post.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
