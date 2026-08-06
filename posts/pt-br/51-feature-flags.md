# 🧠 Fundamentos do C#: Feature Flags e Configuração Dinâmica

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Toda a base de linguagem, arquitetura e segurança de uma API .NET  
- Como versionar e documentar uma API em produção  

Chegou a hora da segunda metade da trilha: construir os recursos que separam um sistema que "funciona" de um sistema **operado com maturidade** em produção.

👉 **Vamos começar por Feature Flags**

---

# 💡 O que é uma Feature Flag?

👉 **Feature Flag = uma condicional que liga ou desliga uma funcionalidade sem precisar de um novo deploy**

```csharp
if (featureManager.IsEnabled("NovoCheckout"))
{
    return NovoFluxoDeCheckout();
}

return FluxoDeCheckoutAtual();
```

Sem flags, ativar uma funcionalidade nova significa fazer deploy. Com flags, o código já está em produção, **desligado**, e você liga quando quiser — sem tocar em infraestrutura de novo.

---

# 🏗️ Usando `Microsoft.FeatureManagement`

```bash
dotnet add package Microsoft.FeatureManagement.AspNetCore
```

```json
// appsettings.json
{
  "FeatureManagement": {
    "NovoCheckout": true,
    "RelatoriosAvancados": false
  }
}
```

```csharp
// Program.cs
builder.Services.AddFeatureManagement();
```

```csharp
public class CheckoutController : ControllerBase
{
    private readonly IFeatureManager _featureManager;

    public CheckoutController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }

    [HttpPost]
    public async Task<IActionResult> Finalizar()
    {
        if (await _featureManager.IsEnabledAsync("NovoCheckout"))
            return Ok(NovoFluxo());

        return Ok(FluxoAtual());
    }
}
```

👉 `IFeatureManager` é injetado via DI — o mesmo mecanismo que você já domina desde o post sobre ASP.NET Core, agora controlando comportamento em vez de dependências

---

# 🎯 Rollout gradual: liberando aos poucos

```csharp
public class RegraPercentual : IFeatureFilter
{
    public Task<bool> EvaluateAsync(FeatureFilterEvaluationContext contexto)
    {
        var percentual = contexto.Parameters.GetValue<int>("Percentual");
        return Task.FromResult(new Random().Next(100) < percentual);
    }
}
```

```json
{
  "FeatureManagement": {
    "NovoCheckout": {
      "EnabledFor": [
        { "Name": "RegraPercentual", "Parameters": { "Percentual": 10 } }
      ]
    }
  }
}
```

👉 Em vez de "ligado para todo mundo" ou "desligado para todo mundo", você libera para 10% dos usuários, observa (lembra do post sobre health checks e logging?), e aumenta o percentual gradualmente se tudo estiver saudável

---

# 🔀 Feature Flags vs Configuração comum

## 🔹 Configuração comum (`appsettings.json`)
- Valores que raramente mudam (connection strings, URLs de serviços)  
- Definida no deploy, não muda em tempo real  

## 🔹 Feature Flag
- Controla **comportamento**, não só valores  
- Pode mudar em tempo real, sem reiniciar a aplicação (com um provider de configuração dinâmica, como Azure App Configuration)  
- Tem um ciclo de vida: nasce, é testada, é liberada, e **deve ser removida** quando a funcionalidade se torna permanente  

---

# 🧹 A dívida técnica das flags esquecidas

👉 **Toda flag é dívida técnica temporária, por design**

```csharp
// ❌ Flag esquecida há 8 meses, ninguém lembra por que existe
if (await _featureManager.IsEnabledAsync("TesteAntigoDeCheckoutDe2023"))
```

Uma flag que devia durar duas semanas de teste e ainda está no código um ano depois é sinal de processo quebrado — cada flag deveria ter um dono e uma data esperada de remoção

---

# ⚠️ Erros comuns

- Deixar flags antigas no código depois que a decisão já foi tomada, acumulando complexidade condicional  
- Usar flags para esconder código quebrado em vez de configuração comum para valores estáveis  
- Testar só com a flag ligada, esquecendo de validar o comportamento com ela desligada  
- Não documentar o propósito e a data esperada de remoção de cada flag  

---

# 📌 Conclusão

- Feature Flags desacoplam o deploy do lançamento de uma funcionalidade  
- `IFeatureManager` controla flags via DI, do mesmo jeito que qualquer outra dependência  
- Rollout percentual libera funcionalidades gradualmente, com observação real  
- Toda flag é temporária por natureza — remover é parte do processo, não um extra  

👉 Com feature flags, lançar algo novo deixa de ser um evento arriscado e vira uma decisão reversível e controlada

---

# 🔥 Próximo passo

Agora que você sabe controlar o que roda em produção, o próximo nível é:

👉 **Fundamentos do C#: Background Jobs com Hangfire**

Aqui você vai aprender a executar tarefas fora do ciclo de vida de uma requisição HTTP.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
