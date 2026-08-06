# 🧠 Fundamentos do C#: Classes e Objetos (Introdução à Programação Orientada a Objetos)

⏱️ Tempo de leitura: 10 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Variáveis, tipos, condicionais e loops  
- Métodos, parâmetros e retorno  
- Coleções e LINQ  

Tudo isso são ferramentas. Mas até agora, seu código organiza **ações** — não organiza **conceitos do mundo real**.

👉 **Esse é o papel da Programação Orientada a Objetos (OOP), e é aqui que o C# realmente mostra sua força**

Como esse é um dos temas mais importantes da carreira de um desenvolvedor C#, vamos com calma e profundidade — vale a pena ler devagar.

---

# 💡 O que é Orientação a Objetos?

👉 **OOP = organizar o código em torno de "objetos" que combinam dados e comportamento**

Em vez de pensar em "funções que processam dados soltos", você passa a pensar em **entidades**: um `Cliente`, um `Pedido`, uma `ContaBancaria`. Cada uma tem:

- **Dados** que a descrevem (estado)  
- **Ações** que ela pode realizar (comportamento)  

👉 Isso aproxima o código da forma como pensamos sobre o mundo real

---

# 🏗️ Classe vs Objeto: o conceito mais importante deste post

Essa é a base de tudo, e onde a maioria dos iniciantes se confunde.

## 🔹 Classe

👉 **Classe = um molde, uma planta baixa**

```csharp
class Pessoa
{
    public string Nome;
    public int Idade;
}
```

A classe **não é** uma pessoa. Ela só define **como** uma pessoa é estruturada: que toda pessoa tem nome e idade.

## 🔹 Objeto

👉 **Objeto = uma instância real, criada a partir da classe**

```csharp
Pessoa pessoa1 = new Pessoa();
pessoa1.Nome = "Maria";
pessoa1.Idade = 25;

Pessoa pessoa2 = new Pessoa();
pessoa2.Nome = "João";
pessoa2.Idade = 30;
```

👉 A partir de **uma** classe, você pode criar **infinitos** objetos diferentes, cada um com seu próprio estado

Pense assim: `Pessoa` é a planta de uma casa. `pessoa1` e `pessoa2` são duas casas construídas a partir dela — parecidas na estrutura, mas com moradores diferentes.

---

# 🧱 Anatomia de uma classe

Uma classe bem construída normalmente tem quatro partes:

## 🔹 1. Campos (fields)

```csharp
class ContaBancaria
{
    private decimal saldo;
}
```

👉 Campos guardam o estado interno do objeto. Por convenção, costumam ser `private` — só a própria classe mexe neles diretamente

## 🔹 2. Propriedades (properties)

```csharp
class ContaBancaria
{
    private decimal saldo;

    public decimal Saldo
    {
        get { return saldo; }
        private set { saldo = value; }
    }
}
```

👉 Propriedades controlam **como** o mundo externo lê e escreve o estado do objeto — sem expor o campo diretamente

## 🔹 Auto-properties (a forma mais comum no dia a dia)

Quando não há lógica extra na leitura/escrita, C# permite uma versão mais enxuta:

```csharp
class Pessoa
{
    public string Nome { get; set; }
    public int Idade { get; private set; }
}
```

👉 `get; set;` gera o campo escondido automaticamente — você não precisa declará-lo

## 🔹 3. Construtores

O construtor define como um objeto **nasce**:

```csharp
class Pessoa
{
    public string Nome { get; }
    public int Idade { get; }

    public Pessoa(string nome, int idade)
    {
        Nome = nome;
        Idade = idade;
    }
}

Pessoa pessoa = new Pessoa("Maria", 25);
```

👉 Se você não declarar nenhum construtor, o C# gera um construtor padrão (sem parâmetros) automaticamente — mas assim que você declara um construtor próprio, o padrão desaparece

## 🔹 O papel do `this`

```csharp
public Pessoa(string nome, int idade)
{
    this.Nome = nome; // "this" refere-se ao objeto atual
    this.Idade = idade;
}
```

👉 `this` é útil principalmente quando o nome do parâmetro é igual ao nome do campo/propriedade

## 🔹 4. Métodos

Métodos definem o **comportamento** do objeto:

```csharp
class ContaBancaria
{
    public decimal Saldo { get; private set; }

    public void Depositar(decimal valor)
    {
        if (valor <= 0)
            throw new ArgumentException("Valor de depósito inválido");

        Saldo += valor;
    }

    public bool Sacar(decimal valor)
    {
        if (valor > Saldo) return false;

        Saldo -= valor;
        return true;
    }
}
```

👉 Note como as regras de negócio (não sacar mais que o saldo, não depositar valor negativo) vivem **dentro** da classe, perto dos dados que elas protegem

---

# 🔒 Encapsulamento: o pilar mais imediato

👉 **Encapsulamento = esconder os detalhes internos e expor só o necessário**

Compare as duas versões da conta bancária:

```csharp
// ❌ Sem encapsulamento
class ContaBancariaRuim
{
    public decimal Saldo;
}

conta.Saldo = -500; // nada impede isso
```

```csharp
// ✅ Com encapsulamento
class ContaBancaria
{
    public decimal Saldo { get; private set; }

    public void Depositar(decimal valor)
    {
        if (valor <= 0) throw new ArgumentException("Valor inválido");
        Saldo += valor;
    }
}

conta.Saldo = -500; // erro de compilação — Saldo só muda via Depositar/Sacar
```

👉 Encapsulamento não é sobre esconder por esconder — é sobre **garantir que o objeto nunca fique em um estado inválido**

---

# ⚙️ Membros estáticos vs membros de instância

Nem tudo em uma classe precisa de um objeto criado:

```csharp
class Calculadora
{
    public static int Somar(int a, int b) => a + b; // static: pertence à classe
    public int Historico { get; set; }               // instância: pertence ao objeto
}

int resultado = Calculadora.Somar(2, 3); // sem precisar de "new"
```

## 🔹 Quando usar `static`

- Comportamento que **não depende** do estado de um objeto específico  
- Utilitários, constantes, métodos auxiliares  

👉 Se o método precisa acessar dados que variam por objeto, ele não deve ser `static`

---

# 🔑 Modificadores de acesso

Controlam **quem** pode enxergar cada membro da classe:

- `public` → acessível de qualquer lugar  
- `private` → acessível só dentro da própria classe  
- `protected` → acessível na classe e em quem herda dela  
- `internal` → acessível dentro do mesmo projeto/assembly  

👉 A regra geral: comece o mais restritivo possível (`private`) e só abra acesso (`public`) quando realmente necessário

---

# 🧩 Os quatro pilares da OOP (visão geral)

Classes e objetos são a porta de entrada. A partir daqui, a OOP se apoia em quatro pilares:

1. **Encapsulamento** → proteger o estado interno (você já viu isso hoje)  
2. **Abstração** → expor só o que importa, esconder a complexidade  
3. **Herança** → reaproveitar comportamento entre classes relacionadas  
4. **Polimorfismo** → tratar objetos diferentes de forma uniforme  

👉 Vamos dedicar posts inteiros para herança e polimorfismo em seguida — eles merecem profundidade própria

---

# 🏗️ Exemplo completo: juntando tudo

```csharp
class ContaBancaria
{
    public string Titular { get; }
    public decimal Saldo { get; private set; }

    public ContaBancaria(string titular, decimal saldoInicial = 0)
    {
        Titular = titular;
        Saldo = saldoInicial;
    }

    public void Depositar(decimal valor)
    {
        if (valor <= 0)
            throw new ArgumentException("Valor de depósito inválido");

        Saldo += valor;
    }

    public bool Sacar(decimal valor)
    {
        if (valor <= 0 || valor > Saldo) return false;

        Saldo -= valor;
        return true;
    }

    public override string ToString() => $"{Titular}: R$ {Saldo}";
}

var conta = new ContaBancaria("Vitor", 1000);
conta.Depositar(500);
conta.Sacar(200);

Console.WriteLine(conta); // Vitor: R$ 1300
```

👉 Repare como cada conceito deste post aparece aqui: campos, propriedades, construtor, métodos, encapsulamento e até `this` (implícito nas propriedades)

---

# ⚠️ Erros comuns

- Deixar campos `public` em vez de expor propriedades controladas  
- Esquecer que sem construtor próprio, o C# gera um construtor vazio automaticamente  
- Confundir membros `static` com membros de instância  
- Colocar regras de negócio fora da classe, em vez de perto dos dados que elas protegem  
- Achar que "classe" e "objeto" são sinônimos — são conceitos diferentes  

---

# 📌 Conclusão

- Classe é o molde; objeto é a instância real criada a partir dele  
- Campos guardam estado; propriedades controlam o acesso a esse estado  
- Construtores definem como um objeto nasce  
- Métodos definem o comportamento, e devem proteger a integridade dos dados  
- Encapsulamento garante que o objeto nunca fique em um estado inválido  

👉 Você acabou de dar o passo mais importante da carreira em C#: pensar em objetos, não só em instruções soltas

---

# 🔥 Próximo passo

Agora que você entende classes e objetos, o próximo nível é:

👉 **Fundamentos do C#: Herança e Polimorfismo**

Aqui você vai aprender a reaproveitar comportamento entre classes e tratar objetos diferentes de forma uniforme.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
