# 🧠 Fundamentos do C#: Load Testing com k6

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Chaos Engineering para testar resiliência a falhas  
- Autoscaling do Kubernetes (post 86), reagindo à carga automaticamente  

Chaos Engineering testa "o que acontece quando algo quebra". Load Testing testa uma pergunta diferente: "quantos usuários simultâneos meu sistema aguenta antes de degradar?" Descobrir isso em um teste controlado é muito melhor que descobrir na Black Friday.

👉 **Vamos aprender Load Testing com k6**

---

# 💡 O que o k6 testa que testes normais não testam

```csharp
// Teste de integração (post 59): valida comportamento, com 1 requisição por vez
[Fact]
public async Task PostPedido_DeveRetornar201()
{
    var resposta = await _cliente.PostAsJsonAsync("/pedidos", novoPedido);
    Assert.Equal(HttpStatusCode.Created, resposta.StatusCode);
}
```

👉 Esse teste garante que o endpoint **funciona**. Ele não diz nada sobre o que acontece quando 1000 usuários chamam esse mesmo endpoint ao mesmo tempo

---

# 🏗️ Escrevendo um teste de carga com k6

```javascript
// carga.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },   // sobe gradualmente até 50 usuários
    { duration: '1m', target: 50 },    // mantém 50 usuários por 1 minuto
    { duration: '30s', target: 0 },    // desce gradualmente até 0
  ],
};

export default function () {
  const resposta = http.get('https://minhaapi.com/pedidos/123');

  check(resposta, {
    'status é 200': (r) => r.status === 200,
    'tempo de resposta < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);
}
```

```bash
k6 run carga.js
```

👉 O k6 simula usuários reais fazendo requisições simultâneas, com estágios de subida e descida — parecido com o padrão de tráfego real de uma aplicação, não uma rajada artificial instantânea

---

# 🎯 O que observar nos resultados

```
     http_req_duration..............: avg=145ms  p(95)=380ms  p(99)=720ms
     http_req_failed.................: 0.42%
     http_reqs.......................: 15234  (253.9/s)
```

👉 Lembra do post sobre OpenTelemetry (55)? Os mesmos conceitos de p95/p99 de latência aparecem aqui — a média (`avg`) esconde outliers, então `p(95)` e `p(99)` mostram a experiência real dos usuários mais afetados

---

# 🔍 Descobrindo o ponto de ruptura

```javascript
export const options = {
  stages: [
    { duration: '1m', target: 100 },
    { duration: '1m', target: 500 },
    { duration: '1m', target: 1000 },
    { duration: '1m', target: 2000 }, // continua subindo até algo quebrar
  ],
};
```

👉 Um teste de estresse aumenta a carga até o sistema degradar — o objetivo não é "passar", é descobrir exatamente onde está o limite, para saber com antecedência quando o autoscaling (post 86) precisa entrar em ação

---

# 🔌 Conectando com autoscaling e performance

```yaml
# HorizontalPodAutoscaler (post 86)
minReplicas: 2
maxReplicas: 10
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        averageUtilization: 70
```

👉 O teste de carga valida se esse autoscaler realmente reage a tempo — se o k6 mostra que 500 usuários simultâneos derrubam a latência antes do HPA criar novos Pods, isso é uma configuração que precisa de ajuste, descoberta em teste, não em produção

---

# ⚠️ Erros comuns

- Rodar testes de carga direto em produção sem avisar o time, gerando alertas falsos de incidente  
- Testar só o "caminho feliz", ignorando endpoints mais pesados (relatórios, buscas complexas)  
- Não isolar o ambiente de teste — carga de teste competindo com tráfego real distorce os resultados  
- Ignorar os testes depois de rodar uma vez — capacidade muda conforme o código evolui, load testing deveria ser contínuo, não pontual  

---

# 📌 Conclusão

- Load Testing descobre os limites reais do sistema, sob carga simulada  
- k6 simula usuários concorrentes com estágios de subida e descida, parecido com tráfego real  
- Métricas p95/p99 revelam a experiência dos usuários mais afetados, não só a média  
- Os resultados validam diretamente se o autoscaling (post 86) reage a tempo  

👉 Com Load Testing, você descobre os limites do seu sistema em um ambiente controlado, muito antes deles aparecerem no pior momento possível

---

# 🔥 Próximo passo

Agora que você conhece os limites do seu sistema sob carga, o próximo nível é:

👉 **Fundamentos do C#: Observabilidade Completa (Métricas, Traces, Logs)**

Aqui você vai consolidar tudo que aprendeu sobre observabilidade em uma visão unificada e completa.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
