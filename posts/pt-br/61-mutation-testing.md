# 🧠 Fundamentos do C#: Mutation Testing

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Coleções concorrentes para múltiplas threads acessando os mesmos dados  
- Cobertura de código como métrica de qualidade dos testes  

Sua suíte de testes tem 95% de cobertura. Isso significa que seus testes são bons? Não necessariamente — cobertura só mede se a linha **executou**, não se o teste realmente verificou o comportamento certo. Mutation Testing responde essa pergunta.

👉 **Vamos aprender Mutation Testing**

---

# 💡 O problema da cobertura vazia

```csharp
public class CalculadoraDesconto
{
    public decimal Calcular(decimal valor)
    {
        if (valor > 100)
            return valor * 0.9m;

        return valor;
    }
}
```

```csharp
[Fact]
public void Calcular_DeveExecutarSemErro()
{
    var resultado = _calculadora.Calcular(150);
    Assert.NotNull(resultado); // ❌ não verifica o valor real
}
```

👉 Esse teste dá 100% de cobertura na classe inteira, mas não verificaria nada se alguém trocasse `0.9m` por `0.5m` por engano — o `Assert.NotNull` sempre passa

---

# 🧬 O que é Mutation Testing?

👉 **Mutation Testing = alterar propositalmente pequenos trechos do código (mutantes) e verificar se os testes detectam a mudança**

Exemplos de mutações automáticas:

```csharp
// Original
if (valor > 100)

// Mutante 1: troca o operador
if (valor >= 100)

// Mutante 2: troca a constante
if (valor > 0)

// Mutante 3: nega a condição
if (valor < 100)
```

Se os testes continuam passando com qualquer uma dessas mutações, o mutante **sobreviveu** — sinal de que aquele trecho não está realmente sendo testado.

---

# 🏗️ Usando o Stryker.NET

```bash
dotnet tool install -g dotnet-stryker
```

```bash
cd MeuProjeto.Tests
dotnet stryker
```

```
Mutation testing report:
  Total mutantes: 42
  Mortos: 35 (83%)
  Sobreviventes: 7 (17%)

  Mutation Score: 83%
```

👉 Diferente de cobertura de linha, o Mutation Score mede a **qualidade** dos asserts — quantos dos bugs simulados os testes realmente pegaram

---

# 🎯 Corrigindo um teste fraco

```csharp
// ❌ Antes: mutante sobrevive
[Fact]
public void Calcular_DeveExecutarSemErro()
{
    var resultado = _calculadora.Calcular(150);
    Assert.NotNull(resultado);
}

// ✅ Depois: mutante morre
[Fact]
public void Calcular_ComValorAcimaDe100_DeveAplicar10PorCentoDeDesconto()
{
    var resultado = _calculadora.Calcular(150);
    Assert.Equal(135m, resultado);
}

[Fact]
public void Calcular_ComValorNoLimiteDe100_NaoDeveAplicarDesconto()
{
    var resultado = _calculadora.Calcular(100);
    Assert.Equal(100m, resultado);
}
```

👉 O segundo teste, testando exatamente o limite `100`, é o que mata o mutante que trocou `>` por `>=` — lembra dos testes de borda que discutimos desde o post 30? Mutation testing força você a escrevê-los

---

# ⚠️ Erros comuns

- Rodar mutation testing em todo o código de uma vez, tornando o processo lento demais para rodar com frequência  
- Perseguir 100% de Mutation Score cegamente, quando alguns mutantes são equivalentes e realmente não importam  
- Usar mutation testing como substituto de cobertura, quando eles respondem perguntas diferentes e complementares  
- Rodar em cada commit no CI, quando o custo computacional é alto — melhor usar periodicamente ou em código crítico  

---

# 📌 Conclusão

- Cobertura de código mede execução, não qualidade dos testes  
- Mutation Testing altera o código propositalmente e verifica se os testes detectam a mudança  
- Mutantes sobreviventes indicam testes fracos, sem asserts que realmente validam o comportamento  
- Stryker.NET automatiza esse processo para projetos C#  

👉 Com Mutation Testing, você para de perguntar "meus testes executaram?" e passa a perguntar "meus testes realmente pegam bugs?"

---

# 🔥 Próximo passo

Agora que você sabe medir a qualidade real dos seus testes, o próximo nível é:

👉 **Fundamentos do C#: Roslyn Analyzers**

Aqui você vai aprender a criar suas próprias regras de análise estática, aplicadas automaticamente durante a compilação.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
