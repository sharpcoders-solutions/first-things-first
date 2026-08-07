# 🧠 Fundamentos do C#: IEnumerable e Iteradores Customizados

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como um API Gateway centraliza o roteamento entre serviços  
- Extension methods e como o LINQ é construído por baixo dos panos  

Todo `foreach` que você já escreveu depende de uma interface que você provavelmente nunca implementou diretamente: `IEnumerable<T>`. Vamos abrir essa caixa-preta e ver como criar suas próprias sequências customizadas.

👉 **Vamos entender `IEnumerable<T>` a fundo, e o poder do `yield return`**

---

# 💡 O que o `foreach` realmente faz

```csharp
foreach (var numero in numeros)
{
    Console.WriteLine(numero);
}

// O código acima é açúcar sintático para isto:
var enumerador = numeros.GetEnumerator();
while (enumerador.MoveNext())
{
    var numero = enumerador.Current;
    Console.WriteLine(numero);
}
```

👉 **`foreach` = uma forma resumida de chamar `GetEnumerator()`, e então `MoveNext()`/`Current` repetidamente até acabar**

Qualquer tipo que implemente `IEnumerable<T>` (ou seja, tenha um `GetEnumerator()` que retorna um `IEnumerator<T>`) pode ser usado em um `foreach` — é esse contrato que `List<T>`, `Dictionary<TKey, TValue>` e todo o LINQ (do post sobre extension methods) sabem consumir

---

# 🏗️ Implementando `IEnumerable<T>` na unha

```csharp
public class Range3 : IEnumerable<int>
{
    private readonly int _inicio, _fim;

    public Range3(int inicio, int fim)
    {
        _inicio = inicio;
        _fim = fim;
    }

    public IEnumerator<int> GetEnumerator()
    {
        for (int i = _inicio; i <= _fim; i++)
            yield return i;
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

foreach (var numero in new Range3(1, 5))
    Console.WriteLine(numero); // 1, 2, 3, 4, 5
```

👉 Repare que a implementação manual de um `IEnumerator<T>` (com `MoveNext`, `Current`, `Reset`) seria bem mais verbosa — o `yield return` faz o compilador gerar essa máquina de estados automaticamente

---

# ✨ `yield return`: gerando valores sob demanda

```csharp
public IEnumerable<int> NumerosPares(int limite)
{
    for (int i = 0; i <= limite; i += 2)
    {
        Console.WriteLine($"Gerando {i}");
        yield return i;
    }
}

foreach (var par in NumerosPares(6))
{
    Console.WriteLine($"Recebido {par}");
    if (par == 4) break;
}

// Saída:
// Gerando 0
// Recebido 0
// Gerando 2
// Recebido 2
// Gerando 4
// Recebido 4
```

👉 **`yield return` = pausa a execução do método, entrega um valor, e retoma exatamente de onde parou na próxima chamada**

Note que "Gerando 6" **nunca** é impresso — o `break` interrompe antes de o método continuar. Isso é **lazy evaluation**: os valores são gerados um a um, sob demanda, não todos de uma vez em uma lista pré-computada

---

# 💾 Por que isso importa: memória e performance

```csharp
// ❌ Materializa TODOS os números na memória de uma vez
public List<int> NumerosPares(int limite)
{
    var lista = new List<int>();
    for (int i = 0; i <= limite; i += 2)
        lista.Add(i);
    return lista;
}

// ✅ Gera um número por vez, sem nunca guardar a sequência inteira
public IEnumerable<int> NumerosPares(int limite)
{
    for (int i = 0; i <= limite; i += 2)
        yield return i;
}
```

👉 Para sequências grandes (ou até infinitas), `yield return` evita alocar uma lista inteira na memória — cada valor é consumido e descartado antes do próximo ser gerado, exatamente como você viu no post sobre streams processando arquivos grandes

---

# ♾️ Sequências infinitas: só possível com lazy evaluation

```csharp
public IEnumerable<int> Fibonacci()
{
    int anterior = 0, atual = 1;

    while (true)
    {
        yield return anterior;
        (anterior, atual) = (atual, anterior + atual);
    }
}

var primeiros10 = Fibonacci().Take(10);
```

👉 Isso seria **impossível** com um método que retorna `List<int>` — a lista nunca terminaria de ser preenchida. Com `yield return`, o `Take(10)` do LINQ simplesmente para de pedir novos valores depois do décimo, e o método nunca gera o décimo primeiro

---

# ⚠️ Erros comuns

- Materializar uma sequência com `.ToList()` cedo demais, perdendo o benefício de lazy evaluation que `yield return` oferece  
- Esquecer que um método com `yield return` só executa quando você começa a **iterar** sobre ele, não quando você o chama  
- Fazer efeitos colaterais (gravar em banco, logar) dentro de um iterador `yield return`, sem perceber que eles só rodam conforme a enumeração avança  
- Tentar implementar `IEnumerator<T>` manualmente quando `yield return` resolveria o mesmo problema com muito menos código  

---

# 📌 Conclusão

- `foreach` é açúcar sintático para `GetEnumerator()` + `MoveNext()`/`Current`  
- `yield return` gera uma máquina de estados automaticamente, sem implementação manual de `IEnumerator<T>`  
- Iteradores são lazy: valores são gerados sob demanda, um de cada vez  
- Lazy evaluation viabiliza sequências grandes (ou infinitas) sem esgotar memória  

👉 Iteradores customizados abrem a porta para outro recurso que também redefine como um tipo se comporta: operadores customizados, que fazem `+`, `==` e outros símbolos funcionarem nos seus próprios tipos

---

# 🔥 Próximo passo

Agora que você domina iteradores customizados, o próximo nível é:

👉 **Fundamentos do C#: Operator Overloading**

Aqui você vai aprender a sobrecarregar operadores como `+`, `==` e `<` para que façam sentido nos seus próprios tipos.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
