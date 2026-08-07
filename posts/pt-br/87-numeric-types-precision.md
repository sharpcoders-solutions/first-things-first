# 🧠 Fundamentos do C#: Tipos Numéricos e Precisão (float, double e decimal)

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `DateOnly` e `TimeOnly` para representar data e hora com precisão  
- `decimal` usado, sem muita explicação, em todo cálculo de dinheiro desde os primeiros posts sobre e-commerce  

Você já deve ter ouvido (ou até vivido) a história de um sistema financeiro quebrado por causa de arredondamento incorreto. Isso quase sempre vem de uma escolha errada entre `float`, `double` e `decimal` — três tipos que parecem intercambiáveis, mas não são.

👉 **Vamos entender a diferença real entre eles**

---

# 💡 O experimento clássico

```csharp
double resultado = 0.1 + 0.2;
Console.WriteLine(resultado); // 0.30000000000000004
Console.WriteLine(resultado == 0.3); // false
```

👉 Isso não é um bug do C# — é uma consequência de como `float` e `double` representam números: em **binário de ponto flutuante**, seguindo o padrão IEEE 754. Alguns números decimais simples (como `0.1`) não têm representação binária exata, do mesmo jeito que `1/3` não tem representação decimal exata (`0.333...`)

---

# 🔬 `float` e `double`: precisão aproximada, rápidos

```csharp
float numeroFloat = 3.14159265358979f;   // ~7 dígitos de precisão, 4 bytes
double numeroDouble = 3.14159265358979;  // ~15-17 dígitos de precisão, 8 bytes
```

👉 `float` e `double` são otimizados para **performance e alcance**, não para precisão decimal exata. Ambos podem representar números astronomicamente grandes ou minúsculos, com uma margem de erro aceitável para a maioria dos cálculos científicos e gráficos

```csharp
// Onde float/double brilham: física, gráficos, machine learning
float velocidade = CalcularVelocidade(massa, forca);
double distancia = CalcularTrajetoria(angulo, velocidadeInicial);
```

---

# 💰 `decimal`: precisão exata, mais lento

```csharp
decimal preco = 19.99m; // o sufixo "m" indica um literal decimal
decimal total = preco * 3;

Console.WriteLine(total); // 59.97 — exato, sem surpresas
```

👉 **`decimal` = um tipo de 128 bits que representa números em **base 10**, não em binário — projetado especificamente para eliminar os erros de arredondamento que `float`/`double` têm com valores decimais comuns**

O trade-off: `decimal` ocupa mais memória (16 bytes contra 8 do `double`) e é mais lento em operações aritméticas, porque não é suportado diretamente pela unidade de ponto flutuante do processador — é implementado em software

---

# ⚖️ Comparando os três lado a lado

```csharp
float f = 0.1f + 0.2f;
double d = 0.1 + 0.2;
decimal m = 0.1m + 0.2m;

Console.WriteLine(f); // 0.3 (mas pode divergir em casas menos visíveis)
Console.WriteLine(d); // 0.30000000000000004
Console.WriteLine(m); // 0.3 — exato
```

| | `float` | `double` | `decimal` |
|---|---|---|---|
| Tamanho | 4 bytes | 8 bytes | 16 bytes |
| Precisão | ~7 dígitos | ~15-17 dígitos | 28-29 dígitos, exato em base 10 |
| Velocidade | Rápido | Rápido | Mais lento |
| Uso ideal | Gráficos, jogos | Cálculos científicos | Dinheiro, valores exatos |

---

# 🎯 A regra prática: dinheiro é sempre `decimal`

```csharp
public class ItemPedido
{
    public decimal Preco { get; set; }
    public int Quantidade { get; set; }
    public decimal Subtotal => Preco * Quantidade;
}
```

👉 Você já usou `decimal` para dinheiro em praticamente todo post desta trilha que envolveu preços — agora você entende o porquê: qualquer erro de arredondamento em um valor financeiro, por menor que seja, é inaceitável, e `decimal` é o único dos três tipos desenhado para nunca introduzir esse tipo de erro em operações decimais comuns

---

# 🔍 `float`/`double` também não são "errados" — são especializados

```csharp
// Física e simulações: pequenas imprecisões são aceitáveis e esperadas
double energia = 0.5 * massa * velocidade * velocidade;

// Coordenadas de tela em um jogo
float posicaoX = 152.375f;
```

👉 O erro **não** é usar `float`/`double` — o erro é usá-los para dinheiro. Em domínios onde uma margem mínima de erro é irrelevante (posições na tela, cálculos físicos aproximados), `float`/`double` são a escolha certa pela performance e pelo menor consumo de memória

---

# ⚠️ Erros comuns

- Usar `double` para representar dinheiro, introduzindo erros de arredondamento que se acumulam ao longo de milhares de transações  
- Comparar valores `float`/`double` com `==` esperando igualdade exata, quando pequenas imprecisões de ponto flutuante tornam isso não confiável  
- Usar `decimal` em cálculos científicos de alta performance, pagando um custo de velocidade desnecessário onde a precisão exata não importa  
- Misturar `float` e `double` na mesma expressão sem conversões explícitas, gerando perda de precisão silenciosa  

---

# 📌 Conclusão

- `float`/`double` representam números em binário de ponto flutuante — rápidos, mas com imprecisão inerente em valores decimais  
- `decimal` representa números em base 10 — exato para valores decimais comuns, mas mais lento e maior  
- Dinheiro e qualquer valor que exija precisão decimal exata devem sempre usar `decimal`  
- Cálculos científicos, gráficos e física geralmente devem usar `float`/`double`, pela performance  

👉 Entender por que esses três tipos existem prepara o terreno para o próximo recurso: como escrever código genérico que funciona com qualquer um deles, sem duplicar lógica

---

# 🔥 Próximo passo

Agora que você escolhe o tipo numérico certo para cada cenário, o próximo nível é:

👉 **Fundamentos do C#: Generic Math**

Aqui você vai aprender a escrever um único método genérico que funciona com `int`, `double`, `decimal` e qualquer outro tipo numérico, usando os static abstract interface members que você já conhece.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
