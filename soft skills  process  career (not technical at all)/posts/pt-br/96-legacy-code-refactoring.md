# 🧠 Fundamentos do C#: Refatoração de Código Legado

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Clean Code, os princípios de código legível  
- Testes (post 30) e Clean Architecture (post 33), aplicados desde o início em código novo  

Tudo que você aprendeu até aqui parte de projetos começando do zero. Mas a maior parte da carreira de um desenvolvedor sênior é gasta em sistemas que já existem — sem testes, sem arquitetura clara, e que ninguém tem coragem de mexer. Isso é código legado.

👉 **Vamos aprender Refatoração de Código Legado**

---

# 💡 Definindo "código legado"

👉 **Código legado = código sem testes automatizados, independente da idade**

```csharp
// Isso É código legado, mesmo escrito ontem
public void ProcessarPedido(Pedido p)
{
    // 200 linhas sem nenhum teste cobrindo esse comportamento
}
```

👉 Michael Feathers (autor de referência no assunto) define assim: se não há testes, você não sabe se uma mudança quebra algo — e é exatamente essa incerteza que torna código legado assustador de mexer

---

# 🏗️ O primeiro passo: criar uma rede de segurança

```csharp
// Teste de caracterização: documenta o comportamento ATUAL, mesmo que estranho
[Fact]
public void ProcessarPedido_ComportamentoAtual_DocumentaEstadoExistente()
{
    var pedido = new Pedido { Valor = 100, Status = "X" }; // status "X" é estranho, mas é o que existe

    var resultado = _servico.ProcessarPedido(pedido);

    Assert.Equal("Y", resultado.Status); // documenta o que o código realmente faz hoje
}
```

👉 Lembra do post 30? Antes de refatorar, você escreve testes que capturam o comportamento **atual**, mesmo que pareça errado — o objetivo aqui não é corrigir bugs ainda, é ter uma rede de segurança que avisa se você mudar algo sem querer

---

# 🎯 Refatorando em passos pequenos e seguros

```csharp
// Passo 1: extrai um método, sem mudar comportamento
public void ProcessarPedido(Pedido pedido)
{
    ValidarPedido(pedido); // extraído, mas ainda faz exatamente a mesma coisa
    // resto do código original, intocado
}

// Passo 2: com o teste de caracterização passando, agora sim melhora a lógica
private void ValidarPedido(Pedido pedido)
{
    if (pedido.Valor <= 0)
        throw new ArgumentException("Valor inválido");
}
```

👉 Cada passo é pequeno o suficiente para rodar os testes de caracterização depois e confirmar que nada quebrou — refatoração legada é sobre disciplina de passos pequenos, não reescrita heroica de uma vez

---

# 🔌 Seams: pontos de injeção em código não testável

```csharp
// ❌ Impossível de testar isoladamente
public class ServicoPedido
{
    public void Processar(Pedido pedido)
    {
        var cliente = new HttpClient(); // dependência concreta, direto no meio do código
        cliente.PostAsJsonAsync("https://api-externa.com/pedidos", pedido);
    }
}

// ✅ Um "seam" criado via injeção de dependência (lembra do post sobre DI?)
public class ServicoPedido
{
    private readonly HttpClient _cliente;

    public ServicoPedido(HttpClient cliente) // agora pode injetar um mock nos testes
    {
        _cliente = cliente;
    }
}
```

👉 Um "seam" (costura) é um ponto onde você consegue alterar o comportamento sem editar o código original — geralmente introduzido via injeção de dependência (post sobre ASP.NET Core), permitindo finalmente testar algo que antes era impossível de isolar

---

# ⚖️ Refatorar vs Reescrever

## 🔹 Refatoração incremental
- Sistema continua funcionando durante todo o processo  
- Risco distribuído em pequenos passos  
- Ganhos aparecem gradualmente  

## 🔹 Reescrita completa
- Alto risco — "big bang rewrites" frequentemente falham ou demoram muito mais que o estimado  
- Só faz sentido quando o sistema é pequeno o suficiente, ou verdadeiramente irrecuperável  

👉 A recomendação quase universal na indústria é: prefira refatoração incremental. Reescritas completas têm um histórico ruim de estourar prazo e orçamento

---

# ⚠️ Erros comuns

- Tentar refatorar e adicionar funcionalidade nova ao mesmo tempo, misturando dois tipos de mudança e dificultando isolar a causa se algo quebrar  
- Refatorar sem testes de caracterização primeiro, removendo a única rede de segurança disponível  
- Fazer mudanças grandes demais de uma vez, tornando difícil identificar o que exatamente causou uma regressão  
- Ignorar código legado até que ele se torne uma crise — refatoração contínua e pequena é mais barata que uma reescrita de emergência  

---

# 📌 Conclusão

- Código legado é definido pela ausência de testes, não pela idade  
- Testes de caracterização documentam o comportamento atual antes de qualquer mudança  
- Seams via injeção de dependência tornam código antes intestável em testável  
- Refatoração incremental, em passos pequenos, é preferível a reescritas completas na maioria dos casos  

👉 Com essas técnicas, código legado deixa de ser um território assustador e vira um sistema que você pode melhorar com confiança, um passo seguro de cada vez

---

# 🔥 Próximo passo

Agora que você sabe trabalhar com código legado, o próximo nível é:

👉 **Fundamentos do C#: Mentoria e Liderança Técnica**

Aqui você vai aprender a multiplicar seu impacto além do próprio código, ajudando outros desenvolvedores a crescer.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
