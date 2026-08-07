# 🧠 Fundamentos do C#: Configurando Ambiente de Desenvolvimento

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como o computador funciona  
- Git, GitHub e o fluxo de um time  
- O que é IL, CLR e como o .NET executa seu código  

Toda essa teoria só vira prática quando você tem uma coisa instalada na sua máquina:

👉 **O SDK do .NET**

---

# 💡 O que é o SDK?

👉 **SDK = Software Development Kit**

É o pacote completo que você instala para **desenvolver** aplicações .NET. Ele inclui:

- O compilador (`Roslyn`)  
- A CLI (`dotnet`)  
- O Runtime  
- Bibliotecas e templates de projeto  

👉 Se você vai escrever código C#, o SDK é o mínimo necessário

---

# ⚙️ O que é o Runtime?

👉 **Runtime = o que executa aplicações .NET já prontas**

Ele contém apenas o necessário para **rodar** um programa, incluindo:

- O CLR (Common Language Runtime)  
- As bibliotecas base para execução  

👉 O Runtime não compila código — ele só executa o que já foi compilado

---

# 🔀 SDK vs Runtime: qual a diferença?

Essa é uma dúvida comum entre quem está começando.

## 🔹 SDK
- Usado por **quem desenvolve**  
- Inclui o Runtime  
- Permite compilar, testar e publicar  
- Maior em tamanho  

## 🔹 Runtime
- Usado por **quem apenas executa** a aplicação  
- Não compila código  
- Ideal para servidores de produção  
- Menor em tamanho  

👉 Resumindo: **todo SDK inclui um Runtime, mas nem todo Runtime inclui um SDK**

---

# 🏗️ Instalando o SDK do .NET

O processo é simples e multiplataforma:

1. Acesse [dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)  
2. Baixe o instalador do **SDK** (não o Runtime) para o seu sistema operacional  
3. Execute o instalador  
4. Reinicie o terminal  

## 🔹 Confirmando a instalação

Depois de instalar, abra o terminal e rode:

```bash
dotnet --version
```

👉 Isso deve mostrar a versão do SDK instalada

Para ver tudo o que está na sua máquina (SDKs e Runtimes):

```bash
dotnet --list-sdks
dotnet --list-runtimes
```

---

# 🧱 Múltiplas versões do SDK

É comum ter mais de uma versão do .NET instalada ao mesmo tempo — por exemplo, um projeto legado em .NET 6 e um novo em .NET 8.

👉 Isso não é um problema: o `dotnet` sabe escolher a versão certa

Quando você precisa travar uma versão específica para um projeto, existe o arquivo `global.json`:

```json
{
  "sdk": {
    "version": "8.0.100"
  }
}
```

👉 Ele garante que o projeto sempre use a versão definida, mesmo com várias instaladas

---

# ⚠️ Erros comuns

- Instalar apenas o Runtime achando que dá pra desenvolver com ele  
- Confundir a versão do SDK com a versão do projeto (`TargetFramework`)  
- Não reiniciar o terminal após instalar, e achar que "não funcionou"  
- Ignorar múltiplas versões instaladas e não saber qual está sendo usada  

---

# 📌 Conclusão

- O **SDK** é para desenvolver: compilar, testar, publicar  
- O **Runtime** é para executar: só roda o que já está pronto  
- `dotnet --version` confirma sua instalação  
- `global.json` trava a versão do SDK usada em um projeto  

👉 Com o SDK instalado, seu ambiente está pronto para escrever código de verdade

---

# 🔥 Próximo passo

Agora que seu ambiente está configurado, o próximo nível é:

👉 **Fundamentos do C#: Seu Primeiro Programa (Hello World e Estrutura do Projeto)**

Aqui você vai criar, rodar e entender seu primeiro projeto C# na prática.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
