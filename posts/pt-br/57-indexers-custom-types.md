# 🧠 Fundamentos do C#: Indexers em Tipos Customizados

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como sobrecarregar operadores como `+`, `==` e `<` nos seus tipos  
- `IComparable<T>` para habilitar ordenação customizada  

Você usa `lista[0]` e `dicionario["chave"]` desde as primeiras semanas desta trilha. Mas será que dá pra ter essa mesma sintaxe de colchetes em uma classe sua? Dá — e o recurso que faz isso se chama indexer.

👉 **Vamos aprender a criar indexers customizados**

---

# 💡 O que é um indexer?

```csharp
public class Semana
{
    private readonly string[] _dias =
        { "Domingo", "Segunda", "Terça", "Quarta", "Quinta", "Sexta", "Sábado" };

    public string this[int indice] => _dias[indice];
}

var semana = new Semana();
Console.WriteLine(semana[0]); // Domingo
Console.WriteLine(semana[3]); // Quarta
```

👉 **Indexer = um membro especial, declarado com `this[...]`, que permite acessar um objeto usando a sintaxe de colchetes, como se fosse um array**

Por baixo dos panos, `semana[0]` é convertido pelo compilador para uma chamada de método — mas para quem lê o código, ele parece um acesso direto de array

---

# ✏️ Indexer com get e set

```csharp
public class Inventario
{
    private readonly Dictionary<string, int> _estoque = new();

    public int this[string produto]
    {
        get => _estoque.TryGetValue(produto, out var quantidade) ? quantidade : 0;
        set => _estoque[produto] = value;
    }
}

var inventario = new Inventario();
inventario["Notebook"] = 15;      // usa o "set"
int quantidade = inventario["Notebook"]; // usa o "get" — 15

int semEstoque = inventario["Mouse"]; // 0 — não lança exceção, retorna default
```

👉 Assim como uma propriedade, um indexer pode ter `get` e `set` separados. Aqui, o `get` trata chaves inexistentes retornando `0` em vez de lançar exceção — uma decisão de design que você controla totalmente

---

# 🔢 Indexers com múltiplos parâmetros

```csharp
public class Matriz
{
    private readonly int[,] _dados;

    public Matriz(int linhas, int colunas)
    {
        _dados = new int[linhas, colunas];
    }

    public int this[int linha, int coluna]
    {
        get => _dados[linha, coluna];
        set => _dados[linha, coluna] = value;
    }
}

var matriz = new Matriz(3, 3);
matriz[0, 0] = 1;
matriz[1, 1] = 5;
Console.WriteLine(matriz[1, 1]); // 5
```

👉 Diferente de arrays comuns, indexers aceitam **qualquer número e tipo** de parâmetros — inclusive combinações como `this[string chave, DateTime data]`, algo que um array nativo do C# jamais permitiria

---

# 🎯 Sobrecarregando indexers

```csharp
public class Cache
{
    private readonly Dictionary<int, string> _porId = new();
    private readonly Dictionary<string, string> _porChave = new();

    public string this[int id] => _porId.TryGetValue(id, out var valor) ? valor : null;
    public string this[string chave] => _porChave.TryGetValue(chave, out var valor) ? valor : null;
}

var cache = new Cache();
var porId = cache[42];        // usa this[int]
var porChave = cache["ativo"]; // usa this[string]
```

👉 Assim como métodos comuns, indexers podem ser sobrecarregados — o compilador escolhe a versão certa com base no tipo do argumento passado entre colchetes

---

# 🛡️ Validação dentro do indexer

```csharp
public class Semana
{
    private readonly string[] _dias =
        { "Domingo", "Segunda", "Terça", "Quarta", "Quinta", "Sexta", "Sábado" };

    public string this[int indice]
    {
        get
        {
            if (indice < 0 || indice >= _dias.Length)
                throw new ArgumentOutOfRangeException(nameof(indice), "Índice deve ser entre 0 e 6");

            return _dias[indice];
        }
    }
}
```

👉 Diferente de um array puro, um indexer é código de verdade — você pode validar, logar, ou aplicar qualquer regra de negócio antes de retornar o valor, exatamente como faria em qualquer outro método

---

# ⚠️ Erros comuns

- Criar indexers para tipos que não têm uma noção natural de "acesso por posição ou chave", tornando a API confusa em vez de intuitiva  
- Não validar o índice recebido, deixando exceções genéricas do array interno vazarem em vez de uma mensagem clara  
- Esquecer que indexers podem ter `get` e `set` independentes — um indexer só leitura é perfeitamente válido  
- Duplicar lógica entre indexer e métodos equivalentes (`ObterPorId`, `DefinirPorId`) em vez de centralizar em um só lugar  

---

# 📌 Conclusão

- Indexers permitem a sintaxe `objeto[chave]` em tipos customizados, via `this[...]`  
- Podem ter `get` e `set` separados, igual propriedades comuns  
- Aceitam múltiplos parâmetros de qualquer tipo, superando as limitações de arrays nativos  
- Podem ser sobrecarregados, permitindo múltiplas formas de acessar o mesmo objeto  

👉 Indexers deixam seus tipos mais expressivos e naturais de usar — o último recurso avançado desta série de posts vai além, permitindo que uma **interface inteira** exija operadores e membros estáticos dos tipos que a implementam

---

# 🔥 Próximo passo

Agora que você sabe criar indexers customizados, o próximo nível é:

👉 **Fundamentos do C#: Static Abstract Interface Members**

Aqui você vai aprender um dos recursos mais recentes do C#, que permite interfaces exigirem operadores e membros estáticos das classes que as implementam.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
