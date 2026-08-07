# 🧠 Fundamentos do C#: Culture e Globalização

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Server GC vs Workstation GC e como configurar o modo certo de coleta  
- `decimal` e formatação de números, usados sem muita discussão desde os primeiros posts sobre e-commerce  

Um preço formatado como `1.234,56` no Brasil aparece como `1,234.56` nos Estados Unidos — mesmo número, representação diferente. Se sua aplicação algum dia tiver usuários em mais de um país, ignorar isso gera bugs sutis e frustrantes.

👉 **Vamos aprender Culture e Globalização em C#**

---

# 💡 O que é uma `CultureInfo`?

👉 **`CultureInfo` = um objeto que representa as convenções de formatação de uma região específica: separador decimal, formato de data, símbolo de moeda, e mais**

```csharp
var culturaBrasil = new CultureInfo("pt-BR");
var culturaEUA = new CultureInfo("en-US");

decimal preco = 1234.56m;

Console.WriteLine(preco.ToString("C", culturaBrasil)); // R$ 1.234,56
Console.WriteLine(preco.ToString("C", culturaEUA));     // $1,234.56
```

👉 O mesmo `decimal`, a mesma chamada de formatação — só a `CultureInfo` muda, e o resultado reflete exatamente a convenção esperada em cada região

---

# ⚠️ O perigo de `CultureInfo.CurrentCulture`

```csharp
// ❌ Depende da cultura configurada no servidor/máquina onde o código roda
decimal valor = decimal.Parse("1234.56"); // funciona no servidor com cultura en-US...
                                            // ...mas quebra ou interpreta errado em um servidor pt-BR
```

👉 `CultureInfo.CurrentCulture` reflete a cultura da **thread atual**, que geralmente vem da configuração do sistema operacional ou do request HTTP — isso significa que o mesmo código pode se comportar de forma completamente diferente dependendo de onde ele roda, um dos bugs mais traiçoeiros em aplicações que fazem parsing de números ou datas

---

# 🔒 `CultureInfo.InvariantCulture`: previsibilidade para dados internos

```csharp
// ✅ Sempre se comporta da mesma forma, independente de onde o código roda
decimal valor = decimal.Parse("1234.56", CultureInfo.InvariantCulture);
string json = valor.ToString(CultureInfo.InvariantCulture);
```

👉 **`InvariantCulture` = uma cultura "neutra", baseada em convenções americanas, que nunca muda independente do ambiente**

**Regra prática: use `InvariantCulture` para qualquer dado que não é exibido diretamente para um usuário** — serialização JSON (lembra do post sobre `System.Text.Json`?), chaves de cache, logs, comunicação entre serviços, arquivos de configuração. Reserve a cultura do usuário exclusivamente para formatação de **exibição**

---

# 🌐 Detectando a cultura do usuário em uma API ASP.NET Core

```csharp
// Program.cs
var opcoesLocalizacao = new RequestLocalizationOptions()
    .SetDefaultCulture("en-US")
    .AddSupportedCultures("en-US", "pt-BR", "es-ES")
    .AddSupportedUICultures("en-US", "pt-BR", "es-ES");

app.UseRequestLocalization(opcoesLocalizacao);
```

```csharp
[HttpGet("produtos/{id}")]
public IActionResult ObterProduto(int id)
{
    var produto = _repositorio.ObterPorId(id);
    var precoFormatado = produto.Preco.ToString("C", CultureInfo.CurrentCulture);
    return Ok(new { produto.Nome, Preco = precoFormatado });
}
```

👉 O middleware de localização lê o header `Accept-Language` da requisição HTTP e configura `CultureInfo.CurrentCulture` automaticamente para aquele request — cada requisição pode ter uma cultura diferente, sem interferir umas nas outras

---

# 📅 Datas e culturas

```csharp
var data = new DateOnly(2026, 3, 15);

Console.WriteLine(data.ToString("d", new CultureInfo("pt-BR"))); // 15/03/2026
Console.WriteLine(data.ToString("d", new CultureInfo("en-US"))); // 3/15/2026
```

👉 Lembra do post sobre `DateOnly`/`TimeOnly`? A mesma data pode ser exibida em ordens completamente diferentes (dia/mês/ano vs mês/dia/ano) dependendo da cultura — outra fonte clássica de confusão quando ignorada

---

# 🔤 Comparação de strings: outra armadilha de cultura

```csharp
// ❌ Comparação sensível à cultura, pode se comportar de forma inesperada
bool igual = "Straße".Equals("STRASSE", StringComparison.CurrentCultureIgnoreCase);

// ✅ Comparação ordinal, previsível, sem regras linguísticas
bool igualOrdinal = "abc".Equals("ABC", StringComparison.OrdinalIgnoreCase);
```

👉 **Regra prática: use `StringComparison.Ordinal`/`OrdinalIgnoreCase` para comparações técnicas** (chaves, identificadores, nomes de arquivo) **e comparação sensível à cultura só para texto realmente exibido ao usuário e comparado linguisticamente** — misturar os dois é uma fonte comum de bugs difíceis de reproduzir, porque dependem da cultura configurada em cada ambiente

---

# ⚠️ Erros comuns

- Usar `CultureInfo.CurrentCulture` (implícito, sem especificar) ao fazer `Parse`/`ToString` de dados internos, quebrando quando o código roda em um ambiente com cultura diferente  
- Comparar strings sem especificar `StringComparison`, deixando o comportamento depender silenciosamente da cultura do sistema  
- Ignorar completamente globalização até o produto já ter usuários internacionais, exigindo uma refatoração grande e arriscada depois  
- Formatar dados enviados entre serviços (APIs internas, filas) usando a cultura do usuário, em vez de `InvariantCulture`  

---

# 📌 Conclusão

- `CultureInfo` controla como números, datas e moedas são formatados para exibição  
- `CurrentCulture` reflete o ambiente de execução — imprevisível para dados internos  
- `InvariantCulture` é a escolha correta para serialização, logs e comunicação entre sistemas  
- `StringComparison.Ordinal` deve ser o padrão para comparações técnicas, não linguísticas  

👉 Com globalização dominada, o próximo passo é olhar para as ferramentas que permitem inspecionar e manipular o próprio código C# em tempo de execução — assemblies, metadata e reflection avançada

---

# 🔥 Próximo passo

Agora que você formata dados corretamente para qualquer região do mundo, o próximo nível é:

👉 **Fundamentos do C#: Assembly, Metadata e Reflection em Profundidade**

Aqui você vai aprender a inspecionar assemblies inteiros, ler metadata customizada, e entender como o runtime organiza tipos compilados.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
