# 🧠 C# Fundamentals: .NET MAUI — Introduction

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Blazor for building web interfaces with C#  
- This whole track, up to now, has been about back-end and web  

Blazor takes C# to the browser. .NET MAUI takes C# even further: truly native applications for iOS, Android, Windows, and macOS, from a single codebase.

👉 **Let's learn .NET MAUI**

---

# 💡 What is .NET MAUI?

👉 **MAUI = Multi-platform App UI — a single C#/XAML project compiled into native apps across multiple platforms**

```
MAUI Project (C# + XAML)
  ├─ Compiles to iOS (native, via Mono/AOT)
  ├─ Compiles to Android (native, via Mono/AOT)
  ├─ Compiles to Windows (native, via WinUI)
  └─ Compiles to macOS (native, via Mac Catalyst)
```

👉 Unlike Blazor WASM (post 82), which runs inside a browser, MAUI generates truly native applications — full access to camera, GPS, push notifications, sensors

---

# 🏗️ Basic structure of a MAUI page

```xml
<!-- OrdersPage.xaml -->
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui">
    <VerticalStackLayout Padding="20">
        <Label Text="My Orders" FontSize="24" />
        <CollectionView ItemsSource="{Binding Orders}">
            <CollectionView.ItemTemplate>
                <DataTemplate>
                    <Label Text="{Binding Status}" />
                </DataTemplate>
            </CollectionView.ItemTemplate>
        </CollectionView>
        <Button Text="Refresh" Clicked="OnRefreshClicked" />
    </VerticalStackLayout>
</ContentPage>
```

```csharp
public partial class OrdersPage : ContentPage
{
    public OrdersPage()
    {
        InitializeComponent();
        BindingContext = new OrdersViewModel();
    }

    private async void OnRefreshClicked(object sender, EventArgs e)
    {
        await ((OrdersViewModel)BindingContext).LoadOrders();
    }
}
```

👉 XAML declares the interface (similar to Razor in Blazor, or HTML on the web), and C# handles the logic — the same separation of concerns you've already practiced since the Clean Architecture post (33)

---

# 🎯 MVVM: MAUI's architectural pattern

```csharp
public class OrdersViewModel : INotifyPropertyChanged
{
    private ObservableCollection<Order> _orders = new();

    public ObservableCollection<Order> Orders
    {
        get => _orders;
        set
        {
            _orders = value;
            OnPropertyChanged();
        }
    }

    public async Task LoadOrders()
    {
        var result = await _apiClient.GetOrdersAsync();
        Orders = new ObservableCollection<Order>(result);
    }

    public event PropertyChangedEventHandler? PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string? name = null) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

👉 MVVM (Model-View-ViewModel) separates the UI (View, in XAML) from the logic (ViewModel, in C#) — when `Orders` changes, the interface automatically updates via data binding, without you manipulating the UI directly

---

# 🔌 Reusing everything you already know

```csharp
// MauiProgram.cs
var builder = MauiApp.CreateBuilder();

builder.Services.AddHttpClient<ApiClient>(client =>
{
    client.BaseAddress = new Uri("https://myapi.com");
});

builder.Services.AddTransient<OrdersViewModel>();
```

👉 Dependency injection (from the ASP.NET Core post), consuming a REST API (post 31) or GraphQL (post 80) — the data access logic you already master on the back-end works the same inside a MAUI app

---

# ⚖️ MAUI vs Blazor Hybrid vs platform-specific native development

## 🔹 .NET MAUI
- Real native UI (operating system controls)  
- One C#/XAML codebase for every platform  

## 🔹 Blazor Hybrid (MAUI + Blazor)
- Reuses Blazor components (post 82) inside a MAUI app  
- Ideal if you already have a Blazor application and want to bring it to mobile/desktop  

## 🔹 Platform-specific native (Swift, Kotlin)
- Maximum control and platform-specific performance  
- Separate teams per platform, no code sharing  

---

# ⚠️ Common Mistakes

- Writing business logic directly in the View's code-behind, losing the testability MVVM offers  
- Ignoring platform differences (gestures, screen sizes, UI conventions) trying for a 100% identical design everywhere  
- Not using `ObservableCollection` and `INotifyPropertyChanged` correctly, making the UI fail to update automatically  
- Underestimating the compiled app's size — each platform bundles its own runtime  

---

# 📌 Conclusion

- .NET MAUI compiles a single C#/XAML codebase into native apps across multiple platforms  
- MVVM separates UI from logic, with data binding connecting the two automatically  
- Dependency injection and API consumption work the same way as on the back-end  
- MAUI is for truly native apps; Blazor Hybrid reuses existing web components  

👉 With .NET MAUI, C# literally covers the entire stack: database, API, web, and now native mobile and desktop applications too

---

# 🔥 Next Step

Now that you can take C# to mobile and desktop, the next level is:

👉 **C# Fundamentals: Azure Functions**

Here you'll learn to run C# code without managing any server, paying only for execution time.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
