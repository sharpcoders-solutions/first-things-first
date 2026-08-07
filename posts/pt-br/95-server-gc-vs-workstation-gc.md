# 🧠 Fundamentos do C#: Server GC vs Workstation GC

⏱️ Tempo de leitura: 7 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Gerações, finalizers e `WeakReference` — como o GC decide o que coletar  
- ArrayPool e Object Pooling (post 70), técnicas para reduzir pressão no GC  

Você já sabe como o GC decide **o que** coletar. Mas o .NET oferece dois modos completamente diferentes de **como** essa coleta acontece — uma escolha que pode ter um impacto enorme na performance da sua aplicação, e que a maioria dos desenvolvedores nunca configura conscientemente.

👉 **Vamos entender Server GC vs Workstation GC**

---

# 💡 Os dois modos: otimizados para cenários opostos

👉 **Workstation GC = otimizado para baixa latência, um núcleo de processamento por vez. Server GC = otimizado para alto throughput, usando múltiplos núcleos em paralelo**

```xml
<!-- .csproj -->
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
</PropertyGroup>
```

👉 Essa única configuração no `.csproj` (ou no `runtimeconfig.json`) muda fundamentalmente como o GC opera por baixo dos panos

---

# 🖥️ Workstation GC: o padrão para aplicações desktop e CLI

```
- Um heap único, gerenciado por uma única thread de GC
- Otimizado para pausas curtas e responsividade
- Padrão para: aplicações desktop, CLI, ferramentas de linha de comando
```

👉 Numa aplicação desktop (lembra do Blazor e do .NET MAUI?), uma pausa longa do GC é imediatamente visível ao usuário — a interface trava. Workstation GC prioriza minimizar essas pausas, mesmo que isso signifique coletar com mais frequência

---

# 🖧 Server GC: o padrão para APIs web e serviços

```
- Um heap POR NÚCLEO de processador, cada um com sua própria thread de GC
- Coletas paralelas em múltiplos núcleos simultaneamente
- Padrão para: aplicações ASP.NET Core (já configurado automaticamente)
```

👉 Server GC divide o heap gerenciado em partes, uma por núcleo de CPU disponível, e coleta cada parte em paralelo. Isso significa **muito mais throughput** total (mais alocação/coleta por segundo), ao custo de usar mais memória (um heap por núcleo, não um único heap compartilhado)

```csharp
// Uma API ASP.NET Core já usa Server GC por padrão desde .NET Core 3.0+
// mas configurações antigas, ou hosts customizados, podem precisar do ajuste explícito
```

---

# ⚖️ Comparando os dois lado a lado

| | Workstation GC | Server GC |
|---|---|---|
| Heaps | Um único | Um por núcleo de CPU |
| Otimizado para | Baixa latência | Alto throughput |
| Uso de memória | Menor | Maior |
| Cenário ideal | Desktop, CLI, apps interativas | APIs web, serviços de alto volume |
| Paralelismo na coleta | Limitado | Total, por núcleo |

---

# 🔄 `ConcurrentGarbageCollection`: reduzindo pausas em ambos os modos

```xml
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
</PropertyGroup>
```

👉 **GC concorrente = permite que coletas de Geração 2 (a mais cara, lembra do post anterior?) rodem em uma thread separada, enquanto a aplicação continua executando**, reduzindo (mas não eliminando) pausas perceptíveis mesmo durante coletas grandes. Isso funciona tanto com Workstation quanto com Server GC, e geralmente vale a pena manter ativado

---

# 🎯 Quando ajustar manualmente

```xml
<!-- Container com poucos núcleos alocados: Server GC pode alocar heaps demais -->
<PropertyGroup>
  <ServerGarbageCollection>false</ServerGarbageCollection>
</PropertyGroup>
```

👉 **Regra prática: na maioria dos casos, o padrão do ASP.NET Core (Server GC) já é a escolha certa para APIs.** A exceção mais comum: containers com poucos núcleos de CPU alocados (lembra do post sobre Kubernetes e limites de recursos?) — nesses casos, Server GC pode alocar mais heaps do que faz sentido para a quantidade real de CPU disponível, e Workstation GC (ou Server GC com `HeapCount` limitado) pode ter melhor comportamento

```xml
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <GCHeapCount>2</GCHeapCount> <!-- limita explicitamente o número de heaps -->
</PropertyGroup>
```

---

# 🔍 Medindo o impacto real

```bash
dotnet-counters monitor --process-id <pid> System.Runtime
```

👉 Lembra do `BenchmarkDotNet` (post sobre performance)? Da mesma forma que você não otimiza sem medir, não troque o modo de GC sem medir o impacto real com ferramentas como `dotnet-counters`, observando métricas de tempo de pausa e throughput antes e depois da mudança

---

# ⚠️ Erros comuns

- Assumir que Server GC é sempre melhor "porque é o padrão de produção", sem considerar o ambiente real de execução (containers com poucos núcleos, por exemplo)  
- Trocar o modo de GC sem medir antes e depois, baseado só em intuição  
- Ignorar `ConcurrentGarbageCollection`, perdendo uma redução de pausa praticamente gratuita  
- Configurar Server GC em uma aplicação desktop, aumentando o uso de memória sem nenhum ganho real de throughput relevante para esse cenário  

---

# 📌 Conclusão

- Workstation GC prioriza baixa latência com um único heap; Server GC prioriza throughput com um heap por núcleo  
- ASP.NET Core já usa Server GC por padrão desde .NET Core 3.0+  
- `ConcurrentGarbageCollection` reduz pausas em ambos os modos, rodando coletas grandes em paralelo  
- Containers com poucos núcleos podem se beneficiar de ajustar `GCHeapCount` ou usar Workstation GC  

👉 Com o Garbage Collector totalmente dominado — gerações, finalizers, `WeakReference`, e agora os dois modos de operação — o próximo passo é sair da performance pura e entrar em outro aspecto fundamental: como seu código se comporta em diferentes culturas e idiomas

---

# 🔥 Próximo passo

Agora que você entende os dois modos de coleta de lixo do .NET, o próximo nível é:

👉 **Fundamentos do C#: Culture e Globalização**

Aqui você vai aprender a lidar com formatação de números, datas e strings de forma correta para usuários de diferentes países e idiomas.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
