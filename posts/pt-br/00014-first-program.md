# 🧠 Fundamentos do C#: Seu Primeiro Programa (Hello World e Estrutura do Projeto)

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- O que é IL, CLR e como o .NET executa seu código  
- A diferença entre SDK e Runtime  
- Como instalar o SDK e confirmar a instalação  

Chegou a hora de sair da teoria:

👉 **Vamos criar, rodar e entender seu primeiro projeto C#**

---

# 💡 Criando um projeto

Com o SDK instalado, você já pode criar um projeto direto pelo terminal:

```bash
dotnet new console -o MeuPrimeiroApp
cd MeuPrimeiroApp
```

👉 O comando `dotnet new` usa um **template** para gerar toda a estrutura inicial do projeto pra você

---

# 🏗️ Entendendo a estrutura gerada

Depois do comando, você verá alguns arquivos e pastas:

## 🔹 `MeuPrimeiroApp.csproj`
- Arquivo de configuração do projeto  
- Define o **TargetFramework** (ex: `net8.0`)  
- Lista dependências (pacotes NuGet)  

## 🔹 `Program.cs`
- Onde seu código realmente vive  
- É o ponto de entrada da aplicação  

## 🔹 Pastas `bin/` e `obj/`
- Geradas automaticamente durante o build  
- `obj/` guarda arquivos intermediários da compilação  
- `bin/` guarda o resultado final (o executável e o IL compilado)  

👉 Você nunca deve versionar `bin/` e `obj/` no Git — eles são gerados a cada build

---

# 📄 O que tem dentro do `Program.cs`?

Em versões modernas do C#, o Hello World é surpreendentemente simples:

```csharp
Console.WriteLine("Hello, World!");
```

👉 Isso é chamado de **top-level statements** — o C# moderno elimina a "cerimônia" de escrever `class Program` e `static void Main` manualmente

## 🔹 Por baixo dos panos

O compilador ainda gera essa estrutura tradicional para você, só que de forma automática:

```csharp
namespace MeuPrimeiroApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

👉 Entender essa versão "completa" ajuda bastante quando você começar a ler código mais antigo ou mais explícito

---

# ⚙️ Rodando o programa

Para executar seu projeto:

```bash
dotnet run
```

O que acontece por trás:

1. O `dotnet` compila seu código-fonte em **IL**  
2. O CLR converte esse IL em código de máquina via **JIT**  
3. O programa roda e imprime o resultado no terminal  

👉 Você já viu esse fluxo no post sobre arquitetura .NET — agora está vendo ele na prática

---

# 🔧 Build vs Run vs Publish

Três comandos que parecem parecidos, mas têm propósitos diferentes:

## 🔹 `dotnet build`
- Compila o projeto  
- Gera os arquivos em `bin/`  
- Não executa nada  

## 🔹 `dotnet run`
- Compila (se necessário) **e** executa  
- Ideal durante o desenvolvimento  

## 🔹 `dotnet publish`
- Gera uma versão pronta para **deploy**  
- Pode incluir o runtime (self-contained) ou não  

👉 No dia a dia de desenvolvimento, `dotnet run` é o seu melhor amigo

---

# ⚠️ Erros comuns

- Editar arquivos dentro de `bin/` ou `obj/` achando que isso muda o comportamento do app  
- Commitar `bin/` e `obj/` no Git (sempre use um `.gitignore`)  
- Confundir `dotnet build` com `dotnet run` e não entender por que "nada aconteceu"  
- Achar que top-level statements são uma linguagem diferente, quando é só uma forma mais enxuta de escrever a mesma coisa  

---

# 📌 Conclusão

- `dotnet new console` cria a estrutura inicial de um projeto  
- `Program.cs` é onde seu código vive, com **top-level statements** simplificando o Hello World  
- `dotnet run` compila e executa o projeto  
- `bin/` e `obj/` são gerados automaticamente e não devem ser versionados  

👉 Agora você já sabe criar, rodar e entender um projeto C# do zero

---

# 🔥 Próximo passo

Agora que você já rodou seu primeiro programa, o próximo nível é:

👉 **Fundamentos do C#: Variáveis, Tipos e Sintaxe Básica**

Aqui você vai aprender a armazenar e manipular dados de verdade no seu código.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
