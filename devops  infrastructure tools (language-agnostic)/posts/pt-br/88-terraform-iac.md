# 🧠 Fundamentos do C#: Terraform / Infrastructure as Code

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Helm para empacotar configuração Kubernetes  
- CI/CD com GitHub Actions (post 36), automatizando build e deploy da aplicação  

Helm versiona o que roda **dentro** do Kubernetes. Mas quem cria o próprio cluster Kubernetes? O banco de dados gerenciado? A rede? Isso também pode — e deve — ser código versionado, não cliques manuais em um painel de nuvem.

👉 **Vamos aprender Terraform e Infrastructure as Code**

---

# 💡 O problema da infraestrutura criada manualmente

```
❌ "Entra no portal da Azure, cria um App Service, configura X, Y, Z..."
```

👉 Passos manuais não são versionados, não são revisáveis em Pull Request, e são impossíveis de reproduzir de forma idêntica em outro ambiente — o mesmo problema que motivou o Git (post 10) para código, mas aplicado à infraestrutura

---

# 🏗️ Terraform: infraestrutura como código

```hcl
resource "azurerm_resource_group" "principal" {
  name     = "rg-api-pedidos"
  location = "brazilsouth"
}

resource "azurerm_service_plan" "plano" {
  name                = "plano-api-pedidos"
  resource_group_name = azurerm_resource_group.principal.name
  location            = azurerm_resource_group.principal.location
  os_type             = "Linux"
  sku_name            = "B1"
}

resource "azurerm_linux_web_app" "api" {
  name                = "api-pedidos"
  resource_group_name = azurerm_resource_group.principal.name
  location            = azurerm_service_plan.plano.location
  service_plan_id     = azurerm_service_plan.plano.id

  site_config {
    application_stack {
      dotnet_version = "9.0"
    }
  }
}
```

👉 Toda a infraestrutura — grupo de recursos, plano de hospedagem, aplicação web — está descrita em arquivos `.tf`, versionáveis no Git como qualquer código C#

---

# 🎯 O fluxo: plan e apply

```bash
terraform init    # baixa os providers necessários (Azure, AWS, etc.)
terraform plan    # mostra o que VAI mudar, sem aplicar ainda
terraform apply   # aplica as mudanças de verdade
```

```
Terraform will perform the following actions:
  + criar azurerm_linux_web_app.api
  ~ atualizar azurerm_service_plan.plano (sku_name: "B1" → "S1")

Plan: 1 to add, 1 to change, 0 to destroy.
```

👉 `terraform plan` funciona como um `git diff` para infraestrutura — você vê exatamente o que vai mudar antes de aplicar, sem surpresas

---

# 🔄 Estado: a fonte da verdade

```
terraform.tfstate → arquivo que rastreia o que já foi criado
```

👉 O Terraform mantém um arquivo de estado que mapeia o código para os recursos reais na nuvem — se alguém mudar algo manualmente no portal, o próximo `plan` detecta essa divergência (chamada de "drift")

---

# 🔌 Conectando com tudo que você já aprendeu

```hcl
resource "azurerm_container_app" "api" {
  name = "api-pedidos"
  # ...

  template {
    container {
      name  = "api-pedidos"
      image = "minhaempresa.azurecr.io/api-pedidos:${var.tag_imagem}" # post 35
    }
  }
}
```

```yaml
# .github/workflows/deploy.yml (post 36)
- name: Terraform Apply
  run: terraform apply -auto-approve -var="tag_imagem=${{ github.sha }}"
```

👉 O mesmo pipeline de CI/CD do post 36 pode rodar `terraform apply` automaticamente, aplicando infraestrutura junto com o deploy da aplicação — infraestrutura e código evoluindo juntos, no mesmo fluxo

---

# ⚠️ Erros comuns

- Editar recursos manualmente no portal da nuvem depois de gerenciá-los com Terraform, causando "drift" entre o estado real e o código  
- Não usar backend remoto para o arquivo de estado (`terraform.tfstate`), arriscando perda de estado ou conflitos entre membros do time  
- Rodar `terraform apply` direto em produção sem revisar o `plan` primeiro  
- Misturar segredos diretamente no código `.tf` versionado, em vez de usar variáveis seguras ou um cofre de segredos  

---

# 📌 Conclusão

- Infrastructure as Code versiona a infraestrutura da mesma forma que versionamos código C#  
- Terraform descreve recursos de nuvem em arquivos declarativos `.tf`  
- `plan` mostra mudanças antes de aplicar; `apply` executa de fato  
- O arquivo de estado rastreia a infraestrutura real, detectando divergências manuais  

👉 Com Terraform, criar um ambiente inteiro deixa de depender de conhecimento tribal e cliques manuais, e vira um processo repetível, revisável e versionado

---

# 🔥 Próximo passo

Agora que você versiona infraestrutura como código, o próximo nível é:

👉 **Fundamentos do C#: Deploy Blue-Green e Canary**

Aqui você vai aprender estratégias de deploy que reduzem o risco de colocar uma nova versão em produção.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
