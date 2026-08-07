# 🧠 Fundamentos do C#: Criando e Publicando Pacotes NuGet

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Roslyn Analyzers para distribuir regras customizadas  
- Você já usou dezenas de pacotes NuGet ao longo desta trilha (Serilog, Hangfire, xUnit, Polly...)  

Toda vez que você escreveu `dotnet add package`, alguém empacotou e publicou aquele código para o mundo usar. Agora é sua vez de fazer o mesmo com seu próprio código.

👉 **Vamos aprender a criar e publicar pacotes NuGet**

---

# 💡 O que é um pacote NuGet?

👉 **NuGet = o gerenciador de pacotes do .NET — um pacote é uma DLL compilada, empacotada com metadados, versão e dependências**

Todo pacote que você já usou nesta trilha segue o mesmo formato — desde o `Microsoft.EntityFrameworkCore` até o `Hangfire.AspNetCore`

---

# 🏗️ Criando uma biblioteca compartilhável

```bash
dotnet new classlib -n MinhaEmpresa.Utils
cd MinhaEmpresa.Utils
```

```csharp
namespace MinhaEmpresa.Utils;

public static class ExtensoesString
{
    public static string ParaSlug(this string valor)
    {
        return valor.ToLowerInvariant()
            .Replace(" ", "-")
            .Normalize(NormalizationForm.FormD);
    }
}
```

👉 Nada de especial até aqui — é uma classe normal, do mesmo jeito que você escreve desde o post sobre métodos (post 17). O que muda é como ela é empacotada e distribuída

---

# 📦 Configurando o `.csproj` para empacotamento

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <PackageId>MinhaEmpresa.Utils</PackageId>
    <Version>1.0.0</Version>
    <Authors>Vitor Santos</Authors>
    <Description>Extensões utilitárias compartilhadas da MinhaEmpresa</Description>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
  </PropertyGroup>
</Project>
```

```bash
dotnet pack -c Release
```

👉 Isso gera um arquivo `.nupkg` — o mesmo formato de arquivo que você baixa automaticamente toda vez que roda `dotnet add package`

---

# 🚀 Publicando no NuGet.org

```bash
dotnet nuget push bin/Release/MinhaEmpresa.Utils.1.0.0.nupkg \
  --api-key SUA_CHAVE_API \
  --source https://api.nuget.org/v3/index.json
```

👉 Depois de publicado, qualquer pessoa no mundo pode instalar com `dotnet add package MinhaEmpresa.Utils` — o mesmo comando que você usou para o Serilog, Polly e todos os outros

---

# 🏢 Feeds privados para pacotes internos

```bash
dotnet nuget add source https://pkgs.dev.azure.com/minhaempresa/_packaging/interno/nuget/v3/index.json \
  --name "feed-interno"

dotnet nuget push MinhaEmpresa.Utils.1.0.0.nupkg --source "feed-interno"
```

👉 Nem todo pacote deveria ir para o NuGet.org público — código interno da empresa vai para um feed privado (Azure Artifacts, GitHub Packages), com a mesma mecânica, mas acesso restrito ao time

---

# 🔢 Versionamento semântico

```
1.0.0 → 1.0.1  (patch: correção de bug, sem quebrar nada)
1.0.1 → 1.1.0  (minor: nova funcionalidade, compatível)
1.1.0 → 2.0.0  (major: quebra de compatibilidade)
```

👉 Cada dependência que você adicionou ao longo desta trilha respeita essa convenção — seguir o mesmo padrão evita que uma atualização "pequena" quebre o código de quem consome seu pacote

---

# ⚠️ Erros comuns

- Publicar um pacote sem testes (lembra do post 30?), fazendo com que bugs cheguem direto para quem consome  
- Não seguir versionamento semântico, quebrando compatibilidade em uma versão "patch"  
- Esquecer de documentar o pacote (README, XML doc comments), deixando quem usa sem saber como funciona  
- Publicar segredos ou chaves de API acidentalmente dentro do pacote  

---

# 📌 Conclusão

- NuGet empacota código C# compilado, com metadados e dependências  
- `dotnet pack` gera o `.nupkg`; `dotnet nuget push` publica  
- Feeds privados mantêm código interno fora do NuGet.org público  
- Versionamento semântico comunica o impacto de cada atualização  

👉 Com pacotes NuGet, seu código deixa de viver isolado em um projeto e passa a ser reutilizável em toda a organização — ou no mundo inteiro

---

# 🔥 Próximo passo

Agora que você sabe empacotar e compartilhar código, o próximo nível é:

👉 **Fundamentos do C#: Lock, Monitor e Sincronização**

Aqui você vai aprender a proteger seções críticas de código quando múltiplas threads acessam o mesmo estado ao mesmo tempo.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
