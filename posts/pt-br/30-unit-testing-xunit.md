# 🧠 Fundamentos do C#: Testes Unitários com xUnit

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Os cinco princípios SOLID  
- Design patterns essenciais (Singleton, Factory, Strategy, Repository)  

Você já sabe escrever código bem projetado. Mas como ter **certeza** de que ele continua funcionando depois que outra pessoa (ou você mesmo, seis meses depois) mexer nele?

👉 **É para isso que existem os testes automatizados**

Esse é um marco importante: o post de número 30 desta trilha. A partir de agora, você não escreve só código — você escreve código que **prova** que funciona.

---

# 💡 Por que testar?

👉 **Teste unitário = código que verifica automaticamente se outro código se comporta como esperado**

Sem testes, cada mudança é um salto de fé. Com testes:

✅ Você refatora sem medo de quebrar algo escondido  
✅ Bugs são pegos antes de chegar em produção  
✅ O teste vira **documentação viva** de como o código deveria se comportar  

👉 Lembra do SOLID e dos design patterns? Código desacoplado (graças a interfaces e DIP) é exatamente o que torna os testes fáceis de escrever — os dois posts anteriores prepararam o terreno para este

---

# 🏗️ Configurando o projeto de testes

```bash
dotnet new xunit -o MeuProjeto.Tests
cd MeuProjeto.Tests
dotnet add reference ../MeuProjeto/MeuProjeto.csproj
```

👉 O template `xunit` já vem com tudo configurado: o framework de testes, o executor e as dependências necessárias

---

# 🧪 Seu primeiro teste

```csharp
public class CalculadoraTests
{
    [Fact]
    public void Somar_DoisNumerosPositivos_RetornaSomaCorreta()
    {
        // Arrange
        var calculadora = new Calculadora();

        // Act
        int resultado = calculadora.Somar(2, 3);

        // Assert
        Assert.Equal(5, resultado);
    }
}
```

## 🔹 O padrão AAA

- **Arrange** → prepara os dados e objetos necessários  
- **Act** → executa a ação que está sendo testada  
- **Assert** → verifica se o resultado é o esperado  

👉 Todo teste bem escrito segue essa estrutura, mesmo sem os comentários explícitos

## 🔹 Convenção de nomes

`MetodoTestado_Cenario_ResultadoEsperado` — o nome do teste já conta a história do que está sendo verificado, sem precisar abrir o código

---

# 🎯 `[Fact]` vs `[Theory]`: testando múltiplos cenários

```csharp
public class CalculadoraTests
{
    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(-1, 1, 0)]
    [InlineData(0, 0, 0)]
    public void Somar_VariosValores_RetornaSomaCorreta(int a, int b, int esperado)
    {
        var calculadora = new Calculadora();

        int resultado = calculadora.Somar(a, b);

        Assert.Equal(esperado, resultado);
    }
}
```

👉 `[Fact]` testa **um** cenário fixo. `[Theory]` + `[InlineData]` roda o **mesmo teste** várias vezes, com entradas diferentes — evita duplicar código para cada caso

---

# ✅ Assertions mais usadas

```csharp
Assert.Equal(5, resultado);                 // valores iguais
Assert.True(condicao);                       // condição verdadeira
Assert.Null(objeto);                         // objeto é nulo
Assert.NotNull(objeto);                      // objeto não é nulo

Assert.Throws<InvalidOperationException>(() =>
{
    conta.Sacar(1000); // deve lançar exceção
});
```

👉 `Assert.Throws` é essencial para testar os cenários de erro que você aprendeu no post sobre exceções — um bom conjunto de testes cobre tanto o caminho feliz quanto as falhas esperadas

---

# 🎭 Testando com dependências: Mocking

Lembra do exemplo de `INotificador` e `ProcessadorPedido` do post sobre SOLID (Dependency Inversion)? É exatamente esse desacoplamento que torna o teste possível:

```csharp
class NotificadorFalso : INotificador
{
    public bool FoiChamado { get; private set; }

    public void Enviar(string mensagem)
    {
        FoiChamado = true;
    }
}
```

```csharp
[Fact]
public void Processar_PedidoValido_ChamaNotificador()
{
    // Arrange
    var notificadorFalso = new NotificadorFalso();
    var processador = new ProcessadorPedido(notificadorFalso);

    // Act
    processador.Processar();

    // Assert
    Assert.True(notificadorFalso.FoiChamado);
}
```

👉 `ProcessadorPedido` recebe `INotificador` pelo construtor (Dependency Inversion), então no teste você passa uma versão **falsa**, sem precisar enviar e-mail ou SMS de verdade

## 🔹 Bibliotecas de mock (Moq)

Escrever uma classe falsa manualmente funciona, mas para interfaces maiores fica repetitivo. A biblioteca **Moq** gera esses "dublês" automaticamente:

```csharp
var mockNotificador = new Mock<INotificador>();
var processador = new ProcessadorPedido(mockNotificador.Object);

processador.Processar();

mockNotificador.Verify(n => n.Enviar(It.IsAny<string>()), Times.Once);
```

👉 `Verify` confirma que o método `Enviar` foi chamado exatamente uma vez — sem precisar escrever uma classe falsa manualmente

---

# ⚠️ Erros comuns

- Testar detalhes de implementação em vez de comportamento (isso quebra o teste a cada refatoração, mesmo sem bug real)  
- Criar testes que dependem uns dos outros (um teste nunca deveria precisar que outro rode antes)  
- Só testar o caminho feliz, ignorando erros e casos-limite  
- Pular testes em código "simples demais para quebrar" — esse é exatamente o código que costuma quebrar sem aviso  
- Nomear testes de forma genérica (`Teste1`, `TesteSomar`) em vez de descrever cenário e resultado esperado  

---

# 📌 Conclusão

- Testes unitários automatizam a verificação de que seu código se comporta como esperado  
- O padrão AAA (Arrange, Act, Assert) estrutura qualquer teste  
- `[Fact]` testa um cenário fixo; `[Theory]` + `[InlineData]` testa múltiplos cenários com o mesmo código  
- Código desacoplado via interfaces (SOLID) é o que torna mocks e testes possíveis  
- `Assert.Throws` garante que erros esperados realmente acontecem  

👉 Com testes automatizados, seu código para de depender de "parece que está funcionando" e passa a **provar** que está

---

# 🔥 Próximo passo

Agora que você sabe validar seu código automaticamente, o próximo nível é:

👉 **Fundamentos do C#: Criando sua Primeira API com ASP.NET Core**

Aqui você vai sair do código isolado e construir sua primeira aplicação web real, aplicando tudo que aprendeu até aqui.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
