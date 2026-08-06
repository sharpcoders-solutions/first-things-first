# 🧠 C# Fundamentals: Docker and Deploying a .NET API

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Clean Architecture applied end to end  
- Authentication and authorization with JWT  

Your API is ready, secure, and well organized — but right now it only runs on your machine. Time to package it so it runs **the same way** everywhere.

👉 **Let's containerize and publish your API with Docker**

---

# 💡 The problem Docker solves

👉 **"It works on my machine" is the problem; Docker is the answer**

Differences in .NET versions, environment variables, OS-level dependencies — all of this can make an application work in one place and fail in another. A **container** packages the application together with everything it needs to run, always the same way.

---

# 🏗️ Creating the `Dockerfile`

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

ENTRYPOINT ["dotnet", "MyApi.dll"]
```

## 🔹 Why two images (multi-stage build)?

- **`build` stage** → uses the **SDK** image (remember the SDK vs Runtime difference?) to compile the project  
- **`runtime` stage** → uses the smaller **Runtime** image, containing only what's needed to execute  

👉 The final result is a lean image, without the build tools that would just waste space in production

---

# 🚀 Building and running the container

```bash
docker build -t my-api .
docker run -p 8080:8080 my-api
```

## 🔹 What each command does

- `docker build` → reads the `Dockerfile` and assembles the image  
- `docker run -p 8080:8080` → spins up a container from the image, mapping the container's port to your machine's port  

👉 Now the same image that runs on your laptop runs, exactly the same, on any server with Docker installed

---

# 🗄️ Connecting dependencies: `docker-compose`

Real applications usually need a database alongside them. `docker-compose` orchestrates multiple containers at once:

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
      - ConnectionStrings__Default=Server=db;Database=MyApiDb;User=sa;Password=Password123!

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=Password123!
    ports:
      - "1433:1433"
```

```bash
docker-compose up
```

👉 A single command spins up the API **and** the database together, already connected — great for local development and for replicating the production environment

---

# 🔐 Configuration via environment variables

```csharp
var connectionString = builder.Configuration.GetConnectionString("Default");
```

👉 Notice that the connection string in `docker-compose.yml` uses `__` (double underscore) to represent the `appsettings.json` hierarchy — ASP.NET Core reads environment variables automatically, with no extra code. This keeps secrets (passwords, JWT keys) out of the source code

---

# ☁️ Publishing to production

The basic flow for getting your image into production:

1. `docker build` → generates the image  
2. `docker push` → sends the image to a registry (Docker Hub, Azure Container Registry, AWS ECR)  
3. The production environment (Azure App Service, AWS ECS, Kubernetes) **pulls** the image from the registry and runs it  

👉 From here, publishing a new version is just generating a new image and repeating the process — no need to manually configure each server

---

# ⚠️ Common Mistakes

- Using the SDK image in production instead of the Runtime image, making the container bigger than it needs to be  
- Putting secrets directly in the `Dockerfile` or in a `docker-compose.yml` that's versioned in Git  
- Forgetting to expose the correct port with `-p`, and not understanding why the application "doesn't respond"  
- Not using a `.dockerignore`, copying `bin/`, `obj/`, and other unnecessary files into the image  

---

# 📌 Conclusion

- Docker packages your API with everything it needs, eliminating "it works on my machine"  
- Multi-stage builds use the SDK to compile and the lighter Runtime to execute  
- `docker-compose` orchestrates the API and database together with a single command  
- Sensitive configuration should come from environment variables, never from source code  

👉 With Docker, your application is ready to run consistently in any environment — from your laptop to a production cluster

---

# 🔥 Next Step

Now that you know how to package your application, the next level is:

👉 **C# Fundamentals: CI/CD with GitHub Actions**

Here you'll automate this entire process, so every code change builds and publishes a new version on its own.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
