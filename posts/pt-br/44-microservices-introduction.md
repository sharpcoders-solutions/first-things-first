# 🧠 Fundamentos do C#: Microsserviços — Introdução e Quando Usar

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- DDD e a ideia de Bounded Context  
- Mensageria para desacoplar sistemas no tempo  

Você já viu que cada Bounded Context pode ter seu próprio modelo. A pergunta natural que vem depois é: será que cada um também deveria ser uma **aplicação separada**?

👉 **Vamos entender o que são microsserviços, e — mais importante — quando eles realmente valem a pena**

---

# 💡 Monólito vs Microsserviços

## 🔹 Monólito

👉 Toda a aplicação (API, regras de negócio, acesso a dados) roda como **um único processo**, um único deploy

```
[ Monólito: Pedidos + Estoque + Pagamentos + Notificações ]
```

## 🔹 Microsserviços

👉 A aplicação é dividida em **serviços independentes**, cada um com seu próprio banco de dados, deploy e ciclo de vida

```
[ Serviço Pedidos ]   [ Serviço Estoque ]   [ Serviço Pagamentos ]   [ Serviço Notificações ]
```

👉 Cada serviço se comunica com os outros via **HTTP/API** ou **mensageria** (exatamente o que você aprendeu no post sobre RabbitMQ) — nunca acessando diretamente o banco de dados de outro serviço

---

# 🗺️ Onde o DDD entra: Bounded Context como fronteira natural

Lembra do exemplo de "Cliente" significando coisas diferentes em Vendas e Suporte? Cada Bounded Context bem definido é um candidato natural a virar um microsserviço:

```
Bounded Context "Vendas"  → Serviço de Vendas
Bounded Context "Estoque" → Serviço de Estoque
Bounded Context "Suporte" → Serviço de Suporte
```

👉 Microsserviços mal divididos geralmente vêm de um erro de modelagem de domínio, não de infraestrutura — dividir sem entender as fronteiras do domínio costuma criar serviços que se chamam o tempo todo, o pior dos dois mundos

---

# ✅ Quando microsserviços fazem sentido

- **Times grandes**, onde equipes diferentes precisam publicar de forma independente, sem esperar umas pelas outras  
- **Escalabilidade desigual**: um serviço de pagamentos pode precisar de 10x mais instâncias que o serviço de notificações  
- **Tecnologias diferentes por domínio**: um serviço de recomendação pode se beneficiar de Python/ML, enquanto o resto do sistema continua em C#  
- **Isolamento de falhas**: um problema no serviço de notificações não deveria derrubar o checkout  

---

# ❌ Quando microsserviços são um erro caro

👉 **A armadilha mais comum: adotar microsserviços antes de precisar deles**

- Times pequenos pagam o custo de **operar** múltiplos serviços (deploy, monitoramento, comunicação de rede) sem o benefício de escala real  
- Debugar um problema que atravessa cinco serviços é ordens de magnitude mais difícil que debugar um monólito bem organizado  
- Transações que envolvem múltiplos serviços perdem a garantia simples de consistência que um único banco de dados oferece  

**Regra prática:** comece com um **monólito bem modularizado** (usando os limites de Bounded Context como divisórias internas de código, não de deploy). Extraia um microsserviço só quando sentir uma dor real e específica que a divisão resolve — não porque "é assim que empresas grandes fazem".

---

# 🔗 Como os serviços se comunicam

## 🔹 Síncrono (HTTP/API)

```csharp
public class ServicoPedidos
{
    private readonly HttpClient _clienteEstoque;

    public async Task<bool> VerificarDisponibilidade(int produtoId, int quantidade)
    {
        var resposta = await _clienteEstoque.GetAsync($"/estoque/{produtoId}");
        var estoque = await resposta.Content.ReadFromJsonAsync<EstoqueDto>();
        return estoque.Quantidade >= quantidade;
    }
}
```

👉 Simples, mas cria acoplamento temporal: se o serviço de estoque estiver fora do ar, a criação do pedido também falha — é aqui que o post sobre Polly se torna essencial, não opcional

## 🔹 Assíncrono (mensageria)

```csharp
_publicador.Publicar(new PedidoCriadoEvento(pedido.Id));
// o serviço de estoque consome esse evento de forma independente, no seu próprio tempo
```

👉 Reduz o acoplamento temporal — o serviço de pedidos não espera o de estoque responder, exatamente o padrão do post sobre RabbitMQ

---

# 🌐 API Gateway: um ponto de entrada único

```
Cliente → [ API Gateway ] → Serviço Pedidos
                          → Serviço Estoque
                          → Serviço Pagamentos
```

👉 Em vez do front-end conhecer o endereço de cada microsserviço individualmente, um **API Gateway** centraliza autenticação, roteamento e agregação de respostas — o cliente conversa com um único ponto, mesmo que por trás existam dez serviços

---

# ⚠️ Erros comuns

- Migrar para microsserviços sem ter o problema de escala ou organização que os justificaria  
- Criar "microsserviços distribuídos": serviços separados que ainda compartilham o mesmo banco de dados, perdendo o isolamento que é a razão de existir do padrão  
- Não aplicar resiliência (Polly) nas chamadas entre serviços, deixando uma falha em cascata derrubar o sistema inteiro  
- Dividir serviços por camada técnica (ex: "serviço de banco de dados") em vez de por domínio de negócio  

---

# 📌 Conclusão

- Monólitos concentram tudo em um processo; microsserviços dividem em serviços independentes  
- Bounded Context (do DDD) é a fronteira natural para dividir microsserviços de forma sensata  
- Microsserviços resolvem problemas de escala de time e de infraestrutura — não são um objetivo por si só  
- Comunicação síncrona cria acoplamento temporal; mensageria assíncrona reduz esse acoplamento  
- API Gateway centraliza o ponto de entrada para múltiplos serviços  

👉 A decisão mais importante sobre microsserviços não é "como implementar", é "será que eu realmente preciso disso agora"

---

# 🔥 Próximo passo

Agora que você entende quando dividir um sistema em serviços, o próximo nível é:

👉 **Fundamentos do C#: gRPC — Comunicação Eficiente entre Serviços**

Aqui você vai aprender uma alternativa mais rápida e tipada que REST para a comunicação síncrona entre microsserviços.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
