# 🧠 C# Fundamentals: Mentoring and Technical Leadership

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Refactoring legacy code  
- Practically the entire technical track, from the first Hello World (post 14) to distributed architecture  

You now command an enormous amount of C# and .NET. But the career post (50) already pointed at this: at some point, a senior developer's greatest impact stops coming just from the code they write, and starts coming from the code they help others write better.

👉 **Let's learn Mentoring and Technical Leadership**

---

# 💡 Code Review as a mentoring tool, not policing

```
❌ "This is wrong. Change it to a switch expression."

✅ "You could simplify this with a switch expression (remember post 27?) — 
   it's more direct and avoids this nested if/else. What do you think?"
```

👉 Remember the Git Workflow post (11)? The Pull Request is the most frequent contact point between developers on a team — how you give feedback there teaches (or alienates) as much as any formal mentoring session

---

# 🏗️ Pairing: transferring knowledge in real time

```csharp
// Senior and junior in the same code, live
public class OrderService
{
    // "Why would you make this Task<Order> instead of Order?"
    // "Because we're calling the database, remember the async/await post 26?"
}
```

👉 Pair programming isn't about the senior dictating the code — it's about thinking out loud, explaining the "why" behind every decision, something a PR review alone rarely captures

---

# 🎯 Teaching through questions, not ready-made answers

```
❌ "Use the Repository Pattern here."

✅ "What happens if tomorrow you need to swap EF Core for a different 
   ORM? How coupled would the code be to it right now?"
   (let the person arrive at the Repository Pattern — post 29 — on their own)
```

👉 Giving the ready-made answer solves the immediate problem. Asking the right question develops the ability to solve the **next** problem alone — the goal of mentoring is to become dispensable, not indispensable

---

# 🔍 Creating team patterns, not just correcting individuals

```csharp
// Remember post 62, Roslyn Analyzers?
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public class ForbidAsyncVoidAnalyzer : DiagnosticAnalyzer
{
    // Instead of fixing "async void" person by person in every PR,
    // the analyzer applies the pattern automatically for the whole team
}
```

👉 Effective technical leadership doesn't scale through repeated manual corrections — it scales through tools (analyzers, post 62), documentation (post 49), and processes (Trunk-Based Development, post 65) that enforce the right pattern by default, not by individual memory

---

# 🌱 Delegating with support, not with fear

```
❌ "Let me handle this critical part, it's too risky for you."

✅ "This part is critical, so let's pair on it — you write,
   I'll review closely, and we'll document the decisions (remember 
   the author's note we write on every post in this track?)"
```

👉 Holding onto every important decision creates a bottleneck — you. Delegating with support (not delegating and disappearing) grows the whole team's capacity, and frees up room for you to grow to the next level too

---

# ⚠️ Common Mistakes

- Always giving the ready-made answer, creating dependency instead of autonomy  
- Doing code review focused only on pointing out mistakes, without acknowledging what was done well  
- Confusing technical leadership with micromanagement — dictating every implementation decision  
- Not documenting architectural decisions (remember the implicit ADRs in posts like Clean Architecture, 33?), forcing every new person to rediscover the "why" through trial and error  

---

# 📌 Conclusion

- Code review is a teaching tool, not just quality control  
- Good questions develop autonomy; ready-made answers create dependency  
- Team patterns scale through tools and processes, not repeated manual correction  
- Delegating with support grows the whole team, without creating a bottleneck in a single person  

👉 With mentoring and technical leadership, your impact stops being limited to code you write personally, and starts multiplying through the whole team around you

---

# 🔥 Next Step

Now that you multiply knowledge within the team, the next level is:

👉 **C# Fundamentals: Contributing to Open Source**

Here you'll learn to multiply that same impact beyond your company, contributing to the .NET ecosystem you use every day.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
