# 🧠 Fundamentos do C#: Helm Charts

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Kubernetes e seus manifestos YAML (Pods, Deployments, Services)  
- NuGet (post 63), empacotando e versionando código C# reutilizável  

O post anterior te mostrou manifestos YAML individuais. Em uma aplicação real, você tem dezenas desses arquivos — Deployment, Service, ConfigMap, HPA — todos precisando mudar juntos entre ambientes (dev, staging, produção). Helm resolve isso.

👉 **Vamos aprender Helm Charts**

---

# 💡 O problema de YAMLs espalhados

```
k8s/
  deployment-dev.yaml
  deployment-staging.yaml
  deployment-producao.yaml
  service.yaml
  configmap-dev.yaml
  configmap-producao.yaml
  ...
```

👉 Duplicar manifestos por ambiente é frágil — uma mudança precisa ser replicada manualmente em vários arquivos, com alto risco de divergência

---

# 🏗️ Helm: o "NuGet" do Kubernetes

👉 **Helm Chart = um pacote versionado de manifestos Kubernetes, parametrizável por ambiente**

```
meu-chart/
  Chart.yaml          # metadados, parecido com o .csproj do post 63
  values.yaml          # valores padrão
  templates/
    deployment.yaml
    service.yaml
    configmap.yaml
```

```yaml
# Chart.yaml
apiVersion: v2
name: api-pedidos
version: 1.0.0
appVersion: "1.0.0"
```

👉 Isso lembra diretamente o post sobre NuGet — um Chart tem versão, metadados, e pode ser publicado em um repositório para reuso, do mesmo jeito que uma biblioteca C#

---

# 🎯 Templates: YAML parametrizado

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.nomeAplicacao }}
spec:
  replicas: {{ .Values.replicas }}
  template:
    spec:
      containers:
        - name: {{ .Values.nomeAplicacao }}
          image: "{{ .Values.imagem.repositorio }}:{{ .Values.imagem.tag }}"
          resources:
            limits:
              memory: {{ .Values.recursos.memoriaLimite }}
```

```yaml
# values.yaml (padrões)
nomeAplicacao: api-pedidos
replicas: 2
imagem:
  repositorio: minhaempresa/api-pedidos
  tag: latest
recursos:
  memoriaLimite: 256Mi
```

👉 `{{ .Values.X }}` insere valores dinamicamente — o mesmo template serve para dev, staging e produção, cada um com seu próprio arquivo de valores

---

# 🌍 Valores por ambiente

```yaml
# values-producao.yaml
replicas: 5
imagem:
  tag: "1.0.0"
recursos:
  memoriaLimite: 512Mi
```

```bash
helm install api-pedidos ./meu-chart -f values-producao.yaml
```

👉 O mesmo Chart, valores diferentes — produção roda 5 réplicas com mais memória, enquanto dev pode usar os padrões do `values.yaml` com 2 réplicas

---

# 🔄 Upgrade e rollback, como versionamento de verdade

```bash
helm upgrade api-pedidos ./meu-chart -f values-producao.yaml --set imagem.tag=1.1.0

helm rollback api-pedidos 1  # volta para a revisão 1, se algo der errado
```

👉 Lembra do versionamento semântico do post 63? O Helm mantém histórico de revisões — cada `upgrade` é rastreado, e reverter para uma versão anterior é um comando, não uma reconstrução manual de manifestos

---

# ⚠️ Erros comuns

- Colocar valores sensíveis (senhas, chaves) diretamente no `values.yaml` versionado, em vez de usar Secrets do Kubernetes  
- Criar templates excessivamente genéricos, tornando o Chart difícil de entender e manter  
- Não testar o Chart com `helm template` antes de aplicar, arriscando erros de sintaxe direto em produção  
- Ignorar `helm lint`, que detecta problemas comuns de configuração antes do deploy  

---

# 📌 Conclusão

- Helm empacota manifestos Kubernetes de forma versionada e reutilizável, parecido com NuGet para C#  
- Templates parametrizados permitem um único Chart servir múltiplos ambientes  
- `values.yaml` por ambiente evita duplicação de manifestos completos  
- Upgrade e rollback tornam mudanças de infraestrutura rastreáveis e reversíveis  

👉 Com Helm, gerenciar Kubernetes em múltiplos ambientes deixa de ser um emaranhado de YAMLs duplicados e vira um processo versionado e repetível

---

# 🔥 Próximo passo

Agora que você empacota configurações Kubernetes de forma reutilizável, o próximo nível é:

👉 **Fundamentos do C#: Terraform / Infrastructure as Code**

Aqui você vai aprender a versionar não só a aplicação, mas toda a infraestrutura ao redor dela.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
