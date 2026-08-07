# 🧠 Fundamentos do C#: WeakReference, Finalizers e o Garbage Collector

⏱️ Tempo de leitura: 8 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- P/Invoke e como memória nativa nunca é rastreada pelo GC  
- O Garbage Collector mencionado dezenas de vezes ao longo desta trilha, sem nunca ser explicado a fundo  

Você sabe que o GC libera memória automaticamente. Mas **como** ele decide o que coletar, **quando** libera memória nativa através de finalizers, e como manter uma referência a um objeto sem impedir que ele seja coletado? Chegou a hora de entender o GC de verdade.

👉 **Vamos entender o Garbage Collector, gerações, finalizers e `WeakReference`**

---

# 💡 Como o GC decide o que coletar

👉 **Um objeto é elegível para coleta quando não existe mais nenhuma referência "forte" alcançável a partir das raízes (roots) do programa** — variáveis locais ativas, campos estáticos, objetos ainda em uso

```csharp
var produto = new Produto("Notebook");
// enquanto "produto" existe e é usado, o objeto NÃO é coletado

produto = null;
// agora não há mais referências fortes — o objeto se torna elegível para coleta
// (mas o GC ainda decide QUANDO efetivamente rodar)
```

👉 O GC não coleta no instante em que a última referência desaparece — ele roda periodicamente, baseado em pressão de memória, não imediatamente após cada `= null`

---

# 🗂️ Gerações: por que o GC não varre tudo toda vez

```
Geração 0: objetos recém-criados, coletados com muita frequência (rápido, barato)
Geração 1: objetos que sobreviveram a uma coleta de Geração 0 (intermediário)
Geração 2: objetos de vida longa, coletados raramente (mais caro, mais raro)
```

👉 **A hipótese geracional: a maioria dos objetos morre jovem.** Variáveis locais de um método, objetos temporários — a maioria nunca sobrevive além de uma única coleta de Geração 0. Em vez de varrer **todo** o heap a cada coleta, o GC foca primeiro na Geração 0, que é rápida e cobre a maior parte do lixo real

```csharp
public void ProcessarPedido()
{
    var dto = new PedidoDto(); // provavelmente morre na Geração 0
    // ... processa ...
} // dto sai de escopo, se torna lixo rapidamente
```

Um objeto que sobrevive a uma coleta de Geração 0 é "promovido" para a Geração 1, e assim por diante — a lógica sendo que se ele já sobreviveu uma vez, é mais provável que continue vivo por mais tempo

---

# 🧹 Finalizers: o último recurso para limpeza

```csharp
public class RecursoNativo
{
    private IntPtr _handleNativo;

    public RecursoNativo()
    {
        _handleNativo = AlocarRecursoNativo();
    }

    ~RecursoNativo() // finalizer — sintaxe de destructor do C++
    {
        LiberarRecursoNativo(_handleNativo);
    }
}
```

👉 **Finalizer (`~NomeDaClasse()`) = um método especial que o GC chama antes de coletar definitivamente um objeto, dando a ele uma última chance de liberar recursos não gerenciados**

Lembra do post sobre `IDisposable`? O `Dispose()` é a forma **determinística** de liberar recursos (você chama explicitamente, `using` cuida disso). O finalizer é uma **rede de segurança**, chamada pelo GC se `Dispose()` nunca foi chamado — mas em um momento imprevisível, e com um custo real de performance

---

# ⚠️ Por que finalizers são caros

```csharp
// Objeto com finalizer: precisa de DUAS coletas para ser totalmente liberado
// 1ª coleta: o GC detecta que é lixo, mas precisa rodar o finalizer primeiro (fila especial)
// 2ª coleta: só depois do finalizer rodar, a memória é realmente liberada
```

👉 Um objeto com finalizer não é coletado imediatamente, mesmo sem referências — ele é colocado em uma fila especial de finalização, processada por uma thread dedicada, e só é verdadeiramente liberado na coleta **seguinte**. Por isso, o padrão recomendado (que você já viu no post sobre `IDisposable`) é implementar `Dispose()` como caminho principal e usar `GC.SuppressFinalize(this)` para dizer ao GC "não precisa rodar o finalizer, eu já limpei tudo manualmente"

---

# 🔗 `WeakReference<T>`: referenciando sem impedir a coleta

```csharp
public class Cache
{
    private readonly Dictionary<int, WeakReference<Produto>> _cache = new();

    public void Adicionar(int id, Produto produto)
    {
        _cache[id] = new WeakReference<Produto>(produto);
    }

    public Produto? ObterOuNulo(int id)
    {
        if (_cache.TryGetValue(id, out var referenciaFraca) &&
            referenciaFraca.TryGetTarget(out var produto))
        {
            return produto; // ainda vivo
        }

        return null; // já foi coletado
    }
}
```

👉 **`WeakReference<T>` = uma referência que aponta para um objeto sem contá-lo como uma referência "forte" — o GC pode coletar o objeto normalmente, mesmo com a `WeakReference` ainda existindo**

Isso é perfeito para caches: você quer reutilizar um objeto se ele ainda estiver na memória, mas **não** quer que o cache seja o motivo de um objeto nunca ser coletado, competindo com a pressão real de memória da aplicação

---

# ⚠️ Erros comuns

- Implementar um finalizer sem necessidade real, adicionando custo de performance para objetos que não seguram nenhum recurso não gerenciado  
- Esquecer `GC.SuppressFinalize(this)` no `Dispose()`, fazendo o objeto ainda passar pela fila de finalização desnecessariamente  
- Usar `WeakReference<T>` para tudo "por segurança", quando na maioria dos casos uma referência forte comum é exatamente o que você quer  
- Chamar `GC.Collect()` manualmente em código de produção, forçando uma coleta completa e cara que o GC teria feito de forma mais eficiente sozinho  

---

# 📌 Conclusão

- Um objeto é elegível para coleta quando não há mais referências fortes alcançáveis  
- Gerações (0, 1, 2) otimizam o GC baseado na hipótese de que a maioria dos objetos morre jovem  
- Finalizers são uma rede de segurança para recursos não gerenciados, mas custam uma coleta extra  
- `WeakReference<T>` referencia um objeto sem impedir sua coleta — ideal para caches  

👉 Entender gerações prepara o terreno para o próximo passo: como o .NET oferece dois modos de GC completamente diferentes, otimizados para cenários opostos

---

# 🔥 Próximo passo

Agora que você entende como o GC decide o que coletar, o próximo nível é:

👉 **Fundamentos do C#: Server GC vs Workstation GC**

Aqui você vai aprender a diferença entre os dois modos de coleta de lixo do .NET, e como escolher o certo para sua aplicação.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
