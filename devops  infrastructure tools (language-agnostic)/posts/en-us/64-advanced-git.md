# 🧠 C# Fundamentals: Advanced Git

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Git & GitHub: the everyday essential commands  
- Git Workflow: branches, pull requests, and a team's flow  

You already know `commit`, `push`, `pull`, and `merge` (posts 10 and 11). Now it's time to master the commands that separate people who "use" Git from people who truly understand Git.

👉 **Let's learn Advanced Git**

---

# 💡 Interactive rebase: rewriting local history

```bash
git rebase -i HEAD~4
```

```
pick a1b2c3 add email validation
pick d4e5f6 fix typo
pick g7h8i9 add test
pick j0k1l2 fix typo again

# change "pick" to "squash" to combine commits
```

👉 Before opening a Pull Request, `rebase -i` lets you clean up "fix typo" and "wip" commits, turning a messy history into a logical sequence of changes — much easier to review

---

# 🍒 Cherry-pick: bringing over a specific commit

```bash
git cherry-pick a1b2c3d
```

👉 You fixed a critical bug on `main`, but you also need that exact same fix on the `release/2.0` branch, without bringing everything else already merged into `main`. `cherry-pick` applies just that one specific commit

---

# 🔍 Bisect: hunting down the commit that broke everything

```bash
git bisect start
git bisect bad          # the current commit has the bug
git bisect good v1.5.0  # this older version worked fine
```

```
Bisecting: 15 revisions left to test
[commit-hash] checked out this revision for you to test
```

```bash
git bisect good  # or "git bisect bad", depending on the test result
# repeat until Git isolates the exact commit
```

👉 Instead of manually reviewing 30 commits looking for the one that introduced a bug, `bisect` runs an automatic binary search — finds the exact commit in `log2(n)` steps

---

# 🕰️ Reflog: recovering the "unrecoverable"

```bash
git reflog
```

```
a1b2c3d HEAD@{0}: reset: moving to HEAD~3
d4e5f6g HEAD@{1}: commit: add feature X
g7h8i9j HEAD@{2}: commit: fix bug Y
```

```bash
git reset --hard d4e5f6g  # go back to the state before the reset
```

👉 Ran a `git reset --hard` and "lost" commits? `reflog` keeps a history of everything `HEAD` has ever pointed to, even commits that seem to have vanished — in practice, it's very hard to truly lose work in Git

---

# 🌿 Stash: temporarily storing work in progress

```bash
git stash push -m "WIP: refactoring the payment service"
git checkout main
git pull
git checkout my-branch
git stash pop
```

👉 Need to urgently switch branches with uncommitted changes? `stash` temporarily stores the working directory's state, without needing a "dirty" commit just to avoid losing work

---

# ⚠️ Common Mistakes

- Running `rebase` on a shared branch that's already published, rewriting history that others have already pulled — use `rebase` only locally, before sharing  
- Using `git push --force` instead of `--force-with-lease`, unknowingly overwriting someone else's work  
- Forgetting that `cherry-pick` creates a new commit with a different hash, which can cause future conflicts if the original branch gets merged later  
- Not using `bisect` and wasting hours testing commits one by one manually  

---

# 📌 Conclusion

- Interactive rebase cleans up local history before sharing  
- Cherry-pick brings over a specific commit without bringing the whole branch  
- Bisect finds the problem commit with automatic binary search  
- Reflog is the safety net against "losing" commits  

👉 With advanced Git, you stop fearing history manipulation and start using it as an investigation and organization tool

---

# 🔥 Next Step

Now that you've mastered Git in depth, the next level is:

👉 **C# Fundamentals: Trunk-Based Development**

Here you'll learn a branching strategy that's an alternative to GitFlow, used by high-performing teams.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
