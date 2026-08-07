# 🧠 C# Fundamentals: Blazor — Introduction

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- SignalR for real-time communication  
- This entire track has assumed the front-end is written in another language (JavaScript/TypeScript), with C# staying purely on the back-end  

What if you could write the entire interface — dynamic HTML, interactivity, all of it — using C#? That's what Blazor offers.

👉 **Let's learn Blazor**

---

# 💡 What is Blazor?

👉 **Blazor = a framework for building interactive web UI using C# and Razor, instead of JavaScript**

```razor
@page "/counter"

<h1>Counter: @count</h1>
<button @onclick="Increment">Click here</button>

@code {
    private int count = 0;

    private void Increment()
    {
        count++;
    }
}
```

👉 Razor syntax (the same syntax family as classic ASP.NET Core MVC), but with full interactivity — clicking the button runs C# code, not JavaScript

---

# 🏗️ The two hosting models

## 🔹 Blazor Server

```
Browser ←── SignalR (post 84) ──→ Server runs the C#
```

👉 The C# code runs on the server — every user interaction becomes a message via SignalR, and the server sends back DOM changes. Fast startup, but depends on a constant connection

## 🔹 Blazor WebAssembly (WASM)

```
Browser runs .NET compiled to WebAssembly, directly on the client
```

👉 The entire .NET runtime runs **inside the browser**, via WebAssembly — no server dependency after the initial load, works even offline

---

# 🎯 Components: the fundamental unit

```razor
<!-- OrderCard.razor -->
<div class="card">
    <h3>Order #@Order.Id</h3>
    <p>Status: @Order.Status</p>
    <button @onclick="() => OnConfirm.InvokeAsync(Order.Id)">Confirm</button>
</div>

@code {
    [Parameter]
    public Order Order { get; set; } = default!;

    [Parameter]
    public EventCallback<int> OnConfirm { get; set; }
}
```

```razor
<!-- Usage in another page -->
<OrderCard Order="@myOrder" OnConfirm="@HandleConfirmation" />
```

👉 This is reminiscent of the classes and objects post (20) — components are reusable units with their own state (`[Parameter]`), the same way React or Angular work, but entirely in C#

---

# 🔌 Dependency injection, the way you already know it

```razor
@page "/orders"
@inject HttpClient Http

<h3>My Orders</h3>

@if (orders is null)
{
    <p>Loading...</p>
}
else
{
    @foreach (var order in orders)
    {
        <OrderCard Order="@order" />
    }
}

@code {
    private List<Order>? orders;

    protected override async Task OnInitializedAsync()
    {
        orders = await Http.GetFromJsonAsync<List<Order>>("api/orders");
    }
}
```

👉 `@inject` uses the same familiar DI container (from the ASP.NET Core post) — and `OnInitializedAsync` is the component's lifecycle hook, similar to `BackgroundService`'s `ExecuteAsync` (post 55)

---

# ⚖️ When to choose Blazor

- **Blazor Server**: internal dashboards, corporate applications with good connectivity, where fast startup matters more than reducing server load  
- **Blazor WASM**: public applications that need to work offline or reduce server load, accepting a slightly larger initial load  
- **Sticking with React/Angular + REST/GraphQL API**: when the team already has strong JavaScript expertise, or the application needs a specific JS library ecosystem  

---

# ⚠️ Common Mistakes

- Choosing Blazor Server for high-traffic public applications without planning for simultaneous SignalR connection scale  
- Ignoring Blazor WASM's initial payload size (the entire .NET runtime needs to be downloaded) on slow connections  
- Mixing UI logic and business logic inside `@code`, without extracting it into injectable services  
- Adopting Blazor assuming it completely replaces the need for HTML/CSS knowledge — C# replaces JavaScript, not the rest of the front-end  

---

# 📌 Conclusion

- Blazor lets you build interactive UI entirely in C#  
- Blazor Server runs on the server via SignalR; Blazor WASM runs directly in the browser via WebAssembly  
- Components are reusable, with parameters and events, similar to modern JS frameworks  
- The same ASP.NET Core dependency injection works natively in components  

👉 With Blazor, C# stops being just the back-end language and becomes capable of covering the entire stack, from the database to the user's click

---

# 🔥 Next Step

Now that you know how to build web interfaces with C#, the next level is:

👉 **C# Fundamentals: .NET MAUI — Introduction**

Here you'll learn to take C# beyond the web, building native apps for mobile and desktop.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
