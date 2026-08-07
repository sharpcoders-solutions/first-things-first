# 🧠 C# Fundamentals: Kubernetes

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- AWS Lambda and the serverless model  
- Docker (post 35), packaging your application into a single container  

Docker solves "how do I package my application." But when you have dozens of containers that need to scale, recover from failures, and communicate with each other, managing that manually becomes impossible. Kubernetes solves orchestration at scale.

👉 **Let's learn Kubernetes**

---

# 💡 From Docker to Kubernetes

👉 **Kubernetes (K8s) = an orchestrator that manages where, how many, and how your containers run, automatically**

```
Docker (post 35): "here's my application, packaged"
Kubernetes: "run 5 copies of this application, restart it if it crashes, distribute load, scale on demand"
```

---

# 🏗️ The fundamental concepts

## 🔹 Pod: the smallest unit

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: orders-api
spec:
  containers:
    - name: orders-api
      image: mycompany/orders-api:1.0.0
      ports:
        - containerPort: 8080
```

👉 A Pod wraps one or more containers (usually one, the Docker image from post 35) that run together as a unit

## 🔹 Deployment: managing replicas

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: orders-api
  template:
    metadata:
      labels:
        app: orders-api
    spec:
      containers:
        - name: orders-api
          image: mycompany/orders-api:1.0.0
```

👉 `replicas: 3` guarantees 3 instances always running — if one goes down, Kubernetes brings up another automatically, with no manual intervention

## 🔹 Service: exposing the Pods

```yaml
apiVersion: v1
kind: Service
metadata:
  name: orders-api-service
spec:
  selector:
    app: orders-api
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

👉 Remember YARP (post 54)? A `Service` plays a similar role — it distributes traffic across the available Pods, without the client needing to know which specific instance is responding

---

# 🎯 Autoscaling: reacting to load

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: orders-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: orders-api
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

👉 When CPU usage goes above 70%, Kubernetes automatically creates more Pods — and scales down when load drops. This connects directly with Native AOT (post 69): new Pods that start up fast respond to demand almost instantly

---

# 🔍 Health Checks, the same way you already set them up

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

👉 Remember post 38? The same liveness and readiness endpoints you built with ASP.NET Core Health Checks are consumed directly by Kubernetes — if `livenessProbe` fails, the Pod gets restarted automatically

---

# 🔧 ConfigMaps and Secrets: configuration in Kubernetes

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: orders-api-config
data:
  ApiConfiguration__TimeoutSeconds: "30"
```

👉 Remember the Options Pattern (post 76)? `ConfigMaps` feed those exact same strongly-typed settings, without hardcoding values into the container image

---

# ⚠️ Common Mistakes

- Running a single Pod without a `Deployment`, losing automatic recovery on failure  
- Not configuring `readinessProbe`, causing Kubernetes to send traffic to a Pod that hasn't finished starting up yet  
- Ignoring resource limits (`resources.limits`), letting a Pod consume memory/CPU uncontrollably and affect its neighbors  
- Underestimating Kubernetes's learning curve — the operational complexity is real and should be weighed against the actual orchestration gain  

---

# 📌 Conclusion

- Kubernetes orchestrates containers at scale: replicas, autoscaling, automatic failure recovery  
- Pods, Deployments, and Services are the fundamental building blocks  
- Health Checks (post 38) and Options Pattern (post 76) connect directly with probes and ConfigMaps  
- Autoscaling combined with Native AOT (post 69) responds to demand almost instantly  

👉 With Kubernetes, your containerized application gains resilience and automatic scale, without you managing server by server manually

---

# 🔥 Next Step

Now that you can orchestrate containers, the next level is:

👉 **C# Fundamentals: Helm Charts**

Here you'll learn to package complex Kubernetes configurations in a reusable, versioned way.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
