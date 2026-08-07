# 🧠 Fundamentos do C#: Background Jobs com Hangfire

⏱️ Tempo de leitura: 6 minutos  
✍️ Autor: Vitor Santos  + 🤖 Copilot

## 🚀 Introdução

Até aqui, você aprendeu:

- Feature flags para controlar comportamento sem novo deploy  
- Mensageria para desacoplar sistemas no tempo  

Nem toda tarefa deveria acontecer durante uma requisição HTTP. Gerar um relatório pesado, enviar um e-mail em massa, limpar dados antigos — nada disso deveria fazer o usuário esperar.

👉 **Vamos aprender a rodar tarefas em segundo plano com Hangfire**

---

# 💡 O problema de colocar tudo na requisição

```csharp
[HttpPost]
public async Task<IActionResult> GerarRelatorioMensal()
{
    var relatorio = await ProcessarDadosDoMesInteiro(); // pode levar minutos
    return Ok(relatorio);
}
```

👉 Uma requisição HTTP que demora minutos derruba a experiência do usuário e arrisca timeout. Algumas tarefas precisam existir **fora** do ciclo de vida de uma requisição

---

# 🏗️ Configurando o Hangfire

```bash
dotnet add package Hangfire.AspNetCore
dotnet add package Hangfire.SqlServer
```

```csharp
// Program.cs
builder.Services.AddHangfire(config => config
    .UseSqlServerStorage(builder.Configuration.GetConnectionString("Padrao")));

builder.Services.AddHangfireServer();

// ...

app.UseHangfireDashboard("/hangfire");
```

👉 O Hangfire persiste os jobs no banco de dados — se a aplicação reiniciar (lembra do post sobre Docker?), os jobs pendentes **não se perdem**, diferente de uma fila em memória

---

# 🔥 Fire-and-forget: rodar uma vez, imediatamente

```csharp
[HttpPost]
public IActionResult CriarPedido(CriarPedidoRequest request)
{
    var pedido = _servico.Criar(request);

    BackgroundJob.Enqueue(() => _servicoEmail.EnviarConfirmacao(pedido.Id));

    return Ok(pedido); // responde na hora, o e-mail roda depois
}
```

👉 O mesmo espírito do post sobre mensageria: a requisição responde imediatamente, e a tarefa secundária roda de forma independente, em outro processo/thread

---

# ⏰ Jobs recorrentes

```csharp
RecurringJob.AddOrUpdate(
    "limpar-carrinhos-abandonados",
    () => _servicoCarrinho.LimparAbandonados(),
    Cron.Daily);
```

👉 Substitui a necessidade de tarefas agendadas no sistema operacional (cron jobs, Task Scheduler) — a agenda vive dentro do próprio código C#, versionada junto com o resto da aplicação

---

# ⏳ Jobs atrasados e encadeados

```csharp
// Roda daqui a 24 horas
BackgroundJob.Schedule(() => _servicoEmail.EnviarLembrete(pedidoId), TimeSpan.FromHours(24));

// Só roda depois que o job anterior terminar com sucesso
var jobId = BackgroundJob.Enqueue(() => ProcessarPagamento(pedidoId));
BackgroundJob.ContinueJobWith(jobId, () => EnviarNotaFiscal(pedidoId));
```

👉 `ContinueJobWith` cria uma cadeia de dependência entre jobs — útil quando uma etapa só faz sentido depois que a anterior terminou com sucesso

---

# 🔁 Retry automático

```csharp
[AutomaticRetry(Attempts = 3)]
public void ProcessarPagamento(int pedidoId)
{
    // se lançar exceção, o Hangfire tenta de novo automaticamente
}
```

👉 Lembra do post sobre Polly? Aqui o retry acontece no nível do job, não da chamada HTTP — o Hangfire tenta novamente com backoff automático, e depois de esgotar as tentativas, marca o job como falho no dashboard

---

# 📊 O dashboard: visibilidade sem código extra

Acessando `/hangfire`, você vê, sem escrever nenhum código adicional:

- Jobs em execução, agendados e concluídos  
- Jobs que falharam, com stack trace completo  
- Histórico de execuções recorrentes  

👉 Combinado com o logging estruturado (Serilog) que você já configurou, isso dá visibilidade completa sobre o que está acontecendo fora do fluxo síncrono da aplicação

---

# ⚠️ Erros comuns

- Colocar tarefas críticas e urgentes (como validar pagamento em tempo real) em background, quando o usuário precisa da resposta imediata  
- Não configurar `AutomaticRetry`, deixando falhas transitórias derrubarem o job sem segunda chance  
- Esquecer de proteger `/hangfire` com autenticação, expondo o dashboard publicamente  
- Rodar jobs longos sem checar `CancellationToken`, dificultando um desligamento gracioso da aplicação  

---

# 📌 Conclusão

- Background jobs tiram tarefas demoradas do ciclo de vida da requisição HTTP  
- Hangfire persiste jobs no banco, sobrevivendo a reinícios da aplicação  
- Jobs recorrentes substituem agendadores externos, com a agenda vivendo no próprio código  
- Retry automático e encadeamento de jobs cobrem a maioria dos cenários reais de processamento assíncrono  

👉 Com Hangfire, sua aplicação separa claramente o que precisa de resposta imediata do que pode (e deve) acontecer em segundo plano

---

# 🔥 Próximo passo

Agora que você sabe processar tarefas em segundo plano, o próximo nível é:

👉 **Fundamentos do C#: Rate Limiting em ASP.NET Core**

Aqui você vai aprender a proteger sua API contra uso excessivo, seja por engano ou por má intenção.

## ✍️ Nota dos autores

Este artigo foi produzido em colaboração entre **Vitor Santos** e um assistente de IA (Copilot), combinando experiência prática com apoio tecnológico para criar conteúdo técnico de qualidade.
