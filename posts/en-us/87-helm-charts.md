# 🧠 C# Fundamentals: Helm Charts

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Kubernetes and its YAML manifests (Pods, Deployments, Services)  
- NuGet (post 63), packaging and versioning reusable C# code  

The previous post showed you individual YAML manifests. In a real application, you have dozens of these files — Deployment, Service, ConfigMap, HPA — all needing to change together across environments (dev, staging, production). Helm solves this.

👉 **Let's learn Helm Charts**

---

# 💡 The problem with scattered YAMLs

```
k8s/
  deployment-dev.yaml
  deployment-staging.yaml
  deployment-production.yaml
  service.yaml
  configmap-dev.yaml
  configmap-production.yaml
  ...
```

👉 Duplicating manifests per environment is fragile — a change needs to be manually replicated across several files, with a high risk of drift

---

# 🏗️ Helm: Kubernetes's "NuGet"

👉 **Helm Chart = a versioned package of Kubernetes manifests, parameterizable per environment**

```
my-chart/
  Chart.yaml          # metadata, similar to post 63's .csproj
  values.yaml          # default values
  templates/
    deployment.yaml
    service.yaml
    configmap.yaml
```

```yaml
# Chart.yaml
apiVersion: v2
name: orders-api
version: 1.0.0
appVersion: "1.0.0"
```

👉 This directly mirrors the NuGet post — a Chart has a version, metadata, and can be published to a repository for reuse, the same way a C# library can

---

# 🎯 Templates: parameterized YAML

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
spec:
  replicas: {{ .Values.replicas }}
  template:
    spec:
      containers:
        - name: {{ .Values.appName }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          resources:
            limits:
              memory: {{ .Values.resources.memoryLimit }}
```

```yaml
# values.yaml (defaults)
appName: orders-api
replicas: 2
image:
  repository: mycompany/orders-api
  tag: latest
resources:
  memoryLimit: 256Mi
```

👉 `{{ .Values.X }}` inserts values dynamically — the same template serves dev, staging, and production, each with its own values file

---

# 🌍 Values per environment

```yaml
# values-production.yaml
replicas: 5
image:
  tag: "1.0.0"
resources:
  memoryLimit: 512Mi
```

```bash
helm install orders-api ./my-chart -f values-production.yaml
```

👉 Same Chart, different values — production runs 5 replicas with more memory, while dev can use `values.yaml`'s defaults with 2 replicas

---

# 🔄 Upgrade and rollback, like real versioning

```bash
helm upgrade orders-api ./my-chart -f values-production.yaml --set image.tag=1.1.0

helm rollback orders-api 1  # revert to revision 1, if something goes wrong
```

👉 Remember the semantic versioning from post 63? Helm keeps a revision history — every `upgrade` is tracked, and reverting to a previous version is a single command, not a manual manifest reconstruction

---

# ⚠️ Common Mistakes

- Putting sensitive values (passwords, keys) directly in a versioned `values.yaml`, instead of using Kubernetes Secrets  
- Creating overly generic templates, making the Chart hard to understand and maintain  
- Not testing the Chart with `helm template` before applying, risking syntax errors straight in production  
- Ignoring `helm lint`, which catches common configuration problems before deploy  

---

# 📌 Conclusion

- Helm packages Kubernetes manifests in a versioned, reusable way, similar to NuGet for C#  
- Parameterized templates let a single Chart serve multiple environments  
- Per-environment `values.yaml` avoids duplicating entire manifests  
- Upgrade and rollback make infrastructure changes trackable and reversible  

👉 With Helm, managing Kubernetes across multiple environments stops being a tangle of duplicated YAMLs and becomes a versioned, repeatable process

---

# 🔥 Next Step

Now that you can package Kubernetes configuration reusably, the next level is:

👉 **C# Fundamentals: Terraform / Infrastructure as Code**

Here you'll learn to version not just the application, but the entire infrastructure around it.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
