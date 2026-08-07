# 🧠 Fundamentos do C#: AssemblyLoadContext e Sistemas de Plugins

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Assembly, metadata e reflection em profundidade, incluindo `Assembly.LoadFrom`  
- WeakReference e como o GC coleta objetos que não têm mais referências fortes (post 94)  

Você viu como carregar um assembly dinamicamente. Mas existe um problema real com `Assembly.LoadFrom`: uma vez carregado, esse assembly **nunca** é descarregado — ele fica preso na memória para sempre, mesmo que você não precise mais dele. `AssemblyLoadContext` resolve exatamente isso.

👉 **Vamos aprender `AssemblyLoadContext` e construir um sistema de plugins de verdade**

---

# 💡 O problema: assemblies carregados nunca somem

```csharp
// Isso nunca é descarregado, mesmo que "assembly" saia de escopo
Assembly assembly = Assembly.LoadFrom(@"C:\plugins\meuplugin.dll");
```

👉 O contexto de carregamento padrão do .NET (`AssemblyLoadContext.Default`) mantém **todo** assembly carregado nele pelo tempo de vida inteiro da aplicação. Para um plugin que você quer atualizar, substituir, ou remover sem reiniciar a aplicação principal, isso é um problema real

---

# 🏗️ Criando um `AssemblyLoadContext` isolado e descarregável

```csharp
public class ContextoDePlugin : AssemblyLoadContext
{
    private readonly AssemblyDependencyResolver _resolver;

    public ContextoDePlugin(string caminhoPlugin) : base(isCollectible: true)
    {
        _resolver = new AssemblyDependencyResolver(caminhoPlugin);
    }

    protected override Assembly? Load(AssemblyName nomeAssembly)
    {
        string? caminho = _resolver.ResolveAssemblyToPath(nomeAssembly);
        return caminho != null ? LoadFromAssemblyPath(caminho) : null;
    }
}
```

👉 **`isCollectible: true` = o parâmetro que torna esse contexto descarregável pelo Garbage Collector**, diferente do contexto padrão. `AssemblyDependencyResolver` cuida de resolver as dependências do plugin (outros `.dll`s que ele precisa), isoladas do resto da aplicação

---

# 🔌 Carregando e usando um plugin

```csharp
public interface IPlugin
{
    string Nome { get; }
    void Executar();
}
```

```csharp
var contexto = new ContextoDePlugin(@"C:\plugins\MeuPlugin.dll");
Assembly assemblyPlugin = contexto.LoadFromAssemblyPath(@"C:\plugins\MeuPlugin.dll");

Type tipoPlugin = assemblyPlugin.GetTypes()
    .First(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface);

var plugin = (IPlugin)Activator.CreateInstance(tipoPlugin)!;
plugin.Executar();
```

👉 Lembra do post sobre assembly e reflection? A técnica de descoberta de tipos é a mesma — a diferença é que agora o assembly vive em um contexto isolado, que pode ser descartado depois

---

# 🗑️ Descarregando o plugin

```csharp
WeakReference DescarregarContexto(ContexoDePlugin contexto)
{
    var referenciaFraca = new WeakReference(contexto, trackResurrection: true);
    contexto.Unload();
    return referenciaFraca;
}

var referencia = DescarregarContexto(contexto);
contexto = null;
plugin = null;

for (int i = 0; i < 10 && referencia.IsAlive; i++)
{
    GC.Collect();
    GC.WaitForPendingFinalizers();
}

Console.WriteLine(referencia.IsAlive ? "Ainda carregado" : "Descarregado com sucesso");
```

👉 Lembra do post sobre `WeakReference`? Essa é exatamente a ferramenta certa para **verificar** se o descarregamento realmente aconteceu — `Unload()` sinaliza a intenção, mas o descarregamento de verdade só acontece quando o GC coleta **todas** as referências para tipos daquele assembly. Se você ainda mantém uma referência viva (uma variável estática, um evento não desinscrito), o contexto nunca é coletado — o mesmo problema de vazamento que você já viu com `event`s mal gerenciados

---

# ⚠️ O maior risco: referências que impedem o descarregamento

```csharp
// ❌ Isso impede o descarregamento para sempre
public static class RegistroDePlugins
{
    public static List<IPlugin> PluginsAtivos = new(); // referência estática forte
}

RegistroDePlugins.PluginsAtivos.Add(plugin); // o plugin nunca será coletado enquanto estiver aqui
```

👉 Qualquer referência forte a um tipo do plugin — em uma coleção estática, em um evento assinado sem cancelar a inscrição, em um `Timer` ainda ativo — impede o `AssemblyLoadContext` de ser coletado. Antes de chamar `Unload()`, você precisa garantir que **nada** no resto da aplicação ainda referencia tipos daquele plugin

---

# 🎯 Casos de uso reais

- **Editores de código com extensões** (como o próprio Visual Studio) — plugins de terceiros carregados e atualizados sem reiniciar o editor  
- **Sistemas de regras de negócio dinâmicas** — carregar uma nova versão de uma regra sem redeployar toda a aplicação  
- **Ferramentas de teste e hot-reload** — recarregar código modificado sem reiniciar o processo inteiro  

👉 Fora desses cenários específicos, a maioria das aplicações nunca precisa de `AssemblyLoadContext` — é um recurso poderoso, mas de nicho, reservado para quando a arquitetura de plugins é um requisito real do produto

---

# ⚠️ Erros comuns

- Esquecer referências estáticas ou eventos não desinscritos, impedindo o descarregamento de funcionar mesmo depois de chamar `Unload()`  
- Não usar `AssemblyDependencyResolver`, causando conflitos de versão entre as dependências do plugin e as da aplicação principal  
- Assumir que `Unload()` descarrega imediatamente — o processo é assíncrono e depende do GC coletar todas as referências restantes  
- Usar `AssemblyLoadContext` para um cenário simples que `Assembly.LoadFrom` já resolveria, adicionando complexidade sem necessidade real de descarregamento  

---

# 📌 Conclusão

- `Assembly.LoadFrom` carrega assemblies que nunca são descarregados, um problema real para plugins de vida curta  
- `AssemblyLoadContext` com `isCollectible: true` permite descarregar um assembly do contexto, liberando memória  
- `AssemblyDependencyResolver` isola as dependências do plugin do resto da aplicação  
- Qualquer referência forte remanescente impede o descarregamento — o mesmo cuidado que você já viu com `WeakReference` e vazamentos de memória  

👉 Com `AssemblyLoadContext`, você fecha o círculo de tudo que aprendeu sobre assemblies, reflection e o Garbage Collector — construindo sistemas que realmente carregam e descarregam código em tempo de execução

---

# 🔥 Próximo passo

Você está chegando ao fim desta trilha. Depois de dominar plugins dinâmicos, o próximo (penúltimo) nível é:

👉 **Fundamentos do C#: Acompanhando a Evolução do .NET**

Aqui você vai aprender a se manter atualizado com o ciclo de lançamentos do .NET, e onde encontrar informação confiável sobre cada nova versão.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
