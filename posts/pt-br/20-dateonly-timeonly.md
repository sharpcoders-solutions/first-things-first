# 🧠 Fundamentos do C#: DateOnly e TimeOnly

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- LINQ para consultar e transformar coleções de forma declarativa  
- `DateTime`, usado desde os primeiros posts sempre que uma entidade precisa registrar quando algo aconteceu  

Você provavelmente já modelou uma "data de nascimento" ou um "horário de abertura da loja" usando `DateTime`. Mas `DateTime` sempre carrega data **e** hora juntos — mesmo quando seu domínio só precisa de uma das duas partes.

👉 **Vamos conhecer `DateOnly` e `TimeOnly`**

---

# 💡 O problema histórico do `DateTime`

```csharp
public class Pessoa
{
    public DateTime DataNascimento { get; set; } // 1990-05-15 00:00:00 — a hora não significa nada aqui
}

public class Loja
{
    public DateTime HorarioAbertura { get; set; } // 0001-01-01 09:00:00 — a data não significa nada aqui
}
```

👉 Modelar uma data de nascimento com `DateTime` sempre deixa um componente de hora (`00:00:00`) sem significado real. Modelar um horário de funcionamento com `DateTime` sempre exige uma data "fantasma" (geralmente `0001-01-01`) que ninguém usa. Isso já causou bugs reais: comparações que falham por causa da parte da hora, fusos horários aplicados onde não fazem sentido

---

# 📅 `DateOnly`: só a data

```csharp
public class Pessoa
{
    public DateOnly DataNascimento { get; set; }
}

var nascimento = new DateOnly(1990, 5, 15);
var hoje = DateOnly.FromDateTime(DateTime.Now);

int idade = hoje.Year - nascimento.Year;
if (nascimento > hoje.AddYears(-idade)) idade--;

Console.WriteLine(nascimento.ToString("dd/MM/yyyy")); // 15/05/1990
```

👉 **`DateOnly` = um `struct` que representa exclusivamente ano, mês e dia — sem hora, sem fuso horário, sem ambiguidade**

Comparações entre duas datas de nascimento nunca mais falham por causa de segundos ou milissegundos diferentes — o tipo simplesmente não tem espaço para carregar essa informação

---

# ⏰ `TimeOnly`: só o horário

```csharp
public class Loja
{
    public TimeOnly HorarioAbertura { get; set; }
    public TimeOnly HorarioFechamento { get; set; }
}

var abertura = new TimeOnly(9, 0);
var fechamento = new TimeOnly(18, 30);

var agora = TimeOnly.FromDateTime(DateTime.Now);
bool estaAberta = agora >= abertura && agora <= fechamento;

var duracao = fechamento - abertura; // TimeSpan de 9h30
```

👉 **`TimeOnly` = um `struct` que representa exclusivamente hora, minuto, segundo — sem data associada**

Perfeito para horários recorrentes (funcionamento da loja, horário de um lembrete diário) que se repetem todo dia, sem depender de qual dia especificamente

---

# 🔄 Convertendo entre `DateTime`, `DateOnly` e `TimeOnly`

```csharp
var agora = DateTime.Now;

DateOnly soData = DateOnly.FromDateTime(agora);
TimeOnly soHora = TimeOnly.FromDateTime(agora);

// Recombinando os dois de volta em um DateTime
DateTime recombinado = soData.ToDateTime(soHora);
```

👉 A conversão nos dois sentidos é direta — você pode continuar usando `DateTime` onde ele faz sentido (registros de auditoria, timestamps completos, do post sobre logging estruturado) e usar `DateOnly`/`TimeOnly` onde a semântica do domínio pede exatamente uma das partes

---

# 🗄️ Mapeamento no Entity Framework Core

```csharp
public class Evento
{
    public int Id { get; set; }
    public DateOnly Data { get; set; }
    public TimeOnly HorarioInicio { get; set; }
}
```

👉 Lembra do EF Core (post 35)? Desde o EF Core 6, `DateOnly` e `TimeOnly` são mapeados nativamente para os tipos `date` e `time` do SQL Server — sem precisar de conversores customizados, e sem o desperdício de armazenar uma parte de data ou hora que nunca será usada

---

# ⚠️ Erros comuns

- Continuar usando `DateTime` com a parte de hora sempre zerada "por costume", quando `DateOnly` comunica a intenção muito mais claramente  
- Misturar comparações entre `DateTime` e `DateOnly`/`TimeOnly` sem converter explicitamente, gerando erros de compilação evitáveis  
- Ignorar `DateOnly`/`TimeOnly` em projetos legados só porque "sempre foi `DateTime`", perdendo a chance de eliminar uma classe inteira de bugs de fuso horário e ambiguidade  
- Usar `TimeOnly` para armazenar duração — `TimeOnly` representa um ponto no tempo dentro de um dia, não um intervalo; para duração, `TimeSpan` continua sendo o tipo certo  

---

# 📌 Conclusão

- `DateOnly` representa exclusivamente ano, mês e dia, sem componente de hora  
- `TimeOnly` representa exclusivamente hora, minuto e segundo, sem data associada  
- Ambos eliminam bugs de fuso horário e comparação causados pela parte não usada de um `DateTime`  
- O EF Core mapeia os dois nativamente para os tipos `date`/`time` do banco, desde a versão 6  

👉 Com tipos de data e hora mais precisos, o próximo passo é olhar para outra categoria de tipos numéricos que também merece mais cuidado do que costuma receber: os tipos de ponto flutuante e decimal

---

# 🔥 Próximo passo

Agora que você modela datas e horários com precisão, o próximo nível é:

👉 **Fundamentos do C#: Tipos Numéricos e Precisão (float, double e decimal)**

Aqui você vai entender por que `0.1 + 0.2` não dá exatamente `0.3` em `double`, e quando cada tipo numérico realmente deve ser usado.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
