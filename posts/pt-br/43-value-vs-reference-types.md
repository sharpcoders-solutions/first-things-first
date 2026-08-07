# 🧠 Fundamentos do C#: Value Types vs Reference Types em Profundidade

⏱️ Tempo de leitura: 9 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- CQRS separando comandos de consultas  
- Repository pattern e injeção de dependência, organizando a aplicação em camadas  

Você já escreveu dezenas de `class` e alguns `record`. Mas existe uma pergunta fundamental que raramente é respondida com profundidade: onde exatamente esses objetos vivem na memória, e por que isso importa para o desempenho do seu código?

👉 **Vamos entender a diferença entre value types e reference types de verdade**

---

# 💡 A diferença central: cópia de valor vs cópia de referência

```csharp
struct Ponto
{
    public int X;
    public int Y;
}

class Retangulo
{
    public int Largura;
    public int Altura;
}

var p1 = new Ponto { X = 1, Y = 2 };
var p2 = p1; // copia os VALORES de X e Y
p2.X = 99;
Console.WriteLine(p1.X); // 1 — p1 não mudou

var r1 = new Retangulo { Largura = 1, Altura = 2 };
var r2 = r1; // copia a REFERÊNCIA para o mesmo objeto
r2.Largura = 99;
Console.WriteLine(r1.Largura); // 99 — r1 mudou também
```

👉 **`struct` é um value type: atribuir copia os dados. `class` é um reference type: atribuir copia o endereço, e os dois nomes apontam para o mesmo objeto**

Esse comportamento não é um detalhe de sintaxe — é a diferença fundamental que explica bugs sutis quando alguém assume que `p2 = p1` significa "duas cópias independentes" quando, na verdade, `p1` e `p1` sendo classes significa "dois nomes para o mesmo objeto".

---

# 🗺️ Onde cada um vive: stack vs heap

👉 Value types **geralmente** vivem na stack (memória rápida, alocação e liberação automáticas ao sair do escopo). Reference types sempre têm seus dados no heap, com a stack guardando só o endereço

```csharp
void Metodo()
{
    int numero = 42;              // valor 42 fica na stack
    var pessoa = new Pessoa();    // objeto Pessoa fica no heap; "pessoa" (a referência) fica na stack
}
```

👉 Essa é uma simplificação útil, mas tem uma exceção importante: um `struct` que é **campo de uma classe**, ou está dentro de um array, vive no heap junto com o objeto que o contém — o value type só "empresta" a stack quando é uma variável local ou parâmetro isolado

---

# ⚖️ Igualdade: outro comportamento que muda

```csharp
struct Ponto { public int X, Y; }
class Retangulo { public int Largura, Altura; }

var p1 = new Ponto { X = 1, Y = 2 };
var p2 = new Ponto { X = 1, Y = 2 };
Console.WriteLine(p1.Equals(p2)); // true — structs comparam por valor, campo a campo

var r1 = new Retangulo { Largura = 1, Altura = 2 };
var r2 = new Retangulo { Largura = 1, Altura = 2 };
Console.WriteLine(r1.Equals(r2)); // false — classes comparam por referência, por padrão
```

👉 Structs herdam de `ValueType`, que já implementa `Equals` comparando campo a campo. Classes herdam de `object`, cujo `Equals` padrão compara se são o **mesmo objeto** na memória — a mesma distinção que você já viu na prática com `record` (igualdade por valor) versus `class` comum (igualdade por referência), só que agora você sabe o porquê por trás do comportamento

---

# 🎯 Quando usar `struct` em vez de `class`

```csharp
public readonly struct Dinheiro
{
    public decimal Valor { get; }
    public string Moeda { get; }

    public Dinheiro(decimal valor, string moeda)
    {
        Valor = valor;
        Moeda = moeda;
    }
}
```

👉 **Regra prática: prefira `struct` para tipos pequenos, imutáveis, que representam um único valor conceitual — coordenadas, dinheiro, intervalos de data**

- **Use `struct`** quando o tipo é pequeno (poucos campos), imutável, e faz sentido ser copiado livremente sem efeitos colaterais  
- **Use `class`** para a grande maioria dos casos: entidades com identidade, objetos que mudam de estado ao longo do tempo, ou qualquer coisa grande o suficiente para que copiar todo o conteúdo seja caro  

O `readonly` no struct acima não é decoração: ele garante ao compilador que nenhum método interno modifica o estado, evitando um problema real chamado "defensive copying", onde o runtime copia o struct silenciosamente toda vez que chama um método nele, só por precaução.

---

# 🔍 Tipos primitivos e `string`: onde eles se encaixam

```csharp
int numero = 10;        // struct (System.Int32) — value type
bool ativo = true;       // struct (System.Boolean) — value type
string nome = "Vitor";   // class (System.String) — reference type, mas imutável
```

👉 Todos os tipos numéricos (`int`, `double`, `decimal`, `bool`, `char`) são structs por baixo dos panos. `string`, por outro lado, é uma classe — mas se comporta de um jeito que confunde muita gente, porque é **imutável**: toda "modificação" cria uma string nova, então na prática ela parece um value type mesmo sendo referência

```csharp
string a = "Vitor";
string b = a;
b += " Santos"; // cria uma STRING NOVA; "a" continua "Vitor"
Console.WriteLine(a); // Vitor
```

---

# ⚠️ Erros comuns

- Assumir que copiar um `struct` grande é sempre mais barato que copiar uma referência — structs grandes (muitos campos) podem ser **mais** caros de copiar do que passar uma referência de 8 bytes  
- Criar `struct` mutáveis e guardá-los em coleções, gerando bugs onde a modificação parece não "colar" (porque cada acesso pode retornar uma cópia)  
- Comparar objetos com `==` esperando igualdade por valor em uma `class` comum, sem perceber que o padrão é comparação por referência  
- Ignorar o `readonly struct` e sofrer com "defensive copying" silencioso em código sensível a performance  

---

# 📌 Conclusão

- Value types (`struct`) copiam dados na atribuição; reference types (`class`) copiam a referência  
- Value types geralmente vivem na stack; reference types sempre têm seus dados no heap  
- Structs comparam por valor por padrão; classes comparam por referência por padrão  
- Prefira `struct` para tipos pequenos e imutáveis; `class` para a maioria dos outros casos  
- `string` é uma classe, mas sua imutabilidade faz com que se comporte de forma parecida com um value type  

👉 Entender essa distinção prepara o terreno para o próximo problema de performance: o que acontece quando um value type precisa ser tratado como `object`

---

# 🔥 Próximo passo

Agora que você entende onde cada tipo vive na memória, o próximo nível é:

👉 **Fundamentos do C#: Boxing, Unboxing e Performance de Tipos**

Aqui você vai ver o que acontece — e o custo real — quando um `struct` precisa ser convertido para `object`, e como evitar isso em código sensível a performance.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
