# 🧠 Fundamentos do C#: System.Text.Json Avançado

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `GeneratedRegex` e como source generators eliminam custo de runtime  
- `System.Text.Json` básico, usado implicitamente desde o primeiro post sobre APIs  

Toda API que você construiu nesta trilha serializa e desserializa JSON automaticamente, sem você pensar muito nisso. Mas `System.Text.Json` tem uma camada avançada — conversores customizados e um source generator próprio — que resolve problemas reais de controle e performance.

👉 **Vamos explorar `System.Text.Json` avançado**

---

# 💡 Revisão: o básico que você já usa sem perceber

```csharp
public record Produto(int Id, string Nome, decimal Preco);

string json = JsonSerializer.Serialize(new Produto(1, "Notebook", 3500m));
var produto = JsonSerializer.Deserialize<Produto>(json);
```

👉 O ASP.NET Core já usa `System.Text.Json` por baixo dos panos em todo `[HttpPost]`/`[HttpGet]` que você escreveu — o `record` (post 27) mapeia naturalmente para JSON, campo por campo

---

# 🎨 Conversores customizados: controlando a serialização manualmente

```csharp
public class DinheiroConversor : JsonConverter<decimal>
{
    public override decimal Read(ref Utf8JsonReader reader, Type tipoParaConverter, JsonSerializerOptions opcoes)
    {
        var texto = reader.GetString();
        return decimal.Parse(texto!.Replace("$", "").Replace(",", ""), CultureInfo.InvariantCulture);
    }

    public override void Write(Utf8JsonWriter writer, decimal valor, JsonSerializerOptions opcoes)
    {
        writer.WriteStringValue($"${valor:N2}");
    }
}
```

```csharp
public record Produto(int Id, string Nome, [property: JsonConverter(typeof(DinheiroConversor))] decimal Preco);

// Serializa Preco como "$3,500.00" em vez de 3500.00
```

👉 **`JsonConverter<T>` = controle total sobre como um tipo específico é lido e escrito em JSON**, essencial quando o formato padrão não bate com o que uma API externa espera, ou quando você precisa de uma representação customizada (como formatar dinheiro com símbolo de moeda)

---

# 🔧 Configurando opções globais de serialização

```csharp
var opcoes = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    WriteIndented = true,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};

string json = JsonSerializer.Serialize(produto, opcoes);
```

```csharp
// Program.cs — aplicando globalmente na API
builder.Services.Configure<Microsoft.AspNetCore.Http.Json.JsonOptions>(opcoes =>
{
    opcoes.SerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
});
```

👉 `PropertyNamingPolicy` resolve a diferença comum entre convenção C# (`PascalCase`) e convenção JavaScript (`camelCase`), sem precisar renomear nenhuma propriedade das suas classes C#

---

# ⚡ `JsonSerializerContext`: source generator de serialização

```csharp
[JsonSerializable(typeof(Produto))]
[JsonSerializable(typeof(List<Produto>))]
public partial class ContextoJson : JsonSerializerContext { }
```

```csharp
string json = JsonSerializer.Serialize(produto, ContextoJson.Default.Produto);
var produtoLido = JsonSerializer.Deserialize(json, ContextoJson.Default.Produto);
```

👉 **`JsonSerializerContext` = um source generator (lembra do post 68 e do `GeneratedRegex`?) que gera, em tempo de compilação, todo o código de serialização para os tipos declarados**

Sem esse recurso, `System.Text.Json` usa **reflection** (post 67) para descobrir propriedades em tempo de execução — funciona, mas tem custo de performance e, mais importante, **não funciona com Native AOT** (post 69), porque reflection sobre tipos que não existem mais como metadata simplesmente falha

---

# 🚀 Por que isso importa: Native AOT exige o source generator

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
  <JsonSerializerIsReflectionEnabledByDefault>false</JsonSerializerIsReflectionEnabledByDefault>
</PropertyGroup>
```

👉 Lembra do post sobre Native AOT? Quando você compila um executável nativo, metadata de reflection é removida agressivamente para reduzir tamanho e tempo de inicialização. `JsonSerializerContext` gera código que **não depende de reflection**, tornando a serialização JSON compatível com Native AOT — sem ele, seu app Native AOT simplesmente quebra ao tentar serializar

---

# 🔍 Nomes de propriedade e ordem customizados

```csharp
public record Produto(
    [property: JsonPropertyName("product_id")] int Id,
    [property: JsonPropertyName("product_name")] string Nome,
    [property: JsonPropertyOrder(1)] decimal Preco
);
```

👉 `JsonPropertyName` sobrescreve o nome de uma propriedade específica no JSON, útil quando você precisa seguir uma convenção de API externa (como `snake_case`) sem renomear a propriedade C# correspondente

---

# ⚠️ Erros comuns

- Escrever um `JsonConverter<T>` customizado para um problema que `PropertyNamingPolicy` ou `JsonPropertyName` já resolveriam de forma mais simples  
- Publicar com Native AOT sem configurar `JsonSerializerContext`, causando falhas de serialização em runtime que só aparecem depois do deploy  
- Ignorar `JsonIgnoreCondition.WhenWritingNull`, enviando campos `null` desnecessários em respostas de API  
- Misturar reflection-based e source-generator-based serialization no mesmo projeto sem entender as implicações de performance de cada uma  

---

# 📌 Conclusão

- `JsonConverter<T>` dá controle total sobre a serialização de um tipo específico  
- `JsonSerializerOptions` configura comportamento global, como `camelCase` e indentação  
- `JsonSerializerContext` gera código de serialização em tempo de compilação, sem reflection  
- Native AOT exige o source generator — reflection-based serialization não funciona nesse cenário  

👉 Com JSON dominado em profundidade, o próximo passo é ir ainda mais baixo no stack: como interagir com bibliotecas nativas escritas em C/C++ a partir do C#

---

# 🔥 Próximo passo

Agora que você domina serialização JSON avançada, o próximo nível é:

👉 **Fundamentos do C#: P/Invoke e Interoperabilidade Nativa**

Aqui você vai aprender a chamar bibliotecas nativas C/C++ diretamente do C#, indo além do que você viu no post sobre unsafe code.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
