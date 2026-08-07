# 🧠 Fundamentos do C#: Reflection.Emit e Geração de Código em Runtime (IL Emit)

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- `required` members e `init`-only properties para objetos corretamente inicializados  
- Reflection e atributos customizados, para **inspecionar** tipos em runtime  

Você já usa reflection para inspecionar um tipo — descobrir suas propriedades, chamar seus métodos. Mas existe um passo além: em vez de só **ler** código existente, você pode **gerar código novo**, em memória, enquanto o programa está rodando. É isso que `Reflection.Emit` faz.

👉 **Vamos entender geração de código em runtime com IL Emit**

---

# 💡 O que `Reflection.Emit` realmente faz

👉 **`Reflection.Emit` = uma API que permite construir métodos, tipos e assemblies inteiros em memória, escrevendo IL (Intermediate Language) diretamente, sem passar pelo compilador C#**

```csharp
using System.Reflection;
using System.Reflection.Emit;

var metodo = new DynamicMethod("Somar", typeof(int), new[] { typeof(int), typeof(int) });
var il = metodo.GetILGenerator();

il.Emit(OpCodes.Ldarg_0); // carrega o primeiro parâmetro
il.Emit(OpCodes.Ldarg_1); // carrega o segundo parâmetro
il.Emit(OpCodes.Add);     // soma os dois valores no topo da pilha
il.Emit(OpCodes.Ret);     // retorna o resultado

var somar = (Func<int, int, int>)metodo.CreateDelegate(typeof(Func<int, int, int>));
Console.WriteLine(somar(3, 4)); // 7
```

👉 Esse código constrói, **em runtime**, o equivalente a `int Somar(int a, int b) => a + b;` — cada `Emit` escreve uma instrução IL diretamente, a mesma linguagem intermediária que o compilador C# gera para qualquer método que você escreve normalmente

---

# 🤔 Por que gerar código em runtime, se você pode só escrever C#?

```csharp
// Se você sabe os tipos em tempo de compilação, escreva C# normal:
int Somar(int a, int b) => a + b;

// Reflection.Emit só faz sentido quando o tipo/lógica só existe em runtime:
DynamicMethod CriarAcessorDePropriedade(PropertyInfo propriedade)
{
    // gera dinamicamente um getter otimizado para QUALQUER propriedade,
    // descoberta em tempo de execução, sem reflection lenta a cada chamada
}
```

👉 O caso de uso real: quando você precisa de código que só pode ser determinado em runtime — um ORM que gera acessores otimizados para as propriedades de uma entidade descoberta via reflection, um framework de mock que cria implementações falsas de interfaces sob demanda, um serializador que constrói delegates especializados por tipo. Escrever esse código à mão para cada tipo possível é inviável; gerá-lo dinamicamente resolve isso

---

# ⚡ O problema que `Reflection.Emit` resolve: reflection pura é lenta

```csharp
PropertyInfo propriedade = typeof(Produto).GetProperty("Preco")!;

// ❌ Lento: reflection pura, invocada a cada chamada
object valor = propriedade.GetValue(produto);

// ✅ Rápido: gera um delegate uma vez, reutiliza depois
Func<Produto, decimal> getterCompilado = CriarGetter(propriedade);
decimal valor2 = getterCompilado(produto); // tão rápido quanto uma chamada direta
```

👉 Lembra do post sobre assembly e reflection, e do custo de `MethodInfo.Invoke`? `Reflection.Emit` é exatamente a técnica que bibliotecas como Dapper, AutoMapper e serializadores de alta performance usam para eliminar esse custo: gerar, uma única vez, um delegate compilado que acessa a propriedade diretamente — depois disso, nenhuma reflection acontece mais nas chamadas subsequentes

---

# 🏗️ `AssemblyBuilder` e `TypeBuilder`: gerando tipos inteiros, não só métodos

```csharp
var nomeAssembly = new AssemblyName("AssemblyDinamico");
var assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(nomeAssembly, AssemblyBuilderAccess.Run);
var moduleBuilder = assemblyBuilder.DefineDynamicModule("ModuloDinamico");

var typeBuilder = moduleBuilder.DefineType("ClasseGerada", TypeAttributes.Public);
var campoBuilder = typeBuilder.DefineField("_valor", typeof(int), FieldAttributes.Private);

var propertyBuilder = typeBuilder.DefineProperty("Valor", PropertyAttributes.None, typeof(int), null);
var getterBuilder = typeBuilder.DefineMethod("get_Valor", MethodAttributes.Public, typeof(int), Type.EmptyTypes);

var ilGetter = getterBuilder.GetILGenerator();
ilGetter.Emit(OpCodes.Ldarg_0);
ilGetter.Emit(OpCodes.Ldfld, campoBuilder);
ilGetter.Emit(OpCodes.Ret);

propertyBuilder.SetGetMethod(getterBuilder);

Type tipoGerado = typeBuilder.CreateType()!;
```

👉 `TypeBuilder` sobe um nível: em vez de gerar só um método solto, você constrói uma classe **completa** — campos, propriedades, métodos, tudo em memória, tudo com IL escrito manualmente. `Assembly.LoadFrom` (lembra do post sobre AssemblyLoadContext?) carrega assemblies existentes; `AssemblyBuilder` **cria** um assembly novo, que nunca existiu em disco

---

# ⚖️ `Reflection.Emit` vs Source Generators: a mesma ideia, momentos diferentes

| | Source Generators | `Reflection.Emit` |
|---|---|---|
| Quando gera código | Em tempo de **compilação** | Em **runtime** |
| Compatível com Native AOT | ✅ sim | ❌ não |
| Pode usar informação só disponível em runtime | ❌ não | ✅ sim |
| Debugabilidade | Alta — código gerado é visível como C# | Baixa — IL puro, difícil de inspecionar |

👉 Lembra do post sobre source generators? Ambos resolvem "gerar código em vez de escrever à mão", mas em momentos opostos do ciclo de vida. Sempre que a informação necessária existe em tempo de compilação (tipos conhecidos, atributos declarados), prefira source generators — mais rápido, mais debugável, e compatível com Native AOT. Reserve `Reflection.Emit` para os casos raros onde a decisão realmente só pode ser tomada em runtime

---

# ⚠️ Erros comuns

- Usar `Reflection.Emit` para problemas que um source generator resolveria com muito menos complexidade e melhor debugabilidade  
- Gerar código dinamicamente em um hot path sem cachear o resultado — a geração em si tem custo, então o `DynamicMethod`/`TypeBuilder` deve ser criado uma vez e reutilizado  
- Publicar com Native AOT ou trimming sem saber que `Reflection.Emit` simplesmente não funciona nesses cenários — o runtime precisa poder compilar IL em tempo real, algo que Native AOT elimina  
- Escrever IL manualmente sem testar exaustivamente — um `OpCode` na ordem errada não dá erro de compilação, só falha (ou pior, se comporta de forma sutilmente errada) em runtime  

---

# 📌 Conclusão

- `Reflection.Emit` gera métodos, tipos e assemblies inteiros em memória, escrevendo IL diretamente em runtime  
- O caso de uso real é eliminar o custo de reflection repetida, gerando um delegate compilado uma única vez  
- `AssemblyBuilder`/`TypeBuilder` permitem construir classes completas dinamicamente, não só métodos soltos  
- Prefira source generators sempre que a informação estiver disponível em tempo de compilação — reserve `Reflection.Emit` para decisões que só existem em runtime  

👉 Com geração de código em runtime dominada, você fecha o círculo sobre tipos e reflection — o próximo passo é olhar novamente para uma biblioteca que você já usa desde o início, mas agora nos seus recursos mais avançados: `System.Text.Json`

---

# 🔥 Próximo passo

Agora que você gera código dinamicamente em tempo de execução, o próximo nível é:

👉 **Fundamentos do C#: System.Text.Json Avançado**

Aqui você vai aprender conversores customizados, source generators de serialização, e outros recursos avançados do serializador JSON nativo do .NET.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
