# 🧠 Fundamentos do C#: Kubernetes

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- AWS Lambda e o modelo serverless  
- Docker (post 35), empacotando sua aplicação em um único container  

O Docker resolve "como empacotar minha aplicação". Mas quando você tem dezenas de containers, precisando escalar, se recuperar de falhas e se comunicar entre si, gerenciar isso manualmente vira inviável. Kubernetes resolve orquestração em escala.

👉 **Vamos aprender Kubernetes**

---

# 💡 Do Docker ao Kubernetes

👉 **Kubernetes (K8s) = um orquestrador que gerencia onde, quantos e como seus containers rodam, automaticamente**

```
Docker (post 35): "aqui está minha aplicação empacotada"
Kubernetes: "rode 5 cópias dessa aplicação, reinicie se cair, distribua a carga, escale sob demanda"
```

---

# 🏗️ Os conceitos fundamentais

## 🔹 Pod: a menor unidade

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-pedidos
spec:
  containers:
    - name: api-pedidos
      image: minhaempresa/api-pedidos:1.0.0
      ports:
        - containerPort: 8080
```

👉 Um Pod embrulha um ou mais containers (geralmente um, a imagem Docker do post 35) que rodam juntos como uma unidade

## 🔹 Deployment: gerenciando réplicas

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-pedidos
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-pedidos
  template:
    metadata:
      labels:
        app: api-pedidos
    spec:
      containers:
        - name: api-pedidos
          image: minhaempresa/api-pedidos:1.0.0
```

👉 `replicas: 3` garante 3 instâncias rodando sempre — se uma cair, o Kubernetes sobe outra automaticamente, sem intervenção manual

## 🔹 Service: expondo os Pods

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-pedidos-service
spec:
  selector:
    app: api-pedidos
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

👉 Lembra do YARP (post 54)? Um `Service` faz papel parecido — distribui tráfego entre os Pods disponíveis, sem o cliente precisar saber qual instância específica está respondendo

---

# 🎯 Autoscaling: reagindo à carga

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-pedidos-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-pedidos
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

👉 Quando o uso de CPU passa de 70%, o Kubernetes automaticamente cria mais Pods — e reduz quando a carga cai. Isso combina diretamente com o Native AOT (post 69): Pods novos que iniciam rápido respondem à demanda quase instantaneamente

---

# 🔍 Health Checks, do mesmo jeito que você já configurou

```yaml
livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  periodSeconds: 5
```

👉 Lembra do post 38? Os mesmos endpoints de liveness e readiness que você construiu com Health Checks do ASP.NET Core são consumidos diretamente pelo Kubernetes — se o `livenessProbe` falhar, o Pod é reiniciado automaticamente

---

# 🔧 ConfigMaps e Secrets: configuração no Kubernetes

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-pedidos-config
data:
  ApiConfiguration__TimeoutSeconds: "30"
```

👉 Lembra do Options Pattern (post 76)? `ConfigMaps` alimentam essas mesmas configurações fortemente tipadas, sem hardcodar valores na imagem do container

---

# ⚠️ Erros comuns

- Rodar um único Pod sem `Deployment`, perdendo a auto-recuperação em caso de falha  
- Não configurar `readinessProbe`, fazendo o Kubernetes enviar tráfego para um Pod que ainda não terminou de inicializar  
- Ignorar limites de recursos (`resources.limits`), permitindo que um Pod consuma memória/CPU de forma descontrolada e afete os vizinhos  
- Subestimar a curva de aprendizado do Kubernetes — a complexidade operacional é real e deve ser avaliada contra o ganho real de orquestração  

---

# 📌 Conclusão

- Kubernetes orquestra containers em escala: réplicas, autoscaling, recuperação automática de falhas  
- Pods, Deployments e Services são os blocos fundamentais  
- Health Checks (post 38) e Options Pattern (post 76) se conectam diretamente com probes e ConfigMaps  
- Autoscaling combinado com Native AOT (post 69) responde à demanda quase instantaneamente  

👉 Com Kubernetes, sua aplicação containerizada ganha resiliência e escala automática, sem você gerenciar servidor por servidor manualmente

---

# 🔥 Próximo passo

Agora que você orquestra containers, o próximo nível é:

👉 **Fundamentos do C#: Helm Charts**

Aqui você vai aprender a empacotar configurações complexas do Kubernetes de forma reutilizável e versionada.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
