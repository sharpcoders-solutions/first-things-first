# 🧠 Fundamentos do C#: Coleções (Arrays, Listas e Dicionários)

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Variáveis, tipos e conversões  
- Condicionais e loops  
- Métodos, parâmetros e retorno  

Mas na prática, dados raramente aparecem sozinhos — eles vêm em grupos: uma lista de usuários, um carrinho de compras, um dicionário de configurações.

👉 **Hora de aprender a guardar e manipular grupos de dados**

---

# 💡 Arrays: a coleção mais básica

👉 **Array = uma coleção de tamanho fixo, com todos os elementos do mesmo tipo**

```csharp
int[] numeros = { 1, 2, 3, 4, 5 };
Console.WriteLine(numeros[0]); // 1
```

## 🔹 Características

- Tamanho definido na criação e **não muda depois**  
- Acesso rápido por índice (`numeros[2]`)  
- Índices começam em `0`  

```csharp
string[] nomes = new string[3];
nomes[0] = "Maria";
nomes[1] = "João";
nomes[2] = "Valentina";
```

👉 Arrays são ótimos quando você já sabe exatamente quantos elementos vai precisar

---

# 📦 `List<T>`: quando o tamanho muda

Na maioria dos casos reais, você não sabe quantos itens vai ter. É aí que entra a `List<T>`:

```csharp
List<string> nomes = new List<string>();
nomes.Add("Maria");
nomes.Add("João");
nomes.Remove("Maria");

Console.WriteLine(nomes.Count); // 1
```

## 🔹 Operações comuns

- `Add(item)` → adiciona um elemento  
- `Remove(item)` → remove pela referência do valor  
- `RemoveAt(indice)` → remove por posição  
- `Contains(item)` → verifica se existe  
- `Count` → quantidade de elementos  

👉 `List<T>` é a coleção mais usada no dia a dia — cresce e encolhe automaticamente

## 🔹 Percorrendo uma lista

```csharp
foreach (string nome in nomes)
{
    Console.WriteLine(nome);
}
```

---

# 🗂️ `Dictionary<K, V>`: pares de chave e valor

Quando você precisa buscar valores por uma chave em vez de um índice numérico:

```csharp
Dictionary<string, int> idades = new Dictionary<string, int>();
idades["Maria"] = 25;
idades["João"] = 30;

Console.WriteLine(idades["Maria"]); // 25
```

## 🔹 Operações comuns

- `dicionario[chave] = valor` → adiciona ou atualiza  
- `ContainsKey(chave)` → verifica se a chave existe  
- `Remove(chave)` → remove pela chave  
- `TryGetValue(chave, out valor)` → busca segura, sem lançar exceção  

```csharp
if (idades.TryGetValue("Valentina", out int idade))
{
    Console.WriteLine(idade);
}
else
{
    Console.WriteLine("Não encontrado");
}
```

👉 Use `TryGetValue` sempre que a chave não for garantida — evita `KeyNotFoundException`

## 🔹 Percorrendo um dicionário

```csharp
foreach (KeyValuePair<string, int> par in idades)
{
    Console.WriteLine($"{par.Key}: {par.Value}");
}
```

---

# 🔀 Array vs List vs Dictionary: qual escolher?

## 🔹 Use `Array` quando:
- O tamanho é fixo e conhecido  
- Performance máxima é prioridade  

## 🔹 Use `List<T>` quando:
- O tamanho pode mudar  
- Você precisa de ordem e acesso por índice  

## 🔹 Use `Dictionary<K, V>` quando:
- Você busca valores por uma chave, não por posição  
- Chaves precisam ser únicas  

👉 Na dúvida, `List<T>` costuma ser o ponto de partida mais seguro

---

# ⚠️ Erros comuns

- Tentar adicionar itens em um array (arrays não crescem — use `List<T>`)  
- Acessar uma chave inexistente em um `Dictionary` sem usar `TryGetValue`  
- Usar `List<T>` quando um `Dictionary` resolveria a busca de forma mais eficiente  
- Esquecer que índices começam em `0`, causando `IndexOutOfRangeException`  

---

# 📌 Conclusão

- `Array` guarda coleções de tamanho fixo  
- `List<T>` cresce e encolhe dinamicamente  
- `Dictionary<K, V>` busca valores por chave, não por posição  
- Escolher a coleção certa evita código complicado e ineficiente  

👉 Com essas três estruturas, você já consegue modelar a maioria dos problemas do dia a dia

---

# 🔥 Próximo passo

Agora que você sabe guardar grupos de dados, o próximo nível é:

👉 **Fundamentos do C#: Introdução ao LINQ**

Aqui você vai aprender a consultar, filtrar e transformar coleções de forma muito mais expressiva.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
