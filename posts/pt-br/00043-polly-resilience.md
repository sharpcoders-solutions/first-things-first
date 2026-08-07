# 🧠 Fundamentos do C#: Resiliência com Polly

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Cache em memória e distribuído  
- Health checks das suas dependências  

Você já sabe **detectar** quando uma dependência está falhando. Mas detectar não é o mesmo que **lidar bem** com a falha. Uma API externa instável, uma rede lenta por alguns segundos — isso é normal, e sua aplicação precisa sobreviver a isso sem quebrar.

👉 **É para isso que existe resiliência, e a biblioteca Polly é o padrão do ecossistema .NET**

---

# 💡 O que é resiliência de software?

👉 **Resiliência = a capacidade da aplicação continuar funcionando (ou falhar graciosamente) diante de falhas temporárias**

Falhas transitórias são inevitáveis: timeout de rede, serviço momentaneamente sobrecarregado, instabilidade de conexão. A pergunta não é **se** vão acontecer, é **como sua aplicação reage** quando acontecem.

---

# 🏗️ Instalando o Polly

```bash
dotnet add package Microsoft.Extensions.Http.Polly
```

```csharp
// Program.cs
builder.Services.AddHttpClient<ApiExternaClient>()
    .AddPolicyHandler(GetPoliticaDeRetry());

static IAsyncPolicy<HttpResponseMessage> GetPoliticaDeRetry()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(3, tentativa => TimeSpan.FromSeconds(Math.Pow(2, tentativa)));
}
```

👉 O Polly se integra direto com `HttpClient` via injeção de dependência — a mesma configuração que você já viu em posts anteriores, agora protegendo chamadas HTTP externas

---

# 🔁 Retry: tentando de novo

```csharp
var politica = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, tentativa => TimeSpan.FromSeconds(tentativa * 2));

await politica.ExecuteAsync(() => _httpClient.GetAsync("/api/dados"));
```

## 🔹 Backoff exponencial

👉 Em vez de tentar de novo imediatamente (o que pode piorar um serviço já sobrecarregado), o intervalo entre tentativas **cresce**: 2s, 4s, 8s... dando tempo para o serviço se recuperar

---

# ⚡ Circuit Breaker: parando de insistir

```csharp
var circuitBreaker = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(
        handledEventsAllowedBeforeBreaking: 5,
        durationOfBreak: TimeSpan.FromSeconds(30));
```

## 🔹 Como funciona

- Depois de **5 falhas seguidas**, o circuito "abre" — novas chamadas falham **imediatamente**, sem nem tentar  
- Depois de 30 segundos, o circuito entra em modo de teste, permitindo uma chamada para ver se o serviço voltou  

👉 Sem circuit breaker, sua aplicação continuaria martelando um serviço já derrubado, piorando a situação. Com ele, você para de bater na porta de um serviço que sabe que não vai responder — e sua própria aplicação fica mais responsiva, já que não fica presa esperando timeouts repetidos

---

# ⏱️ Timeout: não esperar para sempre

```csharp
var timeout = Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(5));
```

👉 Sem timeout explícito, uma dependência lenta pode travar sua aplicação por tempo indefinido — lembra do post sobre async/await? Uma thread presa esperando é uma thread que não está atendendo outros usuários

---

# 🔗 Combinando políticas

```csharp
var politicaCombinada = Policy.WrapAsync(
    circuitBreaker,
    politicaDeRetry,
    timeout);

await politicaCombinada.ExecuteAsync(() => _httpClient.GetAsync("/api/dados"));
```

👉 As políticas se combinam: timeout limita cada tentativa, retry tenta novamente diante de falha, e circuit breaker impede tentativas repetidas contra um serviço já sabidamente indisponível

---

# 🛟 Fallback: tendo um plano B

```csharp
var fallback = Policy<List<Produto>>
    .Handle<Exception>()
    .FallbackAsync(new List<Produto>()); // devolve lista vazia em vez de quebrar

var produtos = await fallback.ExecuteAsync(() => BuscarProdutosDaApiExterna());
```

👉 Nem toda falha precisa virar um erro para o usuário final. Um fallback devolve um resultado padrão (uma lista vazia, um valor em cache, uma resposta genérica) quando a operação principal falha

---

# ⚠️ Erros comuns

- Fazer retry em erros que **não são transitórios** (ex: `400 Bad Request` nunca vai funcionar só de tentar de novo)  
- Configurar retries sem backoff, martelando um serviço já instável ainda mais rápido  
- Não combinar timeout com retry, permitindo que cada tentativa demore indefinidamente  
- Aplicar circuit breaker com limites tão baixos que o circuito abre por instabilidades momentâneas normais  

---

# 📌 Conclusão

- Resiliência é sobre lidar bem com falhas transitórias, não evitá-las  
- Retry com backoff exponencial dá tempo para o serviço se recuperar  
- Circuit breaker evita insistir em um serviço já sabidamente indisponível  
- Timeout impede que uma dependência lenta trave sua aplicação indefinidamente  
- Fallback garante um plano B quando a operação principal falha  

👉 Com Polly, sua aplicação para de assumir que tudo vai dar certo e passa a se preparar, de forma explícita, para quando algo não der

---

# 🔥 Próximo passo

Agora que sua aplicação lida bem com falhas, o próximo nível é:

👉 **Fundamentos do C#: Mensageria com RabbitMQ**

Aqui você vai aprender a desacoplar sistemas no tempo, usando filas de mensagens em vez de chamadas diretas e síncronas.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
