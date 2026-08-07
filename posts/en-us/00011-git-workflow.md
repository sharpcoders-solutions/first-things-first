# 🧠 Git Workflow: Branches, Pull Requests, and a Team's Flow

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How computers work  
- How programs execute  
- Threads and concurrency  
- How to use IDEs  
- What Git and GitHub are  

By now you know how to **commit** and **push code**. But that's just the basics.

👉 **The next level is understanding how a professional team actually works with Git**

---

# 💡 Why not work directly on `main`?

`main` (or `master`, in older projects) represents stable code — what's in production or ready to go.

👉 Editing it directly is risky: any mistake affects everyone immediately

That's why the basic workflow of any team follows four steps:

1. Create a **branch** from `main`  
2. Work on it in isolation  
3. Open a **pull request** when done  
4. Review, adjust, and only then merge back  

👉 This isolation lets multiple people work on the same project without stepping on each other's work

---

# 🌱 Branches: your parallel timeline

A branch is, in practice, an isolated copy of the code where you can work without affecting anyone else.

```bash
git checkout -b feature/user-signup
```

## 🔹 Branch best practices

- **Name with intent** — `feature/user-signup`, `fix/login-bug`, `chore/update-dependencies`  
- **Keep it small** — the smaller the scope, the faster it gets reviewed  
- **Sync often** — bring changes from `main` in early to avoid big divergences  

👉 Small, up-to-date branches mean fewer conflicts and faster reviews

---

# 🔀 What a Pull Request really is

A pull request (PR) isn't just "merging code." It's the moment your work:

✅ Becomes **visible** to the team  
✅ Goes through **review** by someone else  
✅ Gets validated by **automated tests** (CI)  
✅ Generates a documented **technical discussion**  

## 🔹 What makes a good PR

- A clear title and description of **what** changed and **why**  
- A lean scope — a giant PR is hard to review well  
- A link to the related task/issue  
- Screenshots or examples when it involves visible behavior  

👉 A well-written PR is a gift to whoever reviews it

---

# 👀 Code Review isn't about finding fault

Whoever reviews a PR isn't hunting for someone to blame — they're protecting the quality of what goes to production.

## 🔹 For the reviewer

- Comment on the code, not the person  
- Suggest, don't just point out a problem  
- Approve once it's good enough — it doesn't need to be perfect  

## 🔹 For the person being reviewed

- Don't take it personally  
- A reviewer's questions are also a learning opportunity  
- Disagree with an argument, not with automatic defensiveness  

---

# ⚔️ Handling merge conflicts

A conflict happens when two changes touch the same line of code in different ways.

👉 It's not an error — it's just Git telling you it needs your decision

```bash
git status          # shows the files in conflict
# manually resolve the sections marked with <<<<<<<, =======, >>>>>>>
git add .
git commit
```

👉 The longer a branch goes without syncing with `main`, the bigger the conflict tends to be

---

# 🏗️ Common workflows

There's no single "right way." A few workflows dominate the industry:

### ✅ GitHub Flow
- Short branches from `main`  
- PR, review, merge  
- Simple and direct — great for continuous deployment  

### ✅ Git Flow
- Specific branches: `develop`, `release`, `hotfix`  
- More structured — common in well-defined release cycles  

### ✅ Trunk-Based Development
- Small, frequent commits directly (or almost directly) into `main`  
- Requires strong automated tests and feature flags  

👉 What matters isn't memorizing the workflow's name, but understanding **why** the team chose that model

---

# ⚠️ Common Mistakes

- Letting a branch live for weeks without syncing  
- Opening huge PRs that are hard to review  
- Merging without review  
- Writing vague or empty PR descriptions  

---

# 📌 Conclusion

- Branch = safe, isolated work  
- Pull Request = visibility, review, and technical discussion  
- Code review = team quality and standards, not fault-finding  
- A well-defined flow = fewer conflicts, more predictability  

👉 This is how professional teams keep code stable, reviewed, and traceable

---

# 🔥 Next Step

Now that you know how to collaborate like a team, the next level is:

👉 **C# Fundamentals: .NET Architecture**

Here you'll understand what happens behind your code: IL, the CLR, and the structure of the .NET Frameworks.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
