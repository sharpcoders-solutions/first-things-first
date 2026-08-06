# 🧠 Fundamentos do C#: Deploy Blue-Green e Canary

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Terraform para versionar infraestrutura  
- CI/CD com GitHub Actions (post 36), automatizando deploys  

Até agora, "fazer deploy" significou substituir a versão antiga pela nova, de uma vez. Isso funciona, mas se a nova versão tiver um bug, todos os usuários são afetados imediatamente. Blue-Green e Canary reduzem esse risco.

👉 **Vamos aprender Deploy Blue-Green e Canary**

---

# 💡 O problema do deploy "tudo de uma vez"

```
Versão 1.0 rodando → deploy → Versão 1.1 rodando (100% do tráfego, instantaneamente)
```

👉 Se a versão 1.1 tiver um bug crítico, 100% dos usuários sentem o impacto ao mesmo tempo — e o rollback, mesmo rápido, ainda deixa uma janela de impacto total

---

# 🏗️ Blue-Green Deployment

```
Ambiente Blue (produção atual, v1.0) ← 100% do tráfego
Ambiente Green (nova versão, v1.1) ← 0% do tráfego, mas já rodando e testável
```

```yaml
# Service do Kubernetes (post 86) aponta para "blue"
apiVersion: v1
kind: Service
metadata:
  name: api-pedidos
spec:
  selector:
    app: api-pedidos
    versao: blue  # troca para "green" no momento da virada
```

👉 Você sobe a versão nova (Green) em paralelo, totalmente testável, sem receber tráfego real. Quando confiante, você troca o `Service` para apontar para Green — a virada é instantânea, e o rollback é só apontar de volta para Blue

---

# 🎯 Canary Deployment

```yaml
# 95% do tráfego para a versão estável
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-pedidos-estavel
spec:
  replicas: 19  # 95% de 20 réplicas totais

---
# 5% do tráfego para a versão nova
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-pedidos-canario
spec:
  replicas: 1  # 5% de 20 réplicas totais
```

👉 Em vez de uma virada total, você libera a versão nova para uma fatia pequena do tráfego real — se as métricas (lembra do post 55, OpenTelemetry?) continuarem saudáveis, você aumenta gradualmente a porcentagem até 100%

---

# 🔍 Observabilidade é o que torna isso seguro

```
Canário em 5% do tráfego:
  Taxa de erro: 0.1% (normal)
  Latência p99: 120ms (normal)
  → Aumentar para 25%

Canário em 25% do tráfego:
  Taxa de erro: 8% (anormal!)
  → Abortar e reverter para 0%
```

👉 Lembra do post sobre OpenTelemetry (55)? Sem métricas em tempo real comparando a versão nova com a estável, um Canary Deployment é só um Blue-Green mais lento — a decisão de avançar ou recuar depende inteiramente de conseguir observar o comportamento real

---

# ⚖️ Blue-Green vs Canary: quando usar cada um

## 🔹 Blue-Green
- Virada rápida, tudo ou nada  
- Rollback instantâneo (só trocar o roteamento de volta)  
- Exige capacidade duplicada temporariamente (dois ambientes completos rodando)  

## 🔹 Canary
- Exposição gradual, risco limitado a uma fração dos usuários  
- Permite comparar métricas reais entre versões antes de comprometer 100%  
- Mais lento para completar a virada total  

---

# ⚠️ Erros comuns

- Fazer Blue-Green sem migrar dados/schema de banco de forma compatível com ambas as versões simultaneamente  
- Rodar Canary sem métricas automatizadas, decidindo avançar "no olho" em vez de dados objetivos  
- Esquecer sessões com estado (sticky sessions) durante a transição, quebrando a experiência de usuários no meio de uma operação  
- Não automatizar o rollback — se depende de alguém acordar às 3h e reagir manualmente, o valor da estratégia cai muito  

---

# 📌 Conclusão

- Blue-Green troca o tráfego de uma vez, com rollback instantâneo  
- Canary libera a versão nova gradualmente, limitando o impacto de bugs  
- Ambas as estratégias dependem de observabilidade real para serem seguras  
- A escolha depende de quanto risco você aceita versus quanta capacidade extra você tem disponível  

👉 Com Blue-Green e Canary, colocar código novo em produção deixa de ser um evento de risco total e vira um processo controlado e reversível

---

# 🔥 Próximo passo

Agora que você reduz risco de deploy, o próximo nível é:

👉 **Fundamentos do C#: Chaos Engineering**

Aqui você vai aprender a testar a resiliência do seu sistema provocando falhas de propósito, antes que elas aconteçam por acidente.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
