# 🧠 Fundamentos do C#: API Gateway com YARP

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Rate Limiting para proteger uma única API  
- gRPC para comunicação eficiente entre serviços  

Quando sua arquitetura cresce para vários serviços independentes, o cliente não deveria precisar saber o endereço de cada um. É aí que entra um API Gateway.

👉 **Vamos aprender a construir um Gateway com YARP (Yet Another Reverse Proxy)**

---

# 💡 O que é um API Gateway?

👉 **API Gateway = um ponto de entrada único que roteia requisições para os serviços corretos por trás dele**

Sem gateway, o cliente precisa saber:

```
GET http://servico-pedidos:5001/pedidos/123
GET http://servico-estoque:5002/estoque/456
GET http://servico-pagamento:5003/pagamentos/789
```

Com gateway:

```
GET http://api.minhaempresa.com/pedidos/123
GET http://api.minhaempresa.com/estoque/456
GET http://api.minhaempresa.com/pagamentos/789
```

👉 O gateway conhece a topologia interna; o cliente só conhece **um** endereço

---

# 🏗️ Configurando o YARP

```bash
dotnet add package Yarp.ReverseProxy
```

```csharp
// Program.cs
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

// ...

app.MapReverseProxy();
```

```json
// appsettings.json
{
  "ReverseProxy": {
    "Routes": {
      "rota-pedidos": {
        "ClusterId": "cluster-pedidos",
        "Match": { "Path": "/pedidos/{**catch-all}" }
      },
      "rota-estoque": {
        "ClusterId": "cluster-estoque",
        "Match": { "Path": "/estoque/{**catch-all}" }
      }
    },
    "Clusters": {
      "cluster-pedidos": {
        "Destinations": {
          "destino1": { "Address": "http://servico-pedidos:5001" }
        }
      },
      "cluster-estoque": {
        "Destinations": {
          "destino1": { "Address": "http://servico-estoque:5002" }
        }
      }
    }
  }
}
```

👉 Toda a configuração de roteamento fica declarativa no `appsettings.json` — nenhuma linha de código de roteamento manual

---

# 🎯 Centralizando responsabilidades transversais

## 🔹 Rate Limiting no Gateway, não em cada serviço

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("global", opcoes =>
    {
        opcoes.PermitLimit = 1000;
        opcoes.Window = TimeSpan.FromMinutes(1);
    });
});
```

👉 Lembra do post anterior? Ao invés de configurar rate limiting em cada microsserviço, você centraliza no gateway — um só lugar protege todos os serviços de trás

## 🔹 Autenticação centralizada

```csharp
app.MapReverseProxy().RequireAuthorization();
```

👉 O JWT (post 37) é validado uma vez no gateway. Os serviços internos podem confiar que toda requisição que chega já foi autenticada

## 🔹 Load balancing entre múltiplas instâncias

```json
"cluster-pedidos": {
  "LoadBalancingPolicy": "RoundRobin",
  "Destinations": {
    "destino1": { "Address": "http://servico-pedidos-1:5001" },
    "destino2": { "Address": "http://servico-pedidos-2:5001" }
  }
}
```

👉 Se você escalou o serviço de pedidos horizontalmente (múltiplas instâncias rodando, lembra do post sobre Docker?), o gateway distribui a carga entre elas automaticamente

---

# 🔄 Transformações de requisição

```json
"rota-pedidos": {
  "ClusterId": "cluster-pedidos",
  "Match": { "Path": "/api/v2/pedidos/{**catch-all}" },
  "Transforms": [
    { "PathPattern": "/pedidos/{**catch-all}" }
  ]
}
```

👉 O cliente externo chama `/api/v2/pedidos`, mas o gateway reescreve para `/pedidos` internamente — o versionamento de API pode viver só na camada do gateway, sem forçar mudanças nos serviços internos

---

# ⚠️ Erros comuns

- Colocar lógica de negócio no gateway, que deveria ser uma camada puramente de roteamento e responsabilidades transversais  
- Não configurar timeouts, deixando um serviço lento travar requisições no gateway indefinidamente  
- Esquecer que o gateway é um ponto único de falha — sem alta disponibilidade nele, toda a arquitetura cai junto  
- Duplicar autenticação em cada serviço interno depois de já centralizá-la no gateway, gerando complexidade desnecessária  

---

# 📌 Conclusão

- Um API Gateway centraliza o roteamento para múltiplos serviços atrás de um único endereço  
- YARP configura rotas e clusters declarativamente, sem código de roteamento manual  
- Rate limiting, autenticação e load balancing ficam centralizados no gateway, não duplicados em cada serviço  
- Transformações de path desacoplam a API pública da estrutura interna dos serviços  

👉 Com um Gateway, sua arquitetura de microsserviços ganha um ponto de entrada único, consistente e mais fácil de proteger

---

# 🔥 Próximo passo

Agora que você tem um ponto de entrada centralizado, o próximo nível é:

👉 **Fundamentos do C#: IEnumerable e Iteradores Customizados**

Aqui você vai aprender como o `foreach` realmente funciona por baixo dos panos, e como criar suas próprias sequências customizadas com `yield return`.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
