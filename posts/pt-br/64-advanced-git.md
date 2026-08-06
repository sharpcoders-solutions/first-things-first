# 🧠 Fundamentos do C#: Git Avançado

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Git & GitHub: os comandos essenciais do dia a dia  
- Git Workflow: branches, pull requests e o fluxo de um time  

Você já sabe `commit`, `push`, `pull` e `merge` (posts 10 e 11). Agora é hora de dominar os comandos que separam quem "usa" Git de quem realmente entende Git.

👉 **Vamos aprender Git Avançado**

---

# 💡 Rebase interativo: reescrevendo o histórico local

```bash
git rebase -i HEAD~4
```

```
pick a1b2c3 adiciona validação de e-mail
pick d4e5f6 corrige typo
pick g7h8i9 adiciona teste
pick j0k1l2 corrige typo de novo

# troque "pick" por "squash" para juntar commits
```

👉 Antes de abrir um Pull Request, `rebase -i` deixa você limpar commits de "corrige typo" e "wip", transformando um histórico bagunçado em uma sequência lógica de mudanças — muito mais fácil de revisar

---

# 🍒 Cherry-pick: trazendo um commit específico

```bash
git cherry-pick a1b2c3d
```

👉 Você corrigiu um bug crítico na branch `main`, mas precisa daquele mesmo fix também na branch `release/2.0`, sem trazer todo o resto que já foi mergeado na `main`. `cherry-pick` aplica só aquele commit específico

---

# 🔍 Bisect: caçando o commit que quebrou tudo

```bash
git bisect start
git bisect bad          # o commit atual está com bug
git bisect good v1.5.0  # essa versão antiga funcionava
```

```
Bisecting: 15 revisions left to test
[commit-hash] checkout dessa revisão para você testar
```

```bash
git bisect good  # ou "git bisect bad", dependendo do teste
# repete até o Git isolar o commit exato
```

👉 Ao invés de revisar manualmente 30 commits procurando qual introduziu um bug, `bisect` faz uma busca binária automática — encontra o commit exato em `log2(n)` passos

---

# 🕰️ Reflog: recuperando o "irrecuperável"

```bash
git reflog
```

```
a1b2c3d HEAD@{0}: reset: moving to HEAD~3
d4e5f6g HEAD@{1}: commit: adiciona feature X
g7h8i9j HEAD@{2}: commit: corrige bug Y
```

```bash
git reset --hard d4e5f6g  # volta para o estado antes do reset
```

👉 Fez um `git reset --hard` e "perdeu" commits? O `reflog` guarda um histórico de tudo que o `HEAD` já apontou, mesmo commits que parecem ter sumido — na prática, é muito difícil perder trabalho de verdade no Git

---

# 🌿 Stash: guardando trabalho em progresso temporariamente

```bash
git stash push -m "WIP: refatorando o serviço de pagamento"
git checkout main
git pull
git checkout minha-branch
git stash pop
```

👉 Precisa trocar de branch urgentemente com mudanças não commitadas? `stash` guarda o estado do working directory temporariamente, sem precisar de um commit "sujo" só para não perder o trabalho

---

# ⚠️ Erros comuns

- Fazer `rebase` em uma branch compartilhada já publicada, reescrevendo histórico que outras pessoas já baixaram — use `rebase` só localmente, antes de compartilhar  
- Usar `git push --force` sem `--force-with-lease`, sobrescrevendo trabalho de outra pessoa sem perceber  
- Esquecer que `cherry-pick` cria um novo commit com hash diferente, podendo gerar conflitos futuros se a branch original for mergeada depois  
- Não usar `bisect` e perder horas testando commits manualmente um por um  

---

# 📌 Conclusão

- Rebase interativo limpa o histórico local antes de compartilhar  
- Cherry-pick traz um commit específico sem trazer toda a branch  
- Bisect encontra o commit problemático com busca binária automática  
- Reflog é a rede de segurança contra "perder" commits  

👉 Com Git avançado, você para de ter medo de mexer no histórico e passa a usá-lo como uma ferramenta de investigação e organização

---

# 🔥 Próximo passo

Agora que você domina Git em profundidade, o próximo nível é:

👉 **Fundamentos do C#: Trunk-Based Development**

Aqui você vai aprender uma estratégia de branching alternativa ao GitFlow, usada por times de alta performance.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
