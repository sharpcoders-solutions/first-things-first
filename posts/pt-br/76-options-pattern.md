# 🧠 Fundamentos do C#: Options Pattern e Configuração Avançada

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Nullable Reference Types em profundidade  
- `appsettings.json` básico, usado desde o post sobre feature flags (51)  

Você já leu configuração com `builder.Configuration["Chave"]` algumas vezes nesta trilha. Isso funciona, mas é frágil — nenhuma checagem de tipo, nenhum autocomplete, erros só aparecem em runtime. Options Pattern resolve isso.

👉 **Vamos aprender Options Pattern**

---

# 💡 O problema da configuração fracamente tipada

```csharp
// ❌ Frágil: string mágica, sem checagem de tipo, sem autocomplete
var tempoLimite = int.Parse(builder.Configuration["Api:TempoLimiteSegundos"]);
```

👉 Se alguém renomear a chave no `appsettings.json` sem atualizar essa string, o erro só aparece em runtime — nada no compilador te avisa

---

# 🏗️ Configurando o Options Pattern

```json
// appsettings.json
{
  "ConfiguracaoApi": {
    "TempoLimiteSegundos": 30,
    "UrlBase": "https://api.exemplo.com",
    "TentativasMaximas": 3
  }
}
```

```csharp
public class ConfiguracaoApi
{
    public const string Secao = "ConfiguracaoApi";

    public int TempoLimiteSegundos { get; set; }
    public string UrlBase { get; set; } = default!;
    public int TentativasMaximas { get; set; }
}
```

```csharp
// Program.cs
builder.Services.Configure<ConfiguracaoApi>(
    builder.Configuration.GetSection(ConfiguracaoApi.Secao));
```

👉 Fortemente tipado (lembra do post sobre NRT?), com autocomplete e checagem de tipo em tempo de compilação — o erro de "chave errada" vira erro de compilação, não de produção

---

# 🎯 Consumindo via injeção de dependência

```csharp
public class ClienteApi
{
    private readonly ConfiguracaoApi _configuracao;

    public ClienteApi(IOptions<ConfiguracaoApi> opcoes)
    {
        _configuracao = opcoes.Value;
    }

    public async Task<string> Buscar(string caminho)
    {
        using var http = new HttpClient { BaseAddress = new Uri(_configuracao.UrlBase) };
        http.Timeout = TimeSpan.FromSeconds(_configuracao.TempoLimiteSegundos);

        var resposta = await http.GetAsync(caminho);
        return await resposta.Content.ReadAsStringAsync();
    }
}
```

👉 O mesmo padrão de injeção de dependência que você usa desde o post sobre ASP.NET Core — `IOptions<T>` é só mais uma dependência resolvida pelo container

---

# 🔄 IOptionsSnapshot: configuração que recarrega

```csharp
public class ServicoComRecarregamento
{
    private readonly IOptionsSnapshot<ConfiguracaoApi> _opcoes;

    public ServicoComRecarregamento(IOptionsSnapshot<ConfiguracaoApi> opcoes)
    {
        _opcoes = opcoes;
    }

    public void Processar()
    {
        // sempre pega o valor mais atual do appsettings.json, sem reiniciar a aplicação
        Console.WriteLine(_opcoes.Value.TempoLimiteSegundos);
    }
}
```

👉 `IOptions<T>` é resolvido uma vez, na inicialização. `IOptionsSnapshot<T>` recarrega o valor a cada requisição — combinado com `reloadOnChange: true` no `appsettings.json`, você muda configuração sem reiniciar a aplicação, parecido com o espírito das feature flags do post 51

---

# ✅ Validação de configuração na inicialização

```csharp
builder.Services.AddOptions<ConfiguracaoApi>()
    .Bind(builder.Configuration.GetSection(ConfiguracaoApi.Secao))
    .Validate(config => config.TentativasMaximas > 0, "TentativasMaximas deve ser maior que zero")
    .ValidateOnStart();
```

👉 `ValidateOnStart()` falha a aplicação imediatamente na inicialização se a configuração estiver inválida — muito melhor que descobrir em produção, três chamadas depois, que `TentativasMaximas` estava zerado

---

# ⚠️ Erros comuns

- Continuar usando `Configuration["Chave"]` diretamente em código novo, ignorando os benefícios de tipagem forte  
- Usar `IOptions<T>` quando a configuração precisa recarregar em runtime — nesse caso, `IOptionsSnapshot<T>` é o certo  
- Não validar a configuração na inicialização, deixando valores inválidos derrubarem a aplicação só quando usados  
- Misturar segredos (connection strings, chaves de API) direto no `appsettings.json` versionado — use User Secrets ou variáveis de ambiente para isso  

---

# 📌 Conclusão

- Options Pattern transforma configuração fracamente tipada em classes fortemente tipadas  
- `IOptions<T>` resolve uma vez; `IOptionsSnapshot<T>` recarrega a cada requisição  
- `ValidateOnStart()` falha rápido, na inicialização, em vez de silenciosamente em produção  
- O mesmo mecanismo de injeção de dependência de sempre torna o consumo natural  

👉 Com Options Pattern, configuração deixa de ser um conjunto de strings mágicas e passa a ser um cidadão de primeira classe do seu sistema de tipos

---

# 🔥 Próximo passo

Agora que você domina configuração avançada, o próximo nível é:

👉 **Fundamentos do C#: Multi-tenancy**

Aqui você vai aprender a servir múltiplos clientes isolados a partir de uma única aplicação.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
