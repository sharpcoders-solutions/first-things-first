# 🧠 Fundamentos do C#: Streams e Manipulação de Arquivos em C#

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- IDisposable e o padrão Dispose  
- Coleções (post 18) — arrays, listas, dicionários em memória  

Tudo que você guardou em memória até agora desaparece quando a aplicação encerra. Chegou a hora de ler e escrever dados que sobrevivem além do processo — arquivos em disco.

👉 **Vamos aprender Streams e Manipulação de Arquivos**

---

# 💡 O que é um Stream?

👉 **Stream = uma sequência de bytes, lida ou escrita de forma incremental, sem precisar carregar tudo na memória de uma vez**

```csharp
using FileStream stream = new FileStream("dados.txt", FileMode.Open);
```

👉 Lembra do post anterior sobre IDisposable? `FileStream` é o exemplo clássico de recurso não gerenciado — sempre usado dentro de um `using`

---

# 🏗️ Formas simples: `File` para casos comuns

```csharp
// Escrever texto
File.WriteAllText("log.txt", "Aplicação iniciada");

// Ler texto
string conteudo = File.ReadAllText("log.txt");

// Ler linha por linha, sem carregar o arquivo inteiro
foreach (var linha in File.ReadLines("log.txt"))
{
    Console.WriteLine(linha);
}
```

👉 A classe estática `File` abstrai o `FileStream` por baixo dos panos para operações simples — para arquivos pequenos, você raramente precisa manipular o Stream diretamente

---

# 🎯 `StreamReader` e `StreamWriter`: texto sobre streams de bytes

```csharp
using var writer = new StreamWriter("pedidos.csv");
writer.WriteLine("Id,Cliente,Valor");
foreach (var pedido in pedidos)
{
    writer.WriteLine($"{pedido.Id},{pedido.Cliente},{pedido.Valor}");
}
```

```csharp
using var reader = new StreamReader("pedidos.csv");
string? linha;
while ((linha = reader.ReadLine()) is not null)
{
    Console.WriteLine(linha);
}
```

👉 Um arquivo é fundamentalmente bytes — `StreamReader`/`StreamWriter` adicionam a camada de codificação de texto (UTF-8, por padrão) por cima do `FileStream` bruto

---

# 📦 Processando arquivos grandes sem estourar memória

```csharp
// ❌ Carrega o arquivo inteiro na memória de uma vez
var todasAsLinhas = File.ReadAllLines("arquivo-de-10gb.csv");

// ✅ Processa uma linha por vez, memória constante
using var reader = new StreamReader("arquivo-de-10gb.csv");
string? linha;
while ((linha = reader.ReadLine()) is not null)
{
    ProcessarLinha(linha);
}
```

👉 Lembra do post sobre ArrayPool (70)? O mesmo princípio de evitar alocação desnecessária se aplica aqui — ler um arquivo de 10GB inteiro para memória para processar linha por linha é desperdício; streams processam de forma incremental

---

# ⚡ Operações assíncronas em I/O de arquivo

```csharp
public async Task SalvarLogAsync(string mensagem)
{
    await using var writer = new StreamWriter("log.txt", append: true);
    await writer.WriteLineAsync(mensagem);
}
```

👉 Lembra do post sobre async/await? I/O de disco, assim como I/O de rede, é um candidato natural para `async` — a thread não fica bloqueada esperando o disco responder, ela é liberada para outro trabalho enquanto isso

---

# 🔄 Copiando streams: de arquivo para arquivo, ou entre tipos

```csharp
using var origem = new FileStream("origem.zip", FileMode.Open);
using var destino = new FileStream("destino.zip", FileMode.Create);

await origem.CopyToAsync(destino);
```

👉 `CopyToAsync` funciona entre **qualquer** combinação de streams — arquivo para arquivo, arquivo para `HttpResponse`, memória para arquivo — porque todos herdam da mesma classe base `Stream`, independente da origem dos dados

---

# ⚠️ Erros comuns

- Usar `File.ReadAllText`/`ReadAllLines` em arquivos grandes, carregando gigabytes na memória desnecessariamente  
- Esquecer `using` em `FileStream`, `StreamReader` ou `StreamWriter`, deixando o arquivo travado para outros processos  
- Usar métodos síncronos (`ReadLine()`, `WriteLine()`) em código de API, bloqueando a thread desnecessariamente  
- Não especificar o encoding explicitamente ao ler arquivos de fontes externas, causando caracteres corrompidos (acentos, por exemplo)  

---

# 📌 Conclusão

- Streams processam dados incrementalmente, sem exigir que tudo caiba na memória de uma vez  
- `StreamReader`/`StreamWriter` adicionam a camada de texto sobre o `FileStream` de bytes  
- Operações de I/O de arquivo deveriam ser `async`, seguindo o mesmo princípio do I/O de rede  
- `CopyToAsync` funciona entre qualquer combinação de streams, não só arquivos  

👉 Com Streams, você processa arquivos de qualquer tamanho de forma eficiente, sem comprometer a memória da aplicação

---

# 🔥 Próximo passo

Agora que você sabe manipular arquivos com eficiência, o próximo nível é:

👉 **Fundamentos do C#: Logging Estruturado com Serilog**

Aqui você vai aprender a registrar o que acontece na sua aplicação de forma estruturada e pesquisável.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
