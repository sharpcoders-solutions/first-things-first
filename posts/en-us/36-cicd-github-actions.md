# 🧠 C# Fundamentals: CI/CD with GitHub Actions

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How to containerize your API with Docker  
- The entire build, test, and publish flow, done manually  

Manually is exactly the key word for the problem. Running `dotnet test`, `docker build`, and `docker push` by hand, every time, is slow and prone to human error.

👉 **Let's automate all of it with CI/CD**

---

# 💡 What is CI/CD?

👉 **CI (Continuous Integration) = automatically integrating and validating code on every change. CD (Continuous Deployment/Delivery) = automatically publishing that change**

- **CI**: on every push, the pipeline builds the project and runs the automated tests (remember the xUnit post?)  
- **CD**: if everything passes, the pipeline publishes the new version with no manual intervention  

👉 The end goal is the same as the Git Workflow you learned at the very start: reduce risk and increase delivery speed safely

---

# 🏗️ The structure of a GitHub Actions workflow

Workflows live in `.github/workflows/*.yml`:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "8.0.x"

      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore --configuration Release

      - name: Run tests
        run: dotnet test --no-build --configuration Release
```

## 🔹 The main pieces

- `on` → defines when the workflow runs (push, pull request, scheduled)  
- `jobs` → one or more sets of tasks, each running on a clean machine  
- `steps` → the actions executed in sequence inside the job  

👉 This workflow alone already guarantees something powerful: **no broken code reaches `main` without the team noticing** — exactly the problem the automated tests from the previous post were built to solve

---

# 🚀 Adding deployment: building and pushing the Docker image

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    # ... same steps as the CI workflow

  publish-image:
    needs: build-and-test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Log in to the registry
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push the image
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: myregistry/my-api:latest
```

👉 `needs: build-and-test` guarantees the image is only published **if** the tests pass — deployment never happens on top of broken code

## 🔹 Secrets: never credentials in the code

```yaml
username: ${{ secrets.DOCKER_USERNAME }}
```

👉 `secrets` are configured in the GitHub UI (Settings → Secrets), never written directly in the `.yml` — the same caution you already saw in the Docker post, now applied to the pipeline

---

# 🔀 Running on Pull Requests: CI's real value

```yaml
on:
  pull_request:
    branches: [main]
```

👉 With this configuration, every Pull Request runs the pipeline automatically, and the result shows up right on the review screen — exactly the "review + automated test validation" moment you learned about in the Git Workflow post, now actually happening in practice

A PR with a failing pipeline gets visually marked ❌, giving the reviewer a clear signal before they even look at the code.

---

# 📊 The full pipeline, visualized

```
push/PR → checkout → restore → build → test → (if main) build image → push registry → deploy
```

👉 Each arrow represents a step that, without CI/CD, would be a manual step — and every manual step is a chance to forget something or do it differently than last time

---

# ⚠️ Common Mistakes

- Putting credentials directly in the `.yml` file instead of using `secrets`  
- Not running tests before deployment, letting broken code reach production  
- Building giant, slow pipelines without parallelizing independent jobs  
- Ignoring the pipeline's result on Pull Requests, merging anyway even with CI failing  

---

# 📌 Conclusion

- CI automatically validates every change by building and testing the code  
- CD publishes the application with no manual intervention, once CI approves  
- `secrets` keep credentials out of source code  
- Pipelines on Pull Requests give feedback before code review even happens  

👉 With CI/CD, shipping a new version stops being a stressful event and becomes a natural part of the team's workflow

---

# 🔥 Next Step

Now that your application publishes itself, the next level is:

👉 **C# Fundamentals: Structured Logging with Serilog**

Here you'll learn to see what's happening inside your application once it's already live.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
