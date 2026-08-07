# 🧠 Fundamentos do C#: Native AOT

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Source Generators para gerar código em tempo de compilação  
- Performance em C# (Span, Memory e Benchmarking)  

Desde o post sobre arquitetura .NET (post 12), você sabe que o código C# é compilado para IL, e o CLR o traduz para código de máquina via JIT, em tempo de execução. Native AOT elimina essa etapa intermediária completamente.

👉 **Vamos aprender Native AOT**

---

# 💡 Relembrando: como o .NET normalmente funciona

```
Código C# → IL (post 12) → CLR carrega e faz JIT → Código de máquina (em runtime)
```

👉 O JIT (Just-In-Time) compila o IL para código de máquina na hora que a aplicação inicia — é flexível, mas custa tempo de inicialização e memória

---

# 🏗️ Como o Native AOT muda esse fluxo

```
Código C# → IL → Compilação AOT (durante o build) → Executável nativo, sem runtime
```

```xml
<!-- .csproj -->
<PropertyGroup>
  <PublishAot>true</PublishAot>
</PropertyGroup>
```

```bash
dotnet publish -c Release -r linux-x64
```

👉 O resultado é um executável nativo autocontido — sem depender do runtime .NET instalado na máquina de destino, sem etapa de JIT, iniciando quase instantaneamente

---

# ⚡ Os ganhos reais

| Métrica | JIT tradicional | Native AOT |
|---|---|---|
| Tempo de inicialização | ~200-500ms | ~10-50ms |
| Uso de memória | Maior (runtime completo) | Menor (só o necessário) |
| Tamanho do deploy | Requer runtime instalado | Executável autocontido |

👉 Lembra do post sobre Docker (35)? Uma imagem com Native AOT pode ser drasticamente menor — sem precisar da imagem base do .NET runtime, só o executável nativo e suas dependências mínimas

---

# 🎯 Onde isso importa de verdade

- **Serverless / Azure Functions / AWS Lambda** (próximos posts) — cold start rápido é crítico quando você paga por tempo de execução  
- **CLIs distribuídos** — ferramentas de linha de comando que precisam iniciar instantaneamente  
- **Containers em Kubernetes** com autoscaling agressivo, onde novos pods precisam ficar prontos rapidamente  

---

# ⚠️ As limitações

```csharp
// ❌ Reflection dinâmica não funciona bem com Native AOT
var tipo = Type.GetType("MeuNamespace." + nomeDinamico);
var instancia = Activator.CreateInstance(tipo);
```

👉 Lembra do post sobre Reflection (67)? Native AOT precisa saber, **em tempo de compilação**, quais tipos existem — código que descobre tipos dinamicamente em runtime (Reflection não-trimmable) não funciona bem, ou precisa de anotações especiais

```csharp
// ✅ Source Generators (post 68) são os melhores amigos do Native AOT
[JsonSerializable(typeof(Pedido))]
public partial class ContextoSerializacao : JsonSerializerContext { }
```

👉 Não é coincidência que falamos de Source Generators no post anterior — eles resolvem exatamente o que Reflection não consegue fazer em tempo de compilação, e são a base para bibliotecas compatíveis com AOT

---

# ⚠️ Erros comuns

- Usar Native AOT em toda aplicação sem avaliar compatibilidade das bibliotecas usadas — nem toda dependência funciona bem com trimming  
- Esperar os mesmos ganhos em aplicações com pouca carga de inicialização — o benefício é maior justamente em cold starts frequentes  
- Ignorar os warnings de trimming durante o build, que sinalizam código potencialmente incompatível com AOT  
- Achar que Native AOT sempre é "melhor" — para aplicações web de longa duração, o JIT tradicional com tiered compilation às vezes ainda vence em throughput  

---

# 📌 Conclusão

- Native AOT compila C# direto para código nativo, eliminando o JIT em runtime  
- O ganho principal é inicialização quase instantânea e menor uso de memória  
- Reflection dinâmica tem suporte limitado — Source Generators são o caminho recomendado  
- É especialmente valioso em serverless, CLIs e containers com autoscaling agressivo  

👉 Com Native AOT, aplicações .NET competem de igual para igual com linguagens compiladas nativamente em cenários onde cold start é crítico

---

# 🔥 Próximo passo

Agora que você sabe compilar para código nativo, o próximo nível é:

👉 **Fundamentos do C#: ArrayPool e Object Pooling**

Aqui você vai aprender a reduzir a pressão sobre o Garbage Collector reutilizando objetos em vez de alocar novos constantemente.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
