# 🧠 Fundamentos do C#: Chaos Engineering

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Blue-Green e Canary para reduzir risco de deploy  
- Polly (post 40) para resiliência a falhas — retry, circuit breaker, timeout  

Você configurou Polly para lidar com falhas. Mas como saber se essa configuração realmente funciona sob condições reais de falha, antes que um incidente de produção seja o primeiro teste? Chaos Engineering resolve isso.

👉 **Vamos aprender Chaos Engineering**

---

# 💡 O problema de nunca testar falhas de verdade

```csharp
// Configurado desde o post 40, mas... funciona mesmo?
services.AddResiliencePipeline("padrao", builder =>
{
    builder.AddRetry(new RetryStrategyOptions { MaxRetryAttempts = 3 });
    builder.AddCircuitBreaker(new CircuitBreakerStrategyOptions());
});
```

👉 Você escreveu a configuração de resiliência, os testes unitários passam (post 30), mas nunca viu o comportamento real quando o serviço de pagamento realmente cai em produção, com tráfego real, sob carga real

---

# 🏗️ O que é Chaos Engineering?

👉 **Chaos Engineering = provocar falhas de propósito, em ambiente controlado, para descobrir fraquezas antes que aconteçam por acidente**

```
Hipótese: "Se o serviço de pagamento cair, o circuit breaker (post 40) 
deveria abrir em 5 falhas consecutivas, e o sistema deveria degradar 
graciosamente, sem derrubar o checkout inteiro"

Experimento: derrubar o serviço de pagamento propositalmente, em staging,
e observar se a hipótese se confirma
```

---

# 🎯 Tipos de falha para injetar

## 🔹 Latência artificial

```csharp
public class MiddlewareCaosLatencia
{
    public async Task InvokeAsync(HttpContext contexto, RequestDelegate proximo)
    {
        if (_flagCaos.EstaAtivo("latencia-artificial")) // lembra do post 51?
        {
            await Task.Delay(TimeSpan.FromSeconds(5));
        }

        await proximo(contexto);
    }
}
```

👉 Simula uma dependência lenta — testa se seus timeouts (Polly, post 40) realmente disparam quando deveriam

## 🔹 Falha de dependência

```csharp
if (_flagCaos.EstaAtivo("falha-servico-pagamento"))
{
    throw new HttpRequestException("Falha simulada");
}
```

👉 Simula o serviço de pagamento fora do ar — testa se o circuit breaker realmente abre e se o fallback (também do post 40) realmente funciona

## 🔹 Falha de infraestrutura

```bash
kubectl delete pod api-pedidos-7d9f8-x2j4k
```

👉 Lembra do Kubernetes (post 86)? Matar um Pod deliberadamente testa se o `Deployment` realmente sobe outro automaticamente, sem perda perceptível de serviço

---

# 🔍 Onde a observabilidade entra

```
Durante o experimento de caos:
  Taxa de erro do checkout: 2% (aceitável, degradação graciosa)
  Circuit breaker abriu após 5 falhas: ✅ confirmado
  Tempo de recuperação após o Pod voltar: 8 segundos
```

👉 Lembra do OpenTelemetry (post 55)? Sem métricas em tempo real, você não consegue distinguir entre "o sistema se comportou como esperado" e "o sistema quebrou silenciosamente" durante o experimento

---

# ⚖️ Chaos Engineering em produção vs staging

## 🔹 Staging
- Ambiente seguro para começar, sem impacto em usuários reais  
- Pode não replicar exatamente as condições de carga da produção  

## 🔹 Produção (com cautela)
- Netflix popularizou isso com o Chaos Monkey — matando instâncias aleatoriamente em produção  
- Exige maturidade operacional alta, começando com "blast radius" pequeno e controlado  

---

# ⚠️ Erros comuns

- Fazer experimentos de caos sem hipótese clara, sem saber o que exatamente está sendo validado  
- Rodar em produção sem munição de rollback rápido (lembra do Blue-Green do post anterior?) caso o experimento saia do controle  
- Não ter observabilidade suficiente para medir o impacto real do experimento  
- Testar só falhas óbvias, ignorando cenários combinados (latência + falha parcial + pico de tráfego ao mesmo tempo)  

---

# 📌 Conclusão

- Chaos Engineering provoca falhas de propósito para validar resiliência antes de um incidente real  
- Testa hipóteses específicas: "o circuit breaker realmente abre?", "o Pod realmente se recupera?"  
- Observabilidade (post 55) é pré-requisito para medir se o sistema se comportou como esperado  
- Comece em staging, com blast radius pequeno, antes de considerar produção  

👉 Com Chaos Engineering, você descobre as fraquezas do seu sistema em um experimento controlado, não em um incidente às 3 da manhã

---

# 🔥 Próximo passo

Agora que você testa resiliência de propósito, o próximo nível é:

👉 **Fundamentos do C#: Load Testing com k6**

Aqui você vai aprender a simular carga real antes que ela aconteça de verdade, descobrindo os limites do seu sistema.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
