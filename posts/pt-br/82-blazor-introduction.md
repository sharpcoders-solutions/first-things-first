# 🧠 Fundamentos do C#: Blazor — Introdução

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- SignalR para comunicação em tempo real  
- Toda esta trilha assumiu que o front-end é escrito em outra linguagem (JavaScript/TypeScript), e o C# fica só no back-end  

E se você pudesse escrever a interface inteira — HTML dinâmico, interatividade, tudo — usando C#? É isso que o Blazor oferece.

👉 **Vamos aprender Blazor**

---

# 💡 O que é o Blazor?

👉 **Blazor = framework para construir UI web interativa usando C# e Razor, no lugar de JavaScript**

```razor
@page "/contador"

<h1>Contador: @contador</h1>
<button @onclick="Incrementar">Clique aqui</button>

@code {
    private int contador = 0;

    private void Incrementar()
    {
        contador++;
    }
}
```

👉 Sintaxe Razor (a mesma família de sintaxe do ASP.NET Core MVC clássico), mas com interatividade completa — clicar no botão executa código C#, não JavaScript

---

# 🏗️ Os dois modelos de hospedagem

## 🔹 Blazor Server

```
Navegador ←── SignalR (post 81) ──→ Servidor executa o C#
```

👉 O código C# roda no servidor — cada interação do usuário vira uma mensagem via SignalR, e o servidor devolve as mudanças de DOM. Início rápido, mas depende de conexão constante

## 🔹 Blazor WebAssembly (WASM)

```
Navegador executa .NET compilado para WebAssembly, direto no cliente
```

👉 O runtime .NET inteiro roda **dentro do navegador**, via WebAssembly — sem depender do servidor após o carregamento inicial, funciona até offline

---

# 🎯 Componentes: a unidade fundamental

```razor
<!-- CartaoPedido.razor -->
<div class="cartao">
    <h3>Pedido #@Pedido.Id</h3>
    <p>Status: @Pedido.Status</p>
    <button @onclick="() => OnConfirmar.InvokeAsync(Pedido.Id)">Confirmar</button>
</div>

@code {
    [Parameter]
    public Pedido Pedido { get; set; } = default!;

    [Parameter]
    public EventCallback<int> OnConfirmar { get; set; }
}
```

```razor
<!-- Uso em outra página -->
<CartaoPedido Pedido="@meuPedido" OnConfirmar="@TratarConfirmacao" />
```

👉 Isso lembra o post sobre classes e objetos (20) — componentes são unidades reutilizáveis com estado próprio (`[Parameter]`), do mesmo jeito que React ou Angular funcionam, mas inteiramente em C#

---

# 🔌 Injeção de dependência, do jeito que você já conhece

```razor
@page "/pedidos"
@inject HttpClient Http

<h3>Meus Pedidos</h3>

@if (pedidos is null)
{
    <p>Carregando...</p>
}
else
{
    @foreach (var pedido in pedidos)
    {
        <CartaoPedido Pedido="@pedido" />
    }
}

@code {
    private List<Pedido>? pedidos;

    protected override async Task OnInitializedAsync()
    {
        pedidos = await Http.GetFromJsonAsync<List<Pedido>>("api/pedidos");
    }
}
```

👉 `@inject` usa o mesmo container de DI de sempre (post sobre ASP.NET Core) — e `OnInitializedAsync` é o ciclo de vida do componente, parecido com o `ExecuteAsync` do `BackgroundService` (post 52)

---

# ⚖️ Quando escolher Blazor

- **Blazor Server**: dashboards internos, aplicações corporativas com boa conectividade, onde iniciar rápido importa mais que reduzir carga no servidor  
- **Blazor WASM**: aplicações públicas que precisam funcionar offline ou reduzir carga no servidor, aceitando um carregamento inicial um pouco maior  
- **Continuar com React/Angular + API REST/GraphQL**: quando o time já tem expertise forte em JavaScript, ou a aplicação precisa de um ecossistema de bibliotecas JS específico  

---

# ⚠️ Erros comuns

- Escolher Blazor Server para aplicações públicas de alto tráfego sem planejar a escala de conexões SignalR simultâneas  
- Ignorar o tamanho do payload inicial do Blazor WASM (o runtime .NET inteiro precisa ser baixado) em conexões lentas  
- Misturar lógica de UI e lógica de negócio dentro do `@code`, sem extrair para services injetáveis  
- Adotar Blazor pressupondo que substitui completamente o conhecimento de HTML/CSS — o C# substitui o JavaScript, não o resto do front-end  

---

# 📌 Conclusão

- Blazor permite construir UI interativa inteiramente em C#  
- Blazor Server roda no servidor via SignalR; Blazor WASM roda direto no navegador via WebAssembly  
- Componentes são reutilizáveis, com parâmetros e eventos, parecido com frameworks JS modernos  
- A mesma injeção de dependência do ASP.NET Core funciona nativamente nos componentes  

👉 Com Blazor, o C# deixa de ser só a linguagem do back-end e se torna capaz de cobrir a stack inteira, do banco de dados até o clique do usuário

---

# 🔥 Próximo passo

Agora que você sabe construir interfaces web com C#, o próximo nível é:

👉 **Fundamentos do C#: .NET MAUI — Introdução**

Aqui você vai aprender a levar C# além da web, construindo aplicativos nativos para mobile e desktop.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
