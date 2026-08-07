# 🧠 Fundamentos do C#: gRPC — Comunicação Eficiente entre Serviços

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Boxing, unboxing e como tipos por valor afetam performance  
- Como serviços se comunicam via HTTP e mensageria  

Você já construiu APIs REST com JSON — funciona bem para front-ends e integrações públicas. Mas para comunicação **entre seus próprios microsserviços**, existe uma alternativa mais rápida e mais segura em relação a tipos.

👉 **Vamos conhecer o gRPC**

---

# 💡 O que é gRPC?

👉 **gRPC = um framework de comunicação que usa Protocol Buffers (binário) em vez de JSON, e HTTP/2 em vez de HTTP/1.1**

| | REST + JSON | gRPC |
|---|---|---|
| Formato | Texto (JSON) | Binário (Protocol Buffers) |
| Protocolo | HTTP/1.1 | HTTP/2 |
| Contrato | Geralmente informal (Swagger ajuda) | Formal, definido em `.proto` |
| Performance | Boa | Melhor (payload menor, multiplexação) |

👉 O ganho de performance vem de dois lugares: payloads binários são menores que JSON, e HTTP/2 permite múltiplas chamadas na mesma conexão, sem a sobrecarga de abrir uma nova a cada requisição

---

# 📝 Definindo o contrato: arquivos `.proto`

```protobuf
// produtos.proto
syntax = "proto3";

service ProdutoService {
  rpc ObterPorId (ObterProdutoRequest) returns (ProdutoResponse);
  rpc ListarTodos (Vazio) returns (ListaProdutosResponse);
}

message ObterProdutoRequest {
  int32 id = 1;
}

message ProdutoResponse {
  int32 id = 1;
  string nome = 2;
  double preco = 3;
}

message Vazio {}

message ListaProdutosResponse {
  repeated ProdutoResponse produtos = 1;
}
```

👉 Isso substitui o papel que `record`s e DTOs desempenham em uma API REST — só que aqui o **contrato vem primeiro**, e o C# é gerado automaticamente a partir dele, garantindo que cliente e servidor nunca fiquem dessincronizados sobre o formato dos dados

---

# 🏗️ Implementando o serviço

```bash
dotnet new grpc -o MeuServicoGrpc
```

```csharp
public class ProdutoServiceImpl : ProdutoService.ProdutoServiceBase
{
    private readonly IProdutoRepositorio _repositorio;

    public ProdutoServiceImpl(IProdutoRepositorio repositorio)
    {
        _repositorio = repositorio;
    }

    public override Task<ProdutoResponse> ObterPorId(ObterProdutoRequest request, ServerCallContext contexto)
    {
        var produto = _repositorio.ObterPorId(request.Id);

        return Task.FromResult(new ProdutoResponse
        {
            Id = produto.Id,
            Nome = produto.Nome,
            Preco = (double)produto.Preco
        });
    }
}
```

```csharp
// Program.cs
app.MapGrpcService<ProdutoServiceImpl>();
```

👉 Repare que `ProdutoServiceImpl` recebe `IProdutoRepositorio` pelo construtor — o mesmo mecanismo de injeção de dependência que você já domina, aplicado a um serviço gRPC em vez de um controller REST

---

# 📞 Consumindo o serviço a partir de outro microsserviço

```csharp
// Program.cs do serviço consumidor
builder.Services.AddGrpcClient<ProdutoService.ProdutoServiceClient>(options =>
{
    options.Address = new Uri("https://servico-produtos:5001");
});
```

```csharp
public class ServicoPedidos
{
    private readonly ProdutoService.ProdutoServiceClient _clienteProdutos;

    public ServicoPedidos(ProdutoService.ProdutoServiceClient clienteProdutos)
    {
        _clienteProdutos = clienteProdutos;
    }

    public async Task<ProdutoResponse> ObterProduto(int id)
    {
        return await _clienteProdutos.ObterPorIdAsync(new ObterProdutoRequest { Id = id });
    }
}
```

👉 A chamada parece um método local — `_clienteProdutos.ObterPorIdAsync(...)` — mas por trás acontece uma chamada de rede real, tipada de ponta a ponta pelo contrato `.proto`. Erros de tipo que só apareceriam em runtime numa API JSON são pegos em **tempo de compilação** aqui

---

# 🌊 Streaming: além do request/response

```protobuf
service PedidoService {
  rpc AcompanharStatus (AcompanharRequest) returns (stream StatusResponse);
}
```

```csharp
public override async Task AcompanharStatus(
    AcompanharRequest request, IServerStreamWriter<StatusResponse> responseStream, ServerCallContext contexto)
{
    while (!contexto.CancellationToken.IsCancellationRequested)
    {
        var status = await ObterStatusAtual(request.PedidoId);
        await responseStream.WriteAsync(new StatusResponse { Status = status });
        await Task.Delay(2000);
    }
}
```

👉 REST tradicional é sempre um request seguido de um response. gRPC suporta **streaming**: o servidor pode enviar múltiplas respostas ao longo do tempo pela mesma conexão — útil para atualizações de status, notificações em tempo real, ou grandes volumes de dados enviados em partes

---

# 🔀 Quando usar gRPC vs REST

## 🔹 Use gRPC quando:
- A comunicação é **entre seus próprios microsserviços**  
- Performance e baixa latência são críticas  
- Você quer um contrato fortemente tipado, verificado em tempo de compilação  

## 🔹 Use REST quando:
- A API precisa ser consumida por **navegadores** diretamente (suporte limitado a gRPC no browser)  
- É uma API **pública**, consumida por terceiros que esperam JSON e HTTP convencional  
- A simplicidade e familiaridade de REST pesam mais que o ganho de performance  

---

# ⚠️ Erros comuns

- Usar gRPC para uma API pública consumida por front-ends web, sem considerar as limitações de suporte no navegador  
- Não versionar o arquivo `.proto`, quebrando clientes antigos ao mudar um contrato existente  
- Ignorar que gRPC exige HTTP/2, que pode precisar de configuração extra em alguns ambientes de proxy/load balancer  
- Achar que gRPC substitui REST em todo cenário — a escolha depende de quem consome a API  

---

# 📌 Conclusão

- gRPC usa Protocol Buffers (binário) e HTTP/2, mais eficiente que REST + JSON  
- O contrato `.proto` gera código C# automaticamente, garantindo tipagem de ponta a ponta  
- Streaming permite múltiplas respostas ao longo de uma única conexão  
- gRPC brilha na comunicação entre microsserviços; REST continua sendo a escolha certa para APIs públicas  

👉 Com gRPC, a comunicação entre os serviços que você aprendeu a dividir no post anterior fica mais rápida e mais segura em relação a erros de contrato

---

# 🔥 Próximo passo

Agora que você sabe conectar serviços com eficiência, o próximo nível é:

👉 **Fundamentos do C#: Performance em C# (Span, Memory e Benchmarking)**

Aqui você vai aprender a medir e otimizar o desempenho do seu código no nível mais baixo.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
