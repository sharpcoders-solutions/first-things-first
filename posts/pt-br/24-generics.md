# 🧠 Fundamentos do C#: Generics

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Coleções como `List<T>` e `Dictionary<K, V>`  
- Interfaces, herança e tratamento de exceções  

Você já usa `List<T>` desde o post sobre coleções, mas talvez nunca tenha parado pra pensar: **por que esse `T` existe?**

👉 **Hoje você entende o mecanismo por trás disso: Generics**

---

# 💡 O problema que Generics resolve

Imagine escrever uma classe `Caixa` sem generics:

```csharp
class CaixaDeInt
{
    public int Item { get; set; }
}

class CaixaDeString
{
    public string Item { get; set; }
}
```

👉 Mesmo código, repetido para cada tipo. E se você precisar de uma caixa de `Pessoa`, `Produto`, `Pedido`...? A duplicação explode.

## 🔹 A alternativa ruim: usar `object`

```csharp
class CaixaGenerica
{
    public object Item { get; set; }
}

CaixaGenerica caixa = new CaixaGenerica();
caixa.Item = "texto";
int valor = (int)caixa.Item; // 💥 InvalidCastException em tempo de execução
```

👉 Usar `object` resolve a duplicação, mas perde a segurança de tipos — o erro só aparece quando o programa já está rodando

---

# 🧱 A solução: classes genéricas

```csharp
class Caixa<T>
{
    public T Item { get; set; }
}
```

```csharp
Caixa<int> caixaInt = new Caixa<int>();
caixaInt.Item = 10;

Caixa<string> caixaTexto = new Caixa<string>();
caixaTexto.Item = "Olá";

// caixaInt.Item = "texto"; // ❌ erro de compilação — não é possível
```

👉 `T` é um **placeholder de tipo**: você define o tipo real só na hora de usar a classe, e o compilador garante a segurança

Isso é exatamente como `List<T>` e `Dictionary<K, V>` funcionam por baixo dos panos — agora você sabe o "porquê" por trás do `<T>` que já vinha usando.

---

# ⚙️ Métodos genéricos

Generics não são exclusivos de classes — métodos também podem ser genéricos:

```csharp
T ObterPrimeiro<T>(List<T> lista)
{
    return lista[0];
}

int primeiroNumero = ObterPrimeiro(new List<int> { 1, 2, 3 });
string primeiroNome = ObterPrimeiro(new List<string> { "Maria", "João" });
```

👉 O compilador infere o tipo `T` automaticamente com base no argumento passado — você raramente precisa escrever `ObterPrimeiro<int>(...)` explicitamente

---

# 🧩 Múltiplos parâmetros de tipo

Uma classe (ou método) genérico pode ter mais de um placeholder:

```csharp
class Par<T1, T2>
{
    public T1 Primeiro { get; set; }
    public T2 Segundo { get; set; }
}

var par = new Par<string, int> { Primeiro = "idade", Segundo = 30 };
```

👉 É exatamente assim que `Dictionary<TKey, TValue>` é implementado internamente

---

# 🔒 Restrições de tipo (`where`)

Às vezes você precisa garantir que `T` tenha certas capacidades. É aí que entram as **constraints**:

```csharp
class Repositorio<T> where T : class
{
    private List<T> itens = new List<T>();

    public void Adicionar(T item) => itens.Add(item);
}
```

## 🔹 Constraints mais comuns

- `where T : class` → `T` precisa ser um tipo referência  
- `where T : struct` → `T` precisa ser um tipo valor  
- `where T : new()` → `T` precisa ter um construtor sem parâmetros  
- `where T : IComparable<T>` → `T` precisa implementar essa interface  
- `where T : Animal` → `T` precisa ser (ou herdar de) `Animal`  

```csharp
T CriarNovo<T>() where T : new()
{
    return new T(); // só é possível por causa da constraint
}
```

👉 Sem a constraint certa, o compilador não permite operações que dependem de uma capacidade específica do tipo

---

# 🕳️ `default(T)`: o valor "vazio" de qualquer tipo

```csharp
T ObterValorPadrao<T>()
{
    return default(T); // 0 para int, null para string, false para bool...
}
```

👉 `default(T)` devolve o valor padrão apropriado, seja qual for o tipo usado — essencial quando você não sabe se `T` é um tipo valor ou referência

---

# 🏗️ Exemplo real: um repositório genérico

```csharp
class Repositorio<T> where T : class
{
    private readonly List<T> _itens = new List<T>();

    public void Adicionar(T item) => _itens.Add(item);
    public void Remover(T item) => _itens.Remove(item);
    public IEnumerable<T> ListarTodos() => _itens;
}

class Produto
{
    public string Nome { get; set; }
}

class Cliente
{
    public string Nome { get; set; }
}
```

```csharp
var repositorioProdutos = new Repositorio<Produto>();
repositorioProdutos.Adicionar(new Produto { Nome = "Notebook" });

var repositorioClientes = new Repositorio<Cliente>();
repositorioClientes.Adicionar(new Cliente { Nome = "João" });
```

👉 Uma única classe `Repositorio<T>` serve para `Produto`, `Cliente`, ou qualquer outra entidade — sem duplicar código e sem perder segurança de tipos

---

# ⚠️ Erros comuns

- Usar `object` em vez de generics, perdendo segurança de tipos  
- Esquecer constraints necessárias e tentar operações que `T` não garante suportar  
- Criar generics onde uma interface simples já resolveria o problema  
- Confundir `Caixa<T>` (a definição) com `Caixa<int>` (o tipo já fechado, pronto para uso)  

---

# 📌 Conclusão

- Generics eliminam duplicação de código sem abrir mão da segurança de tipos  
- `T` é substituído pelo tipo real só no momento do uso  
- Constraints (`where`) garantem que `T` tenha as capacidades que seu código precisa  
- `List<T>` e `Dictionary<K, V>` são generics que você já usa desde o começo da trilha  

👉 Com generics, você escreve código reutilizável de verdade — uma peça central de qualquer biblioteca ou framework .NET

---

# 🔥 Próximo passo

Agora que você sabe escrever código genérico e reutilizável, o próximo nível é:

👉 **Fundamentos do C#: Delegates, Eventos e Expressões Lambda**

Aqui você vai aprender a tratar métodos como valores — a base de callbacks, eventos e programação funcional em C#.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
