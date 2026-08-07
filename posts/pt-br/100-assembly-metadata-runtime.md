# 🧠 Fundamentos do C#: Assembly, Metadata e Reflection em Profundidade

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Culture e globalização para formatação correta por região  
- Reflection e atributos customizados (post 70), no nível de inspecionar uma classe por vez  

O post 70 te mostrou reflection aplicada a uma classe específica. Agora vamos subir um nível: como inspecionar um **assembly inteiro**, descobrir todos os tipos que ele contém, e entender a estrutura de metadata que torna tudo isso possível.

👉 **Vamos explorar Assembly, Metadata e Reflection em profundidade**

---

# 💡 O que é um Assembly, de verdade?

👉 **Assembly = a unidade de deploy e versionamento do .NET — um arquivo `.dll` ou `.exe` compilado, contendo código IL (Intermediate Language) e metadata sobre tudo que ele define**

```csharp
Assembly assembly = Assembly.GetExecutingAssembly();

Console.WriteLine(assembly.FullName);
// MeuProjeto, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null

Console.WriteLine(assembly.Location);
// C:\projetos\meuprojeto\bin\Debug\net10.0\MeuProjeto.dll
```

👉 Todo `.dll` que você referenciou nesta trilha — Serilog, EF Core, seus próprios projetos — é um assembly. Cada um carrega, além do código compilado, um catálogo completo de metadata: quais tipos existem, quais métodos cada tipo tem, quais atributos foram aplicados

---

# 🔍 Explorando todos os tipos de um assembly

```csharp
Assembly assembly = Assembly.GetExecutingAssembly();

foreach (Type tipo in assembly.GetTypes())
{
    if (tipo.IsPublic && tipo.IsClass)
    {
        Console.WriteLine($"Classe pública: {tipo.FullName}");

        foreach (var metodo in tipo.GetMethods(BindingFlags.Public | BindingFlags.Instance))
        {
            Console.WriteLine($"  Método: {metodo.Name}");
        }
    }
}
```

👉 `GetTypes()` retorna **todos** os tipos definidos naquele assembly — essa é literalmente a técnica por trás de frameworks que "descobrem" automaticamente controllers, handlers do MediatR (post 45), ou validators do FluentValidation, sem que você precise registrar cada um manualmente

---

# 🏷️ Metadata de atributos: indo além do básico

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class ModuloAttribute : Attribute
{
    public string Nome { get; }
    public ModuloAttribute(string nome) => Nome = nome;
}

[Modulo("Vendas")]
public class ServicoPedidos { }

[Modulo("Estoque")]
public class ServicoEstoque { }
```

```csharp
var tiposComModulo = assembly.GetTypes()
    .Where(t => t.GetCustomAttribute<ModuloAttribute>() != null)
    .Select(t => new
    {
        Tipo = t.Name,
        Modulo = t.GetCustomAttribute<ModuloAttribute>()!.Nome
    });

foreach (var item in tiposComModulo)
    Console.WriteLine($"{item.Tipo} pertence ao módulo {item.Modulo}");
```

👉 Lembra do post sobre atributos customizados? Escalando essa técnica para um assembly inteiro, você pode construir sistemas de descoberta automática — agrupar classes por módulo de negócio, gerar documentação, ou validar convenções de arquitetura automaticamente

---

# 🌐 Carregando assemblies dinamicamente

```csharp
Assembly assemblyExterno = Assembly.LoadFrom(@"C:\plugins\meuplugin.dll");

Type[] tipos = assemblyExterno.GetExportedTypes();

foreach (var tipo in tipos.Where(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface))
{
    var instancia = (IPlugin)Activator.CreateInstance(tipo)!;
    instancia.Executar();
}
```

👉 `Assembly.LoadFrom` carrega um assembly que não foi referenciado em tempo de compilação — a base técnica para sistemas de plugins, onde a aplicação principal não conhece as extensões até elas serem carregadas em runtime

---

# ⚙️ `MethodInfo.Invoke`: chamando métodos descobertos dinamicamente

```csharp
Type tipo = Type.GetType("MeuProjeto.ServicoPedidos")!;
object instancia = Activator.CreateInstance(tipo)!;

MethodInfo metodo = tipo.GetMethod("ProcessarPedido")!;
object? resultado = metodo.Invoke(instancia, new object[] { 123 });
```

👉 Isso é reflection no seu nível mais dinâmico: descobrir um tipo por nome (string), criar uma instância, e chamar um método — sem nenhuma referência estática ao tipo em tempo de compilação. Poderoso, mas lembra do post sobre generic math e o custo de reflection? Isso é significativamente mais lento que uma chamada direta, e é exatamente o tipo de código que quebra em Native AOT sem cuidado extra

---

# ⚠️ Erros comuns

- Usar reflection pesada em código de hot path, quando cache de `MethodInfo`/`Type` (calculado uma vez, reutilizado depois) resolveria o problema de performance  
- Carregar assemblies externos sem nenhuma validação, criando um vetor de segurança para código malicioso  
- Esquecer que `Invoke` propaga exceções embrulhadas em `TargetInvocationException`, exigindo `.InnerException` para ver o erro real  
- Depender pesadamente de reflection dinâmica em projetos que também precisam rodar em Native AOT, sem testar essa combinação cedo  

---

# 📌 Conclusão

- Um Assembly é a unidade de deploy do .NET, carregando código IL e metadata completa sobre seus tipos  
- `GetTypes()` permite descobrir todos os tipos de um assembly — a base de frameworks de auto-descoberta  
- Atributos customizados, combinados com reflection em nível de assembly, viabilizam sistemas de convenção e documentação automática  
- `Assembly.LoadFrom` e `MethodInfo.Invoke` permitem carregar e executar código totalmente desconhecido em tempo de compilação  

👉 Falando em carregar assemblies dinamicamente — existe um recurso ainda mais avançado que permite não só carregar, mas também **descarregar** assemblies em runtime, a base de verdadeiros sistemas de plugins

---

# 🔥 Próximo passo

Agora que você explora assemblies e metadata em profundidade, o próximo (e penúltimo) nível é:

👉 **Fundamentos do C#: AssemblyLoadContext e Sistemas de Plugins**

Aqui você vai aprender a carregar e descarregar plugins em tempo de execução, sem precisar reiniciar a aplicação principal.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
