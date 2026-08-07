# 🧠 Fundamentos do C#: .NET MAUI — Introdução

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Blazor para construir interfaces web com C#  
- Toda essa trilha, até aqui, foi sobre back-end e web  

Blazor leva C# para o navegador. O .NET MAUI leva C# ainda mais longe: aplicativos nativos de verdade para iOS, Android, Windows e macOS, a partir de uma única base de código.

👉 **Vamos aprender .NET MAUI**

---

# 💡 O que é o .NET MAUI?

👉 **MAUI = Multi-platform App UI — um único projeto C#/XAML compilado para apps nativos em múltiplas plataformas**

```
Projeto MAUI (C# + XAML)
  ├─ Compila para iOS (nativo, via Mono/AOT)
  ├─ Compila para Android (nativo, via Mono/AOT)
  ├─ Compila para Windows (nativo, via WinUI)
  └─ Compila para macOS (nativo, via Mac Catalyst)
```

👉 Diferente do Blazor WASM (post 85), que roda dentro de um navegador, o MAUI gera aplicativos verdadeiramente nativos — acesso total a câmera, GPS, notificações push, sensores

---

# 🏗️ Estrutura básica de uma página MAUI

```xml
<!-- PaginaPedidos.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui">
    <VerticalStackLayout Padding="20">
        <Label Text="Meus Pedidos" FontSize="24" />
        <CollectionView ItemsSource="{Binding Pedidos}">
            <CollectionView.ItemTemplate>
                <DataTemplate>
                    <Label Text="{Binding Status}" />
                </DataTemplate>
            </CollectionView.ItemTemplate>
        </CollectionView>
        <Button Text="Atualizar" Clicked="OnAtualizarClicado" />
    </VerticalStackLayout>
</ContentPage>
```

```csharp
public partial class PaginaPedidos : ContentPage
{
    public PaginaPedidos()
    {
        InitializeComponent();
        BindingContext = new PedidosViewModel();
    }

    private async void OnAtualizarClicado(object sender, EventArgs e)
    {
        await ((PedidosViewModel)BindingContext).CarregarPedidos();
    }
}
```

👉 XAML declara a interface (parecido com Razor no Blazor, ou HTML na web), e o C# lida com a lógica — a mesma separação de responsabilidades que você já pratica desde o post sobre SOLID (28)

---

# 🎯 MVVM: o padrão arquitetural do MAUI

```csharp
public class PedidosViewModel : INotifyPropertyChanged
{
    private ObservableCollection<Pedido> _pedidos = new();

    public ObservableCollection<Pedido> Pedidos
    {
        get => _pedidos;
        set
        {
            _pedidos = value;
            OnPropertyChanged();
        }
    }

    public async Task CarregarPedidos()
    {
        var resultado = await _apiClient.ObterPedidosAsync();
        Pedidos = new ObservableCollection<Pedido>(resultado);
    }

    public event PropertyChangedEventHandler? PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string? nome = null) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nome));
}
```

👉 MVVM (Model-View-ViewModel) separa a UI (View, em XAML) da lógica (ViewModel, em C#) — quando `Pedidos` muda, a interface atualiza automaticamente via data binding, sem você manipular a UI diretamente

---

# 🔌 Reaproveitando tudo que você já sabe

```csharp
// MauiProgram.cs
var builder = MauiApp.CreateBuilder();

builder.Services.AddHttpClient<ApiClient>(client =>
{
    client.BaseAddress = new Uri("https://minhaapi.com");
});

builder.Services.AddTransient<PedidosViewModel>();
```

👉 Injeção de dependência (post sobre ASP.NET Core), consumo de API REST (post 34) ou GraphQL — a lógica de acesso a dados que você já domina no back-end funciona igual dentro de um app MAUI

---

# ⚖️ MAUI vs Blazor Hybrid vs desenvolvimento nativo específico

## 🔹 .NET MAUI
- UI nativa real (controles do sistema operacional)  
- Uma base de código C#/XAML para todas as plataformas  

## 🔹 Blazor Hybrid (MAUI + Blazor)
- Reutiliza componentes Blazor (post 85) dentro de um app MAUI  
- Ideal se você já tem uma aplicação Blazor e quer levá-la para mobile/desktop  

## 🔹 Nativo específico (Swift, Kotlin)
- Máximo controle e performance específicos da plataforma  
- Times separados por plataforma, sem compartilhamento de código  

---

# ⚠️ Erros comuns

- Escrever lógica de negócio diretamente no code-behind da View, perdendo a testabilidade que o MVVM oferece  
- Ignorar diferenças de plataforma (gestos, tamanhos de tela, convenções de UI) tentando um design 100% idêntico em todo lugar  
- Não usar `ObservableCollection` e `INotifyPropertyChanged` corretamente, fazendo a UI não atualizar automaticamente  
- Subestimar o tamanho do app compilado — cada plataforma embala seu próprio runtime  

---

# 📌 Conclusão

- .NET MAUI compila uma única base de código C#/XAML para apps nativos em múltiplas plataformas  
- MVVM separa UI de lógica, com data binding conectando os dois automaticamente  
- Injeção de dependência e consumo de API funcionam do mesmo jeito que no back-end  
- MAUI é para apps verdadeiramente nativos; Blazor Hybrid reaproveita componentes web existentes  

👉 Com .NET MAUI, o C# cobre literalmente toda a stack: banco de dados, API, web e agora também aplicativos nativos mobile e desktop

---

# 🔥 Próximo passo

Agora que você leva C# para mobile e desktop, o próximo nível é:

👉 **Fundamentos do C#: Azure Functions**

Aqui você vai aprender a rodar código C# sem gerenciar servidor nenhum, pagando só pelo tempo de execução.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
