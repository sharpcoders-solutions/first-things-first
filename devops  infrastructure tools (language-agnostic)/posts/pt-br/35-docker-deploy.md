# 🧠 Fundamentos do C#: Docker e Deploy de uma API .NET

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Clean Architecture aplicada de ponta a ponta  
- Autenticação e autorização com JWT  

Sua API está pronta, segura e bem organizada — mas hoje ela só roda na sua máquina. Chegou a hora de empacotá-la de um jeito que rode **igual** em qualquer lugar.

👉 **Vamos containerizar e publicar sua API com Docker**

---

# 💡 O problema que o Docker resolve

👉 **"Na minha máquina funciona" é o problema; Docker é a resposta**

Diferenças de versão do .NET, variáveis de ambiente, dependências do sistema operacional — tudo isso pode fazer uma aplicação funcionar em um lugar e falhar em outro. Um **container** empacota a aplicação junto com tudo que ela precisa para rodar, sempre da mesma forma.

---

# 🏗️ Criando o `Dockerfile`

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/out .

ENTRYPOINT ["dotnet", "MinhaApi.dll"]
```

## 🔹 Por que duas imagens (multi-stage build)?

- **Estágio `build`** → usa a imagem do **SDK** (lembra da diferença entre SDK e Runtime?) para compilar o projeto  
- **Estágio `runtime`** → usa a imagem menor do **Runtime**, contendo só o necessário para executar  

👉 O resultado final é uma imagem enxuta, sem as ferramentas de build que só ocupariam espaço à toa em produção

---

# 🚀 Construindo e rodando o container

```bash
docker build -t minha-api .
docker run -p 8080:8080 minha-api
```

## 🔹 O que cada comando faz

- `docker build` → lê o `Dockerfile` e monta a imagem  
- `docker run -p 8080:8080` → sobe um container a partir da imagem, mapeando a porta do container para a porta da sua máquina  

👉 Agora a mesma imagem que roda no seu notebook roda, exatamente igual, em qualquer servidor com Docker instalado

---

# 🗄️ Conectando com dependências: `docker-compose`

Aplicações reais normalmente precisam de um banco de dados junto. O `docker-compose` orquestra múltiplos containers de uma vez:

```yaml
version: "3.8"
services:
  api:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      - ConnectionStrings__Padrao=Server=db;Database=MinhaApiDb;User=sa;Password=Senha123!

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=Senha123!
    ports:
      - "1433:1433"
```

```bash
docker-compose up
```

👉 Um único comando sobe a API **e** o banco de dados juntos, já conectados — ótimo para desenvolvimento local e para replicar o ambiente de produção

---

# 🔐 Configuração por variáveis de ambiente

```csharp
var connectionString = builder.Configuration.GetConnectionString("Padrao");
```

👉 Repare que a connection string no `docker-compose.yml` usa `__` (duplo underscore) para representar a hierarquia do `appsettings.json` — o ASP.NET Core lê variáveis de ambiente automaticamente, sem código extra. Isso evita colocar segredos (senhas, chaves JWT) direto no código-fonte

---

# ☁️ Publicando em produção

O fluxo básico para colocar sua imagem em produção:

1. `docker build` → gera a imagem  
2. `docker push` → envia a imagem para um registry (Docker Hub, Azure Container Registry, AWS ECR)  
3. O ambiente de produção (Azure App Service, AWS ECS, Kubernetes) **puxa** a imagem do registry e a executa  

👉 A partir daqui, publicar uma nova versão é só gerar uma nova imagem e repetir o processo — sem depender de configurar manualmente cada servidor

---

# ⚠️ Erros comuns

- Usar a imagem do SDK em produção em vez da imagem de Runtime, deixando o container maior do que precisa  
- Colocar segredos direto no `Dockerfile` ou no `docker-compose.yml` versionado no Git  
- Esquecer de expor a porta correta com `-p`, e não entender por que a aplicação "não responde"  
- Não usar `.dockerignore`, copiando `bin/`, `obj/` e outros arquivos desnecessários para dentro da imagem  

---

# 📌 Conclusão

- Docker empacota sua API com tudo que ela precisa, eliminando o "na minha máquina funciona"  
- Multi-stage build usa o SDK para compilar e o Runtime, mais leve, para executar  
- `docker-compose` orquestra API e banco de dados juntos, com um único comando  
- Configuração sensível deve vir de variáveis de ambiente, nunca do código-fonte  

👉 Com Docker, sua aplicação está pronta para rodar de forma consistente em qualquer ambiente — do seu notebook até um cluster de produção

---

# 🔥 Próximo passo

Agora que você sabe empacotar sua aplicação, o próximo nível é:

👉 **Fundamentos do C#: CI/CD com GitHub Actions**

Aqui você vai automatizar todo esse processo, para que cada mudança no código gere e publique uma nova versão sozinha.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
