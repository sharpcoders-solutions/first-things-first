# 🧠 Fundamentos do C#: Azure Functions

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- .NET MAUI para apps nativos multiplataforma  
- Docker (post 35) e deploy de aplicações .NET completas, sempre rodando  

Toda aplicação que você fez nesta trilha até agora fica "ligada" o tempo todo, esperando requisições. E se você só precisar rodar código esporadicamente — um processamento de imagem quando um arquivo é enviado, uma tarefa às 3 da manhã? Serverless resolve isso.

👉 **Vamos aprender Azure Functions**

---

# 💡 O que é Serverless?

👉 **Serverless = você escreve só a função; a nuvem cuida do servidor, escala automaticamente, e você paga só pelo tempo de execução**

```csharp
public class ProcessarImagem
{
    [Function("ProcessarImagem")]
    public async Task Run(
        [BlobTrigger("uploads/{nome}")] Stream imagem,
        string nome,
        FunctionContext contexto)
    {
        var logger = contexto.GetLogger("ProcessarImagem");
        logger.LogInformation($"Processando {nome}");

        // gerar thumbnail, redimensionar, etc.
    }
}
```

👉 Sem `Program.cs` com `WebApplicationBuilder`, sem servidor rodando 24/7 — a função só executa quando um evento acontece (nesse caso, upload de um arquivo)

---

# 🏗️ Tipos de trigger

## 🔹 HTTP Trigger

```csharp
[Function("ObterPedido")]
public HttpResponseData Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = "pedidos/{id}")] HttpRequestData requisicao,
    int id)
{
    var resposta = requisicao.CreateResponse(HttpStatusCode.OK);
    resposta.WriteAsJsonAsync(new { Id = id, Status = "Confirmado" });
    return resposta;
}
```

👉 Parecido com um endpoint REST (post 31), mas sem toda a infraestrutura do ASP.NET Core rodando continuamente — só ativa quando alguém chama

## 🔹 Timer Trigger

```csharp
[Function("LimpezaNoturna")]
public void Run([TimerTrigger("0 0 3 * * *")] TimerInfo timer)
{
    // roda toda noite às 3h, sem precisar de um servidor "ligado" esperando
}
```

👉 Lembra do Hangfire (post 52)? Isso é conceitualmente parecido, mas sem precisar manter um processo rodando o tempo todo — a nuvem "acorda" a função só na hora certa

## 🔹 Queue Trigger

```csharp
[Function("ProcessarPedido")]
public async Task Run([QueueTrigger("pedidos-pendentes")] string mensagem)
{
    var pedido = JsonSerializer.Deserialize<Pedido>(mensagem);
    // processa o pedido
}
```

👉 Similar ao consumidor do RabbitMQ (post 41), mas sem você gerenciar a infraestrutura de consumo — a função escala automaticamente conforme a fila cresce

---

# ⚡ Onde o Native AOT (post 69) se conecta

```xml
<PropertyGroup>
  <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

👉 Lembra do post sobre Native AOT? Cold start é crítico em serverless — cada nova instância da função precisa iniciar rapidamente, e as mesmas técnicas de inicialização rápida que discutimos naquele post se aplicam diretamente aqui

---

# 💰 O modelo de custo muda tudo

```
Aplicação tradicional (Docker/Kubernetes):
  Paga 24/7, mesmo que a aplicação fique 90% do tempo ociosa

Azure Functions:
  Paga só pelos milissegundos de execução real + número de invocações
```

👉 Para cargas de trabalho esporádicas ou imprevisíveis, isso muda completamente a equação de custo comparado às abordagens de deploy contínuo do post 35

---

# ⚠️ Erros comuns

- Usar Functions para cargas de trabalho constantes e previsíveis, onde uma aplicação tradicional sairia mais barata  
- Não considerar cold start em cenários sensíveis à latência — a primeira invocação após um período ocioso é mais lenta  
- Escrever funções com estado compartilhado em memória, esperando que ele persista entre execuções (cada invocação pode rodar em uma instância diferente)  
- Ignorar limites de tempo de execução — funções têm timeout, não são adequadas para processamento de longa duração sem quebrar em etapas  

---

# 📌 Conclusão

- Serverless elimina a necessidade de gerenciar servidor, escalando automaticamente por evento  
- Triggers HTTP, Timer e Queue cobrem os padrões mais comuns de ativação  
- O modelo de custo por execução favorece cargas esporádicas ou imprevisíveis  
- Native AOT e técnicas de cold start rápido são especialmente relevantes aqui  

👉 Com Azure Functions, você escreve só a lógica de negócio, e deixa toda a infraestrutura de escala e disponibilidade para a nuvem

---

# 🔥 Próximo passo

Agora que você conhece serverless na Azure, o próximo nível é:

👉 **Fundamentos do C#: AWS Lambda com .NET**

Aqui você vai aprender a mesma filosofia serverless, agora na maior nuvem pública do mundo.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
