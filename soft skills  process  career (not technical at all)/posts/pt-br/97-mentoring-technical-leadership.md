# 🧠 Fundamentos do C#: Mentoria e Liderança Técnica

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Refatoração de código legado  
- Praticamente toda a trilha técnica, do primeiro Hello World (post 14) até arquitetura distribuída  

Você já domina uma quantidade enorme de C# e .NET. Mas o post sobre carreira (50) já apontava: em algum momento, o maior impacto de um desenvolvedor sênior deixa de vir só do código que ele escreve, e passa a vir do código que ele ajuda outros a escrever melhor.

👉 **Vamos aprender Mentoria e Liderança Técnica**

---

# 💡 Code Review como ferramenta de mentoria, não de policiamento

```
❌ "Isso está errado. Troca por switch expression."

✅ "Aqui dá pra simplificar com switch expression (lembra do post 27?) — 
   fica mais direto e evita esse if/else aninhado. O que acha?"
```

👉 Lembra do post sobre Git Workflow (11)? O Pull Request é o ponto de contato mais frequente entre desenvolvedores em um time — a forma como você dá feedback ali ensina (ou afasta) tanto quanto qualquer sessão formal de mentoria

---

# 🏗️ Pareamento: transferindo conhecimento em tempo real

```csharp
// Sênior e júnior no mesmo código, ao vivo
public class ServicoPedido
{
    // "Por que você faria isso como Task<Pedido> em vez de Pedido?"
    // "Porque estamos chamando o banco, lembra do post 26 sobre async/await?"
}
```

👉 Pair programming não é sobre o sênior ditar o código — é sobre pensar em voz alta, explicando o "porquê" por trás de cada decisão, algo que review de PR sozinho raramente consegue capturar

---

# 🎯 Ensinando através de perguntas, não respostas prontas

```
❌ "Usa o Repository Pattern aqui."

✅ "O que acontece se amanhã você precisar trocar o EF Core por outro 
   ORM? Como o código ficaria acoplado a isso hoje?"
   (deixa a pessoa chegar ao Repository Pattern — post 29 — por conta própria)
```

👉 Dar a resposta pronta resolve o problema imediato. Fazer a pergunta certa desenvolve a capacidade de resolver o **próximo** problema sozinho — o objetivo da mentoria é se tornar dispensável, não indispensável

---

# 🔍 Criando padrões de time, não só corrigindo indivíduos

```csharp
// Lembra do post 62, Roslyn Analyzers?
[DiagnosticAnalyzer(LanguageNames.CSharp)]
public class ProibirAsyncVoidAnalyzer : DiagnosticAnalyzer
{
    // Em vez de corrigir "async void" pessoa por pessoa em cada PR,
    // o analyzer aplica o padrão automaticamente para todo o time
}
```

👉 Liderança técnica eficaz não escala através de correções manuais repetidas — ela escala através de ferramentas (analyzers, post 62), documentação (post 49) e processos (Trunk-Based Development, post 65) que aplicam o padrão certo por padrão, não por lembrança individual

---

# 🌱 Delegando com segurança, não com medo

```
❌ "Deixa que eu faço essa parte crítica, é arriscado demais pra você."

✅ "Essa parte é crítica, então vamos parear nela — você escreve,
   eu reviso de perto, e documentamos as decisões (lembra da nota 
   dos autores que fazemos em cada post desta trilha?)"
```

👉 Segurar toda decisão importante cria um gargalo — você. Delegar com apoio (não delegar e desaparecer) cresce a capacidade do time inteiro, e libera espaço para você também crescer para o próximo nível

---

# ⚠️ Erros comuns

- Dar sempre a resposta pronta, criando dependência em vez de autonomia  
- Fazer code review focado só em apontar erros, sem reconhecer o que foi bem feito  
- Confundir liderança técnica com microgerenciamento — ditar cada decisão de implementação  
- Não documentar decisões arquiteturais (lembra dos ADRs implícitos em posts como Clean Architecture, 33?), forçando cada nova pessoa a redescobrir o "porquê" por tentativa e erro  

---

# 📌 Conclusão

- Code review é uma ferramenta de ensino, não só de controle de qualidade  
- Perguntas bem feitas desenvolvem autonomia; respostas prontas criam dependência  
- Padrões de time escalam através de ferramentas e processos, não correção manual repetida  
- Delegar com apoio cresce o time inteiro, sem criar um gargalo em uma única pessoa  

👉 Com mentoria e liderança técnica, seu impacto deixa de ser limitado ao código que você escreve pessoalmente, e passa a se multiplicar através de todo o time ao seu redor

---

# 🔥 Próximo passo

Agora que você multiplica conhecimento dentro do time, o próximo nível é:

👉 **Fundamentos do C#: Contribuindo para Open Source**

Aqui você vai aprender a multiplicar esse mesmo impacto além da sua empresa, contribuindo para o ecossistema .NET que você usa todos os dias.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
