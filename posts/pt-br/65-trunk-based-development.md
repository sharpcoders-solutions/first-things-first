# 🧠 Fundamentos do C#: Trunk-Based Development

⏱️ Tempo de leitura: 5 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Git Workflow com branches e Pull Requests  
- Feature Flags para lançar funcionalidades sem novo deploy  

O post 11 te ensinou o fluxo clássico: branch de feature, PR, review, merge. Isso funciona bem, mas em times grandes pode gerar branches vivendo semanas, cheias de conflitos de merge. Trunk-Based Development propõe outro caminho.

👉 **Vamos aprender Trunk-Based Development**

---

# 💡 O problema das branches de vida longa

```
main
  └─ feature/checkout-novo (viva há 3 semanas)
       └─ 40 commits de diferença da main
       └─ conflitos gigantes ao tentar mergear
```

👉 Quanto mais tempo uma branch fica separada da `main`, maior a distância entre elas — e maior o risco de um merge doloroso, cheio de conflitos difíceis de resolver

---

# 🏗️ O que é Trunk-Based Development?

👉 **Trunk-Based Development = todo mundo commita direto (ou quase direto) na branch principal, com frequência de horas, não semanas**

```
main (trunk)
  ├─ commit de Maria (9h da manhã)
  ├─ commit de João (9h15)
  ├─ commit de Valentina (9h40)
  └─ commit de Maria (10h05)
```

👉 Em vez de uma branch de feature viver semanas, os desenvolvedores fazem commits pequenos e frequentes, direto no trunk (ou em branches de vida muito curta, de poucas horas)

---

# 🎯 Como isso funciona sem quebrar a `main`?

## 🔹 Feature Flags escondem trabalho incompleto

```csharp
if (await _featureManager.IsEnabledAsync("NovoCheckout"))
{
    return NovoCheckoutFlow(); // código em progresso, mas já no trunk
}

return CheckoutAtual();
```

👉 Lembra do post 51? Ao invés de uma branch separada esconder trabalho incompleto, a feature flag esconde — o código já está integrado, só desligado

## 🔹 CI/CD roda em cada commit

```yaml
on:
  push:
    branches: [main]
```

👉 Lembra do post sobre GitHub Actions (post 36)? Cada commit no trunk dispara build e testes automaticamente — problemas são detectados em minutos, não semanas depois

## 🔹 Branches de vida curta, quando existem

```bash
git checkout -b fix/bug-critico
# ... corrige, commita, abre PR
# ... merge em poucas horas, não dias
git branch -d fix/bug-critico
```

👉 Quando uma branch é necessária, ela vive **horas**, não semanas — o objetivo é sempre voltar ao trunk o mais rápido possível

---

# ⚖️ GitFlow vs Trunk-Based

## 🔹 GitFlow (post 11)
- Branches de feature, develop, release — mais estrutura  
- Melhor para times que lançam versões espaçadas (bibliotecas, produtos com ciclos de release definidos)  

## 🔹 Trunk-Based
- Integração contínua de verdade — conflitos pequenos e frequentes, não grandes e raros  
- Exige Feature Flags e CI/CD maduros para funcionar bem  
- Comum em times que fazem deploy contínuo, várias vezes ao dia  

---

# ⚠️ Erros comuns

- Adotar Trunk-Based sem CI/CD sólido, quebrando a `main` constantemente para todo mundo  
- Não usar Feature Flags, forçando código incompleto a ficar escondido em branches longas de qualquer forma  
- Fazer commits gigantes "porque agora é tudo no trunk", perdendo a granularidade que o modelo pede  
- Aplicar Trunk-Based em times sem disciplina de testes automatizados — sem rede de segurança, o trunk quebra com frequência  

---

# 📌 Conclusão

- Branches de vida longa acumulam distância e geram merges dolorosos  
- Trunk-Based Development integra código com frequência de horas, não semanas  
- Feature Flags escondem trabalho incompleto sem precisar de branches separadas  
- CI/CD robusto é pré-requisito, não opcional, para esse modelo funcionar  

👉 Com Trunk-Based Development, a integração para de ser um evento estressante no fim do ciclo e vira parte natural do dia a dia

---

# 🔥 Próximo passo

Agora que você conhece estratégias avançadas de branching, o próximo nível é:

👉 **Fundamentos do C#: Unsafe Code e Ponteiros**

Aqui você vai aprender a sair da segurança gerenciada do C# quando performance extrema exige controle direto de memória.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
