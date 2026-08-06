# 🧠 C# Fundamentals: Terraform / Infrastructure as Code

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Helm for packaging Kubernetes configuration  
- CI/CD with GitHub Actions (post 36), automating the application's build and deploy  

Helm versions what runs **inside** Kubernetes. But who creates the Kubernetes cluster itself? The managed database? The network? That too can — and should — be versioned code, not manual clicks in a cloud portal.

👉 **Let's learn Terraform and Infrastructure as Code**

---

# 💡 The problem with manually created infrastructure

```
❌ "Go into the Azure portal, create an App Service, configure X, Y, Z..."
```

👉 Manual steps aren't versioned, aren't reviewable in a Pull Request, and are impossible to reproduce identically in another environment — the same problem that motivated Git (post 10) for code, but applied to infrastructure

---

# 🏗️ Terraform: infrastructure as code

```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-orders-api"
  location = "eastus"
}

resource "azurerm_service_plan" "plan" {
  name                = "plan-orders-api"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  os_type             = "Linux"
  sku_name            = "B1"
}

resource "azurerm_linux_web_app" "api" {
  name                = "orders-api"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_service_plan.plan.location
  service_plan_id     = azurerm_service_plan.plan.id

  site_config {
    application_stack {
      dotnet_version = "9.0"
    }
  }
}
```

👉 All the infrastructure — resource group, hosting plan, web application — is described in `.tf` files, versionable in Git like any C# code

---

# 🎯 The flow: plan and apply

```bash
terraform init    # downloads the required providers (Azure, AWS, etc.)
terraform plan    # shows what WILL change, without applying yet
terraform apply   # actually applies the changes
```

```
Terraform will perform the following actions:
  + create azurerm_linux_web_app.api
  ~ update azurerm_service_plan.plan (sku_name: "B1" → "S1")

Plan: 1 to add, 1 to change, 0 to destroy.
```

👉 `terraform plan` works like a `git diff` for infrastructure — you see exactly what will change before applying, with no surprises

---

# 🔄 State: the source of truth

```
terraform.tfstate → file that tracks what's already been created
```

👉 Terraform keeps a state file that maps the code to the real resources in the cloud — if someone changes something manually in the portal, the next `plan` detects that divergence (called "drift")

---

# 🔌 Connecting with everything you've already learned

```hcl
resource "azurerm_container_app" "api" {
  name = "orders-api"
  # ...

  template {
    container {
      name  = "orders-api"
      image = "mycompany.azurecr.io/orders-api:${var.image_tag}" # post 35
    }
  }
}
```

```yaml
# .github/workflows/deploy.yml (post 36)
- name: Terraform Apply
  run: terraform apply -auto-approve -var="image_tag=${{ github.sha }}"
```

👉 The same CI/CD pipeline from post 36 can run `terraform apply` automatically, applying infrastructure alongside the application deploy — infrastructure and code evolving together, in the same flow

---

# ⚠️ Common Mistakes

- Manually editing resources in the cloud portal after managing them with Terraform, causing "drift" between the real state and the code  
- Not using a remote backend for the state file (`terraform.tfstate`), risking state loss or conflicts between team members  
- Running `terraform apply` directly in production without reviewing the `plan` first  
- Mixing secrets directly into versioned `.tf` code, instead of using secure variables or a secrets vault  

---

# 📌 Conclusion

- Infrastructure as Code versions infrastructure the same way we version C# code  
- Terraform describes cloud resources in declarative `.tf` files  
- `plan` shows changes before applying; `apply` actually executes them  
- The state file tracks the real infrastructure, detecting manual divergences  

👉 With Terraform, creating an entire environment stops depending on tribal knowledge and manual clicks, and becomes a repeatable, reviewable, versioned process

---

# 🔥 Next Step

Now that you version infrastructure as code, the next level is:

👉 **C# Fundamentals: Blue-Green and Canary Deployment**

Here you'll learn deployment strategies that reduce the risk of putting a new version into production.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
