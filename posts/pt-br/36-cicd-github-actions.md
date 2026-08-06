# 🧠 Fundamentos do C#: CI/CD com GitHub Actions

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como containerizar sua API com Docker  
- Todo o fluxo de build, teste e publicação, feito manualmente  

Manualmente é exatamente a palavra-chave do problema. Rodar `dotnet test`, `docker build` e `docker push` na mão, toda vez, é lento e sujeito a erro humano.

👉 **Vamos automatizar tudo isso com CI/CD**

---

# 💡 O que é CI/CD?

👉 **CI (Continuous Integration) = integrar e validar código automaticamente a cada mudança. CD (Continuous Deployment/Delivery) = publicar essa mudança automaticamente**

- **CI**: a cada push, o pipeline compila o projeto e roda os testes automatizados (lembra do post sobre xUnit?)  
- **CD**: se tudo passar, o pipeline publica a nova versão sem intervenção manual  

👉 O objetivo final é o mesmo do Git Workflow que você aprendeu lá no início: reduzir risco e aumentar a velocidade de entrega com segurança

---

# 🏗️ Estrutura de um workflow no GitHub Actions

Workflows vivem em `.github/workflows/*.yml`:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-e-testes:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "8.0.x"

      - name: Restaurar dependências
        run: dotnet restore

      - name: Compilar
        run: dotnet build --no-restore --configuration Release

      - name: Rodar testes
        run: dotnet test --no-build --configuration Release
```

## 🔹 As peças principais

- `on` → define quando o workflow roda (push, pull request, agendado)  
- `jobs` → um ou mais conjuntos de tarefas, cada um rodando em uma máquina limpa  
- `steps` → as ações executadas em sequência dentro do job  

👉 Esse workflow sozinho já garante uma coisa poderosa: **nenhum código quebrado chega à `main` sem que o time perceba**, exatamente o problema que os testes automatizados do post anterior foram feitos para resolver

---

# 🚀 Adicionando o deploy: build e push da imagem Docker

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-e-testes:
    # ... mesmos passos do workflow de CI

  publicar-imagem:
    needs: build-e-testes
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login no registry
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build e push da imagem
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: meurepositorio/minha-api:latest
```

👉 `needs: build-e-testes` garante que a imagem só é publicada **se** os testes passarem — o deploy nunca acontece sobre código quebrado

## 🔹 Secrets: nunca credenciais no código

```yaml
username: ${{ secrets.DOCKER_USERNAME }}
```

👉 `secrets` são configurados na interface do GitHub (Settings → Secrets), nunca escritos direto no `.yml` — o mesmo cuidado que você já viu no post sobre Docker, aplicado agora ao pipeline

---

# 🔀 Rodando em Pull Requests: o verdadeiro valor do CI

```yaml
on:
  pull_request:
    branches: [main]
```

👉 Com essa configuração, todo Pull Request roda o pipeline automaticamente, e o resultado aparece direto na tela de review — exatamente o momento de "revisão + validação por testes automatizados" que você aprendeu no post sobre Git Workflow, só que agora realmente acontecendo

Um PR com o pipeline falhando fica visualmente marcado como ❌, dando ao revisor um sinal claro antes mesmo de olhar o código.

---

# 📊 O pipeline completo, visualizado

```
push/PR → checkout → restore → build → test → (se main) build imagem → push registry → deploy
```

👉 Cada seta representa uma etapa que, sem CI/CD, seria um passo manual — e cada passo manual é uma chance de esquecer algo ou fazer diferente da última vez

---

# ⚠️ Erros comuns

- Colocar credenciais direto no arquivo `.yml` em vez de usar `secrets`  
- Não rodar os testes antes do deploy, permitindo que código quebrado chegue à produção  
- Criar pipelines gigantes e lentos, sem paralelizar jobs independentes  
- Ignorar o resultado do pipeline em Pull Requests, fazendo merge mesmo com o CI falhando  

---

# 📌 Conclusão

- CI valida automaticamente cada mudança, compilando e testando o código  
- CD publica a aplicação sem intervenção manual, depois que o CI aprova  
- `secrets` mantêm credenciais fora do código-fonte  
- Pipelines em Pull Requests dão feedback antes mesmo do code review acontecer  

👉 Com CI/CD, entregar uma nova versão deixa de ser um evento estressante e vira parte natural do fluxo de trabalho do time

---

# 🔥 Próximo passo

Agora que sua aplicação publica sozinha, o próximo nível é:

👉 **Fundamentos do C#: Logging Estruturado com Serilog**

Aqui você vai aprender a enxergar o que está acontecendo dentro da sua aplicação depois que ela já está no ar.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
