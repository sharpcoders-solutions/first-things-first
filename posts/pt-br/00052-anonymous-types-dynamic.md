# 🧠 Fundamentos do C#: Anonymous Types e dynamic

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Tuplas e `ValueTuple` para agrupar valores sem criar um tipo formal  
- Quando preferir tupla e quando preferir um `record`  

Tuplas resolvem bem agrupar dois ou três valores. Mas existe um recurso ainda mais informal — criar um objeto com propriedades nomeadas sem declarar **nenhum** tipo, nem mesmo uma tupla. E, do lado oposto, existe um jeito de abandonar completamente a verificação de tipos em tempo de compilação. Vamos ver os dois, e por que um é útil e o outro quase sempre não é.

👉 **Vamos conhecer anonymous types e `dynamic`**

---

# 💡 Anonymous Types: um objeto sem declarar uma classe

```csharp
var pessoa = new { Nome = "Vitor", Idade = 30 };

Console.WriteLine(pessoa.Nome); // Vitor
Console.WriteLine(pessoa.Idade); // 30
```

👉 **Anonymous type = uma classe gerada pelo compilador em tempo de compilação, sem nome visível no seu código**

Por trás dos panos, o compilador cria uma classe de verdade (algo como `<>f__AnonymousType0`), com propriedades `get`-only, `Equals`, `GetHashCode` e `ToString` implementados automaticamente — você só nunca vê o nome dela

---

# 🎯 Onde anonymous types brilham: projeções LINQ

```csharp
var resumoPedidos = pedidos
    .Where(p => p.Status == StatusPedido.Entregue)
    .Select(p => new { p.Id, p.Cliente, Total = p.Itens.Sum(i => i.Subtotal) });

foreach (var resumo in resumoPedidos)
{
    Console.WriteLine($"{resumo.Id} - {resumo.Cliente}: {resumo.Total:C}");
}
```

👉 Lembra do post sobre extension methods e LINQ? `Select` é o lugar clássico onde anonymous types aparecem — você projeta só os campos que interessam, sem precisar declarar um DTO para um resultado usado uma única vez, dentro de um único método

---

# ⚖️ Igualdade em anonymous types: por valor

```csharp
var p1 = new { Nome = "Vitor", Idade = 30 };
var p2 = new { Nome = "Vitor", Idade = 30 };

Console.WriteLine(p1 == p2);      // false — não sobrecarrega o operador ==
Console.WriteLine(p1.Equals(p2)); // true — Equals gerado compara por valor
```

👉 Anonymous types comparam por valor via `Equals` — o mesmo comportamento de um `record` (lembra?) — mas só se as propriedades tiverem os mesmos nomes, na mesma ordem, com os mesmos tipos. Isso acontece porque o compilador reutiliza a **mesma classe gerada** para dois anonymous types com essa assinatura idêntica

---

# 🚧 As limitações: por que não usar anonymous types em toda parte

```csharp
public object CriarPessoa() // ❌ não dá pra declarar o tipo de retorno
{
    return new { Nome = "Vitor", Idade = 30 };
}
```

👉 **Um anonymous type nunca pode ser o tipo de retorno declarado de um método, nem um parâmetro** — porque ele não tem nome para você escrever na assinatura. Isso limita seu uso a escopos locais: variáveis, projeções LINQ e código que nunca sai do método onde foi criado. Para qualquer coisa que atravesse uma fronteira de método, um `record` é a ferramenta certa

---

# 🎭 `dynamic`: abandonando a verificação de tipos em tempo de compilação

```csharp
dynamic valor = 10;
valor = "agora sou uma string";
valor = new { Nome = "Vitor" };

Console.WriteLine(valor.PropriedadeQueNaoExiste); // compila! Mas explode em runtime
```

👉 **`dynamic` = um tipo que desliga a verificação do compilador; os erros que normalmente apareceriam ao compilar só aparecem em tempo de execução**

Diferente de `object`, que exige um cast explícito para acessar membros, `dynamic` permite chamar qualquer método ou propriedade — e o compilador simplesmente aceita, confiando que aquilo vai existir quando o código rodar

---

# 🕵️ Quando `dynamic` realmente se justifica

```csharp
// Interoperabilidade com COM (Excel, Word) — cenário clássico
dynamic excel = Activator.CreateInstance(Type.GetTypeFromProgID("Excel.Application"));
excel.Visible = true;

// Trabalhando com JSON de estrutura totalmente imprevisível
dynamic json = JsonConvert.DeserializeObject("{\"campo\": \"valor\"}");
Console.WriteLine(json.campo);
```

👉 Os casos legítimos são raros: interoperabilidade com bibliotecas COM antigas, ou parsing de JSON com estrutura desconhecida em tempo de compilação. Fora isso, `dynamic` costuma ser sinal de que um design melhor (interfaces, genéricos, ou até `object` com pattern matching) resolveria o mesmo problema com segurança de tipos

---

# ⚠️ Erros comuns

- Usar `dynamic` para "economizar tempo" evitando definir um tipo, perdendo toda a ajuda do compilador e do IntelliSense  
- Retornar `dynamic` de um método público de uma API, tornando o contrato completamente opaco para quem consome  
- Tentar usar um anonymous type como tipo de retorno de método, esbarrando na limitação de escopo local  
- Confundir a performance de `dynamic` com a de tipos estáticos — chamadas via `dynamic` passam por resolução em runtime, mais lentas que chamadas normais  

---

# 📌 Conclusão

- Anonymous types criam objetos sem declarar uma classe, ideais para projeções LINQ locais  
- Anonymous types comparam por valor, mas nunca podem sair do escopo do método onde foram criados  
- `dynamic` desliga a checagem de tipos em tempo de compilação, empurrando erros para o runtime  
- `dynamic` só se justifica em cenários raros: interoperabilidade COM, ou JSON com estrutura verdadeiramente imprevisível  

👉 Entre tuplas, anonymous types e `dynamic`, você agora tem o quadro completo de como o C# lida com dados que não merecem (ou não podem ter) um tipo nomeado formal

---

# 🔥 Próximo passo

Agora que você conhece os recursos mais informais de tipagem do C#, o próximo nível é:

👉 **Fundamentos do C#: A Evolução da Linguagem C#**

Aqui você vai fazer um tour pelas versões do C#, entendendo como a linguagem chegou até os recursos modernos que você usou ao longo de toda esta trilha.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
