# 🧠 Fundamentos do C#: AWS Lambda com .NET

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Azure Functions e o modelo serverless  
- Deploy contínuo de aplicações .NET, que você pode empacotar e rodar em qualquer nuvem  

O post anterior te apresentou serverless na Azure. Agora vamos ver a mesma filosofia na AWS — a maior nuvem pública do mundo, e provavelmente a que você vai encontrar em algum ponto da carreira.

👉 **Vamos aprender AWS Lambda com .NET**

---

# 💡 Os conceitos, revisitados na AWS

👉 **Lambda = a implementação da AWS do mesmo modelo serverless do post anterior: você escreve a função, a AWS cuida do resto**

Os princípios são os mesmos do post 84 — sem gerenciar servidor, escala automática, paga por execução. O que muda é a sintaxe e o ecossistema ao redor

---

# 🏗️ Criando uma função Lambda em C#

```bash
dotnet new lambda.EmptyFunction -n MinhaFuncaoLambda
```

```csharp
public class Function
{
    public string FunctionHandler(Pedido pedido, ILambdaContext contexto)
    {
        contexto.Logger.LogInformation($"Processando pedido {pedido.Id}");
        return $"Pedido {pedido.Id} processado com sucesso";
    }
}
```

👉 Bem mais simples que o Azure Functions em termos de assinatura — um método que recebe o evento de entrada e o contexto de execução, e retorna o resultado

---

# 🎯 API Gateway + Lambda: o equivalente ao HTTP Trigger

```csharp
public class Function
{
    public APIGatewayProxyResponse FunctionHandler(APIGatewayProxyRequest requisicao, ILambdaContext contexto)
    {
        var pedidoId = requisicao.PathParameters["id"];

        return new APIGatewayProxyResponse
        {
            StatusCode = 200,
            Body = JsonSerializer.Serialize(new { Id = pedidoId, Status = "Confirmado" }),
            Headers = new Dictionary<string, string> { { "Content-Type", "application/json" } }
        };
    }
}
```

👉 Lembra do HTTP Trigger do Azure Functions (post 84)? Aqui, o API Gateway da AWS faz esse papel — recebe a requisição HTTP e invoca a Lambda, parecido em espírito, diferente em implementação

---

# 📬 SQS Trigger: o equivalente ao Queue Trigger

```csharp
public async Task FunctionHandler(SQSEvent evento, ILambdaContext contexto)
{
    foreach (var registro in evento.Records)
    {
        var pedido = JsonSerializer.Deserialize<Pedido>(registro.Body);
        await ProcessarPedido(pedido);
    }
}
```

👉 O SQS (Simple Queue Service) da AWS é o equivalente ao Storage Queue do Azure — a Lambda escala automaticamente conforme a fila cresce, o mesmo padrão do post anterior

---

# ⚡ Native AOT também importa aqui

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
</PropertyGroup>
```

```bash
dotnet lambda deploy-function --function-runtime provided.al2023
```

👉 Lembra do post 69? A AWS oferece o runtime `provided.al2023` justamente para rodar executáveis Native AOT — cold start em C# tradicional podia ser um ponto fraco frente a linguagens como Go ou Node.js, e o Native AOT fecha essa lacuna quase completamente

---

# ⚖️ Azure Functions vs AWS Lambda: o que realmente muda

## 🔹 Conceitos (idênticos)
- Serverless, pague-por-execução, escala automática, triggers por evento  

## 🔹 Implementação (diferente)
- Assinaturas de método diferentes  
- Ecossistema de triggers com nomes e configurações próprias de cada nuvem  
- Ferramentas de deploy específicas (`dotnet lambda` vs Azure Functions Core Tools)  

👉 Uma vez que você entende o modelo mental de serverless (post 84), migrar entre Azure e AWS é principalmente aprender uma nova sintaxe, não um novo paradigma

---

# ⚠️ Erros comuns

- Achar que o código é 100% portável entre Azure Functions e AWS Lambda sem nenhuma adaptação — os SDKs e assinaturas são diferentes  
- Não configurar corretamente as permissões IAM, um dos pontos mais comuns de erro em Lambda (diferente do modelo de autorização do Azure)  
- Empacotar dependências desnecessárias no deploy, aumentando o tamanho do pacote e o tempo de cold start  
- Ignorar limites de memória e timeout configuráveis por função — ajustar esses valores impacta diretamente custo e performance  

---

# 📌 Conclusão

- AWS Lambda segue o mesmo modelo serverless do Azure Functions, com sintaxe e ecossistema próprios  
- API Gateway e SQS são os equivalentes aos triggers HTTP e Queue do Azure  
- Native AOT (post 69) é especialmente valioso na AWS, com suporte via runtime `provided.al2023`  
- O conceito de serverless é transferível entre nuvens; a implementação exige aprendizado específico  

👉 Com AWS Lambda, você confirma que os fundamentos de C# e .NET que construiu nesta trilha inteira são portáveis entre nuvens — só a sintaxe de superfície muda

---

# 🔥 Próximo passo

Agora que você domina serverless nas duas maiores nuvens, o próximo nível é:

👉 **Fundamentos do C#: DateOnly e TimeOnly**

Aqui você vai aprender os tipos que resolvem um problema antigo do `DateTime`: representar só uma data, ou só um horário, sem carregar a parte que não faz sentido para o seu domínio.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
