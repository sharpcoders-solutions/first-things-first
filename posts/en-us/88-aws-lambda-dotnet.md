# 🧠 C# Fundamentals: AWS Lambda with .NET

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Azure Functions and the serverless model  
- Continuous deployment of .NET applications, which you can package and run on any cloud  

The previous post introduced you to serverless on Azure. Now let's see the same philosophy on AWS — the world's largest public cloud, and probably one you'll run into at some point in your career.

👉 **Let's learn AWS Lambda with .NET**

---

# 💡 The concepts, revisited on AWS

👉 **Lambda = AWS's implementation of the same serverless model from the previous post: you write the function, AWS handles the rest**

The principles are the same as post 87 — no server management, automatic scaling, pay per execution. What changes is the syntax and the surrounding ecosystem

---

# 🏗️ Creating a Lambda function in C#

```bash
dotnet new lambda.EmptyFunction -n MyLambdaFunction
```

```csharp
public class Function
{
    public string FunctionHandler(Order order, ILambdaContext context)
    {
        context.Logger.LogInformation($"Processing order {order.Id}");
        return $"Order {order.Id} processed successfully";
    }
}
```

👉 Much simpler than Azure Functions in terms of signature — a method that receives the input event and the execution context, and returns the result

---

# 🎯 API Gateway + Lambda: the equivalent of the HTTP Trigger

```csharp
public class Function
{
    public APIGatewayProxyResponse FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
    {
        var orderId = request.PathParameters["id"];

        return new APIGatewayProxyResponse
        {
            StatusCode = 200,
            Body = JsonSerializer.Serialize(new { Id = orderId, Status = "Confirmed" }),
            Headers = new Dictionary<string, string> { { "Content-Type", "application/json" } }
        };
    }
}
```

👉 Remember Azure Functions' HTTP Trigger (post 87)? Here, AWS's API Gateway plays that role — it receives the HTTP request and invokes the Lambda, similar in spirit, different in implementation

---

# 📬 SQS Trigger: the equivalent of the Queue Trigger

```csharp
public async Task FunctionHandler(SQSEvent sqsEvent, ILambdaContext context)
{
    foreach (var record in sqsEvent.Records)
    {
        var order = JsonSerializer.Deserialize<Order>(record.Body);
        await ProcessOrder(order);
    }
}
```

👉 AWS's SQS (Simple Queue Service) is the equivalent of Azure's Storage Queue — the Lambda scales automatically as the queue grows, the same pattern from the previous post

---

# ⚡ Native AOT matters here too

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
</PropertyGroup>
```

```bash
dotnet lambda deploy-function --function-runtime provided.al2023
```

👉 Remember post 72? AWS offers the `provided.al2023` runtime specifically to run Native AOT executables — cold start in traditional C# could be a weak point against languages like Go or Node.js, and Native AOT closes that gap almost entirely

---

# ⚖️ Azure Functions vs AWS Lambda: what actually changes

## 🔹 Concepts (identical)
- Serverless, pay-per-execution, automatic scaling, event triggers  

## 🔹 Implementation (different)
- Different method signatures  
- Trigger ecosystem with names and configurations specific to each cloud  
- Cloud-specific deployment tools (`dotnet lambda` vs Azure Functions Core Tools)  

👉 Once you understand the serverless mental model (post 87), moving between Azure and AWS is mostly about learning new syntax, not a new paradigm

---

# ⚠️ Common Mistakes

- Assuming code is 100% portable between Azure Functions and AWS Lambda with no adaptation — the SDKs and signatures are different  
- Not configuring IAM permissions correctly, one of the most common error sources in Lambda (different from Azure's authorization model)  
- Bundling unnecessary dependencies into the deployment, increasing package size and cold start time  
- Ignoring the configurable memory and timeout limits per function — tuning these values directly impacts cost and performance  

---

# 📌 Conclusion

- AWS Lambda follows the same serverless model as Azure Functions, with its own syntax and ecosystem  
- API Gateway and SQS are the equivalents of Azure's HTTP and Queue triggers  
- Native AOT (post 72) is especially valuable on AWS, with support via the `provided.al2023` runtime  
- The serverless concept transfers between clouds; the implementation requires specific learning  

👉 With AWS Lambda, you confirm that the C# and .NET fundamentals you built throughout this track are portable across clouds — only the surface syntax changes

---

# 🔥 Next Step

Now that you've mastered serverless on the two biggest clouds, the next level is:

👉 **C# Fundamentals: Volatile, MemoryBarrier, and the .NET Memory Model**

Here you'll understand how threads actually see shared memory, and why instruction reordering can break multithreaded code that looks correct.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
