# 🧰 Soft Skills pra Devs

> Técnico forte é só metade do trabalho. Quem fica preso em sênior por anos geralmente trava em **comunicação, code review, mentoria, gestão de tempo**. Soft skills não são "fofas" — são o que diferencia carreiras estagnadas das que crescem.

---

## 🎯 Por Que Soft Skills Importam Mais Que Você Pensa

Pesquisas (Google Project Aristotle, Stack Overflow, etc) mostram consistentemente: o que torna times altamente produtivos **não é** capacidade técnica individual. É:

1. 🤝 **Segurança psicológica** (poder errar e perguntar)
2. 📞 **Comunicação clara**
3. 🎯 **Confiabilidade** (entregar o que prometeu)
4. 🧭 **Clareza de papéis**
5. 💡 **Senso de propósito**

> 💡 Você pode ser o melhor codador do mundo. Se ninguém quer trabalhar com você, sua carreira trava.

---

## 💬 Comunicação Escrita

A maior parte da comunicação dev é **escrita** (Slack, email, PR, docs). Escrever bem é vantagem competitiva enorme.

### Princípios

#### 1. Comece pelo bottom line

```
❌ "Olá! Espero que esteja bem. Estive analisando o módulo X
    nos últimos dias e percebi que tem alguns pontos que
    poderíamos discutir... [400 palavras de contexto]
    ...então acho que precisamos refatorar."

✅ "Precisamos refatorar o módulo X. Razões:
    1. Performance ruim em Y
    2. Bug recorrente em Z
    3. Difícil de testar
    Detalhes: [link/thread]"
```

Em 2026, ninguém lê parágrafos infinitos. Comece pelo conclusão.

#### 2. Estruture com bullets quando faz sentido

Listas escaneam melhor que prosa:

```
Para a reunião amanhã, decidir:
- API REST ou GraphQL pra integração
- Time responsável pelo deploy
- Prazo final do MVP
```

#### 3. Diga o que precisa do leitor

```
❌ "PR pronto pra review."

✅ "PR pronto pra review. Foco especial na parte de cache
    (linha 47-89) — não tenho certeza da estratégia de TTL.
    Pode revisar até quinta?"
```

#### 4. Pergunte específico

```
❌ "Tô com dúvida no Spring."

✅ "Estou tentando configurar @Async no Spring Boot mas
    o método continua síncrono. Habilitei @EnableAsync na
    config principal. Versão Spring Boot 3.2. O que pode estar
    faltando? [link pro código]"
```

> 🤔 **A regra do XY problem**: você acha que tem problema X. Pergunta sobre Y porque acha que Y resolve X. Mas Y nem é a solução. Sempre **explique o problema original**, não só sua tentativa de solução.

---

## 🤝 Code Review

Code review é onde a qualidade do time é construída — ou destruída.

### Como pedir review

#### Boas práticas

- 📝 **Descrição clara**: o quê e por quê
- 🎯 **Tamanho pequeno**: <400 linhas idealmente
- ✅ **Self-review primeiro**: você é o primeiro a revisar
- 🧪 **Testes**: incluir
- 📷 **Screenshots/vídeos**: pra mudanças visuais
- 🏷️ **Labels**: tipo, prioridade

#### Template de PR

```markdown
## O que faz
[descrição em 2-3 linhas]

## Por que
[motivação, ticket, contexto]

## Como testar
1. [passo 1]
2. [passo 2]

## Pontos de atenção
- [coisa que você gostaria de feedback]

## Screenshots
[se aplicável]
```

### Como fazer review (sendo bom revisor)

#### Comente no código, não na pessoa

```
❌ "Você sempre escreve código difícil de ler"
✅ "Essa função tá longa. Que tal extrair a parte X em uma
    função separada? Fica mais fácil de testar."
```

#### Use linguagem de sugestão

```
❌ "Errado. Refaz."
✅ "Acho que aqui ia funcionar melhor com [abordagem Y]
    porque [razão]. O que acha?"
```

#### Diferencie bloqueante de sugestão

```
🔴 [BLOQUEANTE] Esse SQL é vulnerável a injection
🟡 [SUGESTÃO] Esse nome poderia ser mais claro
💚 [OPCIONAL] Curiosidade: por que essa abordagem?
```

#### Aprove o que está bom

```
✅ "Cool, não sabia que dava pra fazer assim com streams,
    aprendi algo!"
```

> 💡 Code review **não é** lugar pra mostrar quem é mais inteligente. É pra **ensinar e aprender mutuamente**.

### Como **receber** review

#### Não leve pro pessoal

Comentário é sobre o código, não sobre você. Mesmo que pareça brutal, foca no aprendizado.

#### Pergunte se não entendeu

```
"Pode explicar por que essa abordagem é melhor? Quero
entender pra aprender."
```

#### Discorde com argumentos

Se acha que o reviewer está errado:

```
"Considerei essa abordagem mas escolhi [outra] porque [razão].
Te convenço ou prefere mesmo assim?"
```

---

## 👥 Reuniões

Reuniões mal feitas matam produtividade. Aqui regras pra você sobreviver.

### Antes de aceitar uma reunião

- 📋 **Tem agenda?** Sem agenda, recuse
- 🎯 **Tem objetivo claro?** "Brainstorm" sem foco vira chat
- 👥 **Você precisa estar?** Talvez só ler ata depois
- ⏰ **Tem horário fixo?** Reuniões abertas viram eternas

### Durante a reunião

- 📵 **Foco** — laptop fechado se não for tomar nota
- 🗣️ **Fale conciso** — vai pro ponto
- 📝 **Tome notas pessoais** mesmo se tiver ata
- ❓ **Pergunte se não entendeu** — outras pessoas têm a mesma dúvida

### Depois

- ✅ **Action items claros** — quem faz o quê até quando
- 📤 **Compartilhe ata** com decisões

### Async-first é melhor

Em time distribuído, **async** (Slack, Notion, docs) é geralmente melhor que reunião síncrona:

- ⏱️ Cada um responde no seu tempo
- 📜 Fica registrado pra futuro
- 🌍 Funciona com fusos diferentes
- 🧠 Pessoas pensam melhor com tempo

> 💡 **"Could this have been an email?"** Antes de marcar reunião, pergunte isso.

---

## 🎓 Mentoria e Pair Programming

### Quando você é mentor

- 🤔 **Pergunte mais que responda** — leve a pessoa a pensar
- 🐌 **Dê tempo de errar** — é parte do aprendizado
- 🎯 **Foque no problema atual** — não despeje 10 anos de conhecimento
- 🌱 **Adapte ao nível** — júnior precisa de coisas diferentes que sênior

#### Anti-pattern: "Deixa que eu faço"

```
Júnior: "Tô tentando fazer X..."
Sênior pega o teclado e faz em 5 minutos.
```

Você "ajudou" mas o júnior aprendeu zero. **Resista**.

### Quando você é mentorado

- 🙋 **Pergunte cedo** — não trave por 4h num bug
- 📝 **Mas tente um pouco antes** — mostre o que você fez
- 🎯 **Pergunte específico** — vimos antes
- 📚 **Anote o aprendizado** — pra não perguntar de novo

### Pair Programming

Dois devs, um teclado. Pode ser:

- **Driver/Navigator**: um teclado, um pensa adiante
- **Ping-pong**: revezam (TDD: um escreve teste, outro implementa)

✅ Aprendizado mútuo
✅ Bug encontrado antes
✅ Conhecimento distribuído
❌ Cansa muito (não faça 8h)
❌ Não substitui code review

---

## ⏰ Gestão de Tempo

### Pomodoro

25 min focado + 5 min pausa. A cada 4 ciclos, pausa de 15-30 min.

✅ Combate procrastinação
✅ Cria ritmo sustentável
❌ Nem todo trabalho cabe em 25 min

### Time blocking

Bloqueie horários no calendário pra **deep work**:

```
9h-11h: Coding (sem reuniões, Slack mute)
11h-12h: Code reviews
12h-13h: Almoço
13h-14h: Reuniões
14h-17h: Coding
```

### Deep work vs shallow work

- 🧠 **Deep work**: tarefas que exigem foco intenso (codar feature complexa, debugging difícil)
- 🌊 **Shallow work**: respondas Slack, revisar PR pequeno, atualizar tickets

Faça deep work nas suas **horas de pico** (pra maioria, manhã).

### O custo da troca de contexto

Cada vez que você é interrompido (Slack, reunião, "tem 1 minuto?"), perde **~15 minutos** pra voltar ao foco.

Estratégias:
- 🔕 Silenciar notificações
- 🚪 Hora "escondido" diariamente (modo Não Perturbe no Slack)
- 📅 Bloqueios de calendário

---

## 🎯 Carreira

### Auto-marketing (sem ser babaca)

"Trabalhar duro" não é o suficiente. As pessoas precisam **saber** que você trabalhou.

- 📊 **Documente entregas**: o que você fez, qual o impacto
- 📢 **Compartilhe aprendizados**: posts, palestras, threads
- 🤝 **Construa relações**: networking não é só LinkedIn
- 🎯 **Tenha clareza sobre o que quer**: cargo? domínio? remoto?

### Princípios pra crescer

1. **Resolva problemas que ninguém quer resolver** (legacy, infra ruim)
2. **Compartilhe conhecimento** — escreva, ensine
3. **Pegue projetos visíveis** — não só backstage
4. **Aprenda continuamente** — leia código de open source
5. **Trabalhe em times bons** — você vira a média

### Síndrome do impostor

Quase todo dev sente — incluindo seniores. Algumas verdades:

- 🤯 **Você sabe muito mais que você acha**
- 🔍 **Os "expertos" também googlam o tempo todo**
- 📚 **Mais aprende, mais percebe que não sabe** — Dunning-Kruger funciona ao contrário
- 🏃 **Aja** — confiança vem com a ação, não antes

### Negociação de salário

- 📊 **Pesquise mercado** (Glassdoor, Levels.fyi, comunidades)
- 💼 **Tenha alternativas** (outra oferta = leverage)
- 🎯 **Faça um número primeiro** se for confortável
- ⏸️ **Pause antes de aceitar** — "Vou pensar"
- 📜 **Negocie pacote total** — não só salário (PLR, equity, benefícios)

---

## 🤐 Receber e Dar Feedback

### Recebendo feedback negativo

#### 1. Respira fundo

A reação inicial é defensiva. **Respire**.

#### 2. Pergunte exemplos específicos

```
"Pode me dar um exemplo concreto de quando isso aconteceu?"
```

#### 3. Não tente justificar imediatamente

Escute primeiro. Pergunte depois.

#### 4. Agradeça

Mesmo se discordar. Feedback honesto é raro e valioso.

#### 5. Pense depois

Não responda de cabeça quente. Volte 1-2 dias depois com plano.

### Dando feedback negativo

#### Em particular, não em público

Nunca corrija em reunião, no Slack do time. **DM ou conversa 1:1**.

#### Modelo SBI: Situation, Behavior, Impact

```
Situação: "Na reunião de ontem do design..."
Comportamento: "Você interrompeu 3 vezes a Maria..."
Impacto: "...e ela parou de contribuir. Isso prejudica
o time porque perdemos perspectivas dela."
```

#### Foque no comportamento, não na personalidade

```
❌ "Você é arrogante"
✅ "Quando você interrompe, parece arrogante (mesmo que
    não seja a intenção)"
```

#### Termine com pedido específico

```
"Pode prestar atenção em deixar as pessoas terminarem?
Talvez anotar suas ideias pra falar depois ajude."
```

---

## 🧠 Saúde Mental

Devs têm taxas altas de burnout, ansiedade e depressão. Reconhecer isso é o primeiro passo.

### Sinais de burnout

- 🥱 Cansaço crônico mesmo dormindo bem
- 😡 Irritação fácil com colegas/código
- 🚫 Cinismo sobre o trabalho
- 📉 Produtividade caindo, mesmo trabalhando mais
- 😶 Apatia geral

### Prevenção

- ⏰ **Limite horas extras** (sustentável só raramente)
- 💪 **Exercício** (não negociável pra saúde mental)
- 😴 **Sono** (7-8h, não negocie)
- 🌳 **Hobbies fora de tela** (se desconecte)
- 👥 **Vida social** (devs tendem a isolar)
- 🩺 **Terapia** se possível (não estigmatize)

### Disconnect

- 📵 **Apague Slack do celular** ou silencie fora do horário
- 📅 **Férias de verdade** (sem laptop)
- ⛔ **Diga não** a horas extras crônicas

> 💡 **Dev burnout é caro pra empresa também.** Bons gestores entendem isso. Se o seu não entende, é red flag.

---

## ✅ Resumo da página

- **Soft skills** definem carreiras tanto quanto técnico
- **Escreva** começando pelo bottom line, com estrutura, perguntas específicas
- **Code review**: comente código não pessoa, diferencie bloqueante de sugestão
- **Reuniões**: só com agenda; "could be an email?"
- **Mentoria**: pergunte mais que responde; resista pegar o teclado
- **Time blocking** + **deep work** > multitasking
- **Auto-marketing** sem ser babaca: documente entregas, compartilhe
- **Síndrome do impostor**: você sabe mais do que acha
- **Feedback negativo**: SBI, em particular, focado em comportamento
- **Saúde mental** não é luxo — burnout afunda carreiras

---

⬅️ [Anterior: LGPD/GDPR](./37-lgpd-gdpr.md) | 🔙 [Voltar ao índice](./README.md)
