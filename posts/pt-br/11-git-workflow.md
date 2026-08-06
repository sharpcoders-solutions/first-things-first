# 🧠 Git Workflow: Branches, Pull Requests e o Fluxo de um Time

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Como o computador funciona  
- Como programas executam  
- Threads e concorrência  
- Como usar IDEs  
- O que são Git e GitHub  

Agora você já sabe **fazer commits** e **subir código**. Mas isso é só o básico.

👉 **O próximo nível é entender como um time profissional realmente trabalha com Git**

---

# 💡 Por que não trabalhar direto na `main`?

A `main` (ou `master`, em projetos mais antigos) representa o código estável — o que está em produção ou pronto para ir.

👉 Editar direto nela é arriscado: qualquer erro afeta todo mundo na hora

Por isso, o fluxo básico de qualquer time segue quatro passos:

1. Criar uma **branch** a partir da `main`  
2. Trabalhar isolado nela  
3. Abrir um **pull request** quando terminar  
4. Revisar, ajustar e só então integrar de volta  

👉 Esse isolamento permite várias pessoas mexerem no mesmo projeto sem pisar no trabalho umas das outras

---

# 🌱 Branches: sua linha do tempo paralela

Uma branch é, na prática, uma cópia isolada do código onde você pode trabalhar sem afetar ninguém.

```bash
git checkout -b feature/cadastro-usuario
```

## 🔹 Boas práticas de branch

- **Nomeie com intenção** — `feature/cadastro-usuario`, `fix/bug-login`, `chore/atualiza-dependencias`  
- **Mantenha pequena** — quanto menor o escopo, mais rápido é revisada  
- **Sincronize com frequência** — traga mudanças da `main` cedo para evitar divergências grandes  

👉 Branches pequenas e atualizadas geram menos conflito e revisão mais rápida

---

# 🔀 O que realmente é um Pull Request

Um pull request (PR) não é só "juntar código". É o momento em que seu trabalho:

✅ Fica **visível** para o time  
✅ Passa por **revisão** de outra pessoa  
✅ É validado por **testes automatizados** (CI)  
✅ Gera uma **discussão técnica** documentada  

## 🔹 O que faz um bom PR

- Título e descrição claros sobre **o que** mudou e **por quê**  
- Escopo enxuto — PR gigante é difícil de revisar bem  
- Link para a task/issue relacionada  
- Prints ou exemplos quando envolve comportamento visível  

👉 Um PR bem escrito é um presente para quem vai revisar

---

# 👀 Code Review não é sobre achar erro

Quem revisa um PR não está caçando culpados — está protegendo a qualidade do que vai para produção.

## 🔹 Para quem revisa

- Comente no código, não na pessoa  
- Sugira, não apenas aponte problema  
- Aprove quando estiver bom o suficiente — não precisa ser perfeito  

## 🔹 Para quem recebeu o review

- Não leve pro pessoal  
- Perguntas do revisor também são aprendizado  
- Discorde com argumento, não com defesa automática  

---

# ⚔️ Lidando com conflitos de merge

Conflito acontece quando duas mudanças mexem na mesma linha de código de formas diferentes.

👉 Não é um erro — é só o Git avisando que precisa da sua decisão

```bash
git status          # mostra os arquivos em conflito
# resolva manualmente os trechos marcados com <<<<<<<, =======, >>>>>>>
git add .
git commit
```

👉 Quanto mais tempo uma branch fica sem sincronizar com a `main`, maior tende a ser o conflito

---

# 🏗️ Fluxos de trabalho comuns

Não existe um único "jeito certo". Alguns fluxos dominam o mercado:

### ✅ GitHub Flow
- Branches curtas a partir da `main`  
- PR, review, merge  
- Simples e direto — ótimo para deploy contínuo  

### ✅ Git Flow
- Branches específicas: `develop`, `release`, `hotfix`  
- Mais estruturado — comum em ciclos de release bem definidos  

### ✅ Trunk-Based Development
- Commits pequenos e frequentes direto (ou quase direto) na `main`  
- Exige testes automatizados fortes e feature flags  

👉 O importante não é decorar o nome do fluxo, mas entender **por que** o time escolheu aquele modelo

---

# ⚠️ Erros comuns

- Deixar a branch viva por semanas sem sincronizar  
- Abrir PRs enormes, difíceis de revisar  
- Fazer merge sem revisão  
- Escrever descrições de PR vagas ou vazias  

---

# 📌 Conclusão

- Branch = trabalho isolado e seguro  
- Pull Request = visibilidade, revisão e discussão técnica  
- Code review = qualidade e padrão de time, não caça aos erros  
- Fluxo bem definido = menos conflito, mais previsibilidade  

👉 É assim que times profissionais mantêm código estável, revisado e rastreável

---

# 🔥 Próximo passo

Agora que você sabe colaborar como um time, o próximo nível é:

👉 **Fundamentos do C#: Arquitetura .NET**

Aqui você vai entender o que acontece por trás do seu código: IL, CLR e a estrutura dos Frameworks .NET.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
