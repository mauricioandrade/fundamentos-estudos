# 🧠 LLMs: Large Language Models

> ChatGPT, Claude, Gemini, Llama. Os LLMs são a tecnologia que está mudando como devs trabalham. Entender como funcionam te dá superpoder pra usá-los bem (e não ficar perdido com termos como "tokens", "embeddings", "context window").

---

## 🤔 O que é um LLM?

**Large Language Model** = modelo de linguagem grande. Em essência:

> Uma rede neural massiva (bilhões de parâmetros) treinada pra **prever a próxima palavra** baseada no contexto anterior.

### Exemplo simplificado

Input: `"O céu é"`

LLM calcula probabilidades:
```
"azul" → 70%
"escuro" → 12%
"limpo" → 8%
"infinito" → 5%
"meu" → 0.3%
...
```

Escolhe uma (geralmente a mais provável) e adiciona ao texto. Repete.

> 💡 **Em palavras simples**: LLM é um auto-completar **muito bom**. Tão bom que parece raciocinar — mas internamente é estatística + redes neurais sofisticadas.

---

## 🔤 Tokens: A Unidade Básica

LLMs **não veem palavras**. Veem **tokens** — pedaços de palavras.

### Exemplo

```
Texto: "Sistemas distribuídos são complexos"

Tokens: ["Sistem", "as", " distribu", "ídos", " são", " complex", "os"]
                         (7 tokens)
```

> 💡 **Regra prática**: 1 token ≈ 4 caracteres em inglês. Português costuma usar **mais tokens** que inglês.

### Por que importa?

1. 💰 **Cobrança**: APIs cobram **por token** (input + output)
2. 📏 **Limites**: cada modelo tem **context window** (máximo de tokens)
3. 🐢 **Velocidade**: mais tokens = mais lento

### Ferramentas pra contar tokens

- **OpenAI**: `tiktoken` (Python, JS)
- **Anthropic**: API de count_tokens
- **Online**: [platform.openai.com/tokenizer](https://platform.openai.com/tokenizer)

---

## 📏 Context Window

Quantos tokens o modelo **consegue ler de uma vez** (input + output).

### Tamanhos por modelo (referências de 2024-2026)

| Modelo | Context window |
|---|---|
| GPT-3.5 | 16k |
| GPT-4 (original) | 8k |
| GPT-4 Turbo | 128k |
| Claude 3 / 4 | 200k |
| Gemini 1.5 Pro | 1-2M |

> 💡 **1M tokens ≈ ~750.000 palavras ≈ um livro técnico inteiro**.

### Implicações práticas

- 📚 **Mais context = pode mandar mais informação** (RAG, documentos longos)
- 💰 **Mais context = mais caro** (paga por token)
- 🐢 **Mais context = mais lento** (latência sobe)
- ⚠️ **"Lost in the middle"**: alguns modelos esquecem coisas no meio de contexts gigantes

---

## 🎯 Como LLMs São Treinados (resumido)

### 3 fases típicas

#### 1. Pre-training

Modelo aprende a prever próximo token em **trilhões de tokens da internet**:
- Wikipedia, livros, código, sites, papers acadêmicos
- Milhões de dólares em GPUs
- Resultado: modelo "base" que sabe completar texto

#### 2. Fine-tuning (Supervised)

Treino adicional com **exemplos curados** (perguntas + respostas ideais).

```
Pergunta: "Como ferver água?"
Resposta ideal: "Coloque água numa panela..."
```

Modelo aprende a responder no formato esperado.

#### 3. RLHF (Reinforcement Learning from Human Feedback)

Humanos avaliam respostas; modelo aprende a preferir as melhores. **É o que torna ChatGPT "agradável"** em vez de só completar texto.

---

## 🌡️ Parâmetros de Inferência

Quando você chama uma API de LLM, controla o comportamento via parâmetros:

### Temperature

Controla **criatividade vs determinismo**.

```
temperature: 0     → resposta sempre igual, escolhe sempre o token mais provável
temperature: 0.7   → balanço, algumas variações
temperature: 1.5   → muito criativo, pode falar bobagem
```

> 🤔 **Pra que cenário usar?**
> - **0 (zero)**: dados estruturados, classificação, código
> - **0.7**: respostas conversacionais
> - **1.0+**: criatividade (poesia, brainstorm)

### Top-p (Nucleus Sampling)

Considera só tokens cuja probabilidade cumulativa = p.

```
top_p: 0.9 → considera só os tokens que somam 90% de probabilidade
```

> 💡 Use temperature **OU** top_p, não ambos.

### Max tokens

Limite de tokens de **output**.

```
max_tokens: 500 → resposta nunca passará de 500 tokens
```

⚠️ Se o modelo precisar de mais, vai cortar no meio.

### Stop sequences

Sequências que param a geração.

```
stop: ["\n\n", "User:"]
```

---

## 🎨 Modalidades

### Texto (clássico)

GPT-4, Claude, Gemini, Llama, Mistral.

### Multimodal

Input pode ser texto **+ imagem + áudio + vídeo**.

```python
# Claude com imagem
message = client.messages.create(
    model="claude-opus-4-7",
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {...}},
            {"type": "text", "text": "Descreva esta imagem"}
        ]
    }]
)
```

### Geração de imagem

DALL-E 3, Midjourney, Stable Diffusion, Flux. Não são LLMs — são **diffusion models** (mecânica diferente).

### Áudio

- Whisper (transcrição) — OpenAI
- ElevenLabs (text-to-speech)

---

## 💡 O Que LLMs Fazem Bem (e Mal)

### ✅ Fazem bem

- 📝 Resumir e parafrasear texto
- 💻 Gerar/explicar/refatorar código
- 🌍 Traduzir idiomas
- 🤖 Conversar de forma natural
- 📊 Extrair informação de texto não-estruturado
- 💡 Brainstorming, ideação
- ✏️ Escrita criativa
- 📋 Classificação, análise de sentimento

### ❌ Fazem mal

- 🧮 **Matemática complexa** sem ferramenta externa
- 📅 **Informação atualizada** (knowledge cutoff)
- 🎯 **Fatos específicos** (alucinações)
- 🔢 **Contar coisas** com precisão
- 🤔 **Raciocínio multi-step** complexo (melhorando)
- 📖 **Citação exata** de fontes
- ❓ **Saber que não sabe** (modelos "blefam")

---

## 👻 Hallucinations (Alucinações)

LLMs **inventam** informação com confiança total.

### Exemplo clássico

```
Pergunta: "Qual o paper mais citado de Mauricio da Silva sobre sistemas distribuídos?"

Resposta (alucinação): "O paper 'Distributed Consistency in
Modern Cloud Systems' (2019), publicado no IEEE Transactions on
Cloud Computing, com 847 citações..."
```

→ **Esse paper não existe**. O modelo inventou tudo plausivelmente.

### Por que acontece?

LLMs são treinados pra produzir **texto plausível**, não **texto verdadeiro**. Quando não sabem, **completam mesmo assim**.

### Como mitigar

- 🔍 **RAG** ([próximo tópico](./28-rag.md)): força o modelo a usar documentos reais
- ✅ **Verificação**: pedir fontes específicas verificáveis
- 🎯 **Temperature baixa**: 0 reduz invenções
- 🛡️ **Constrained generation**: forçar formato (JSON schema, regex)
- 🧪 **Avaliação humana**: nunca confie 100%

---

## 🏗️ Modelos Famosos (referência 2024-2026)

### Comerciais (via API)

| Provider | Modelos |
|---|---|
| **OpenAI** | GPT-4, GPT-4 Turbo, GPT-4o, o1 (reasoning) |
| **Anthropic** | Claude Opus, Sonnet, Haiku |
| **Google** | Gemini Pro, Flash, Ultra |
| **Mistral** | Mistral Large, Codestral |
| **Cohere** | Command R, Command R+ |

### Open-source

| Modelo | Provedor | Notas |
|---|---|---|
| **Llama 3.x** | Meta | Estado da arte open |
| **Mistral / Mixtral** | Mistral AI | Excelentes pesos abertos |
| **Phi-3 / Phi-4** | Microsoft | Pequenos e poderosos |
| **Gemma** | Google | Versão "comunidade" |
| **DeepSeek** | DeepSeek AI | Reasoning models |

> 💡 **Open-source não é grátis pra rodar**. Você precisa de GPUs (custosas). Faz sentido pra:
> - Privacidade (dados não saem da empresa)
> - Customização (fine-tuning próprio)
> - Custo em escala muito grande

---

## ⚙️ Pequenos vs Grandes Modelos

Não confunda **mais parâmetros = melhor**. Trade-offs:

### Modelos grandes (Opus, GPT-4)

✅ Melhores em raciocínio complexo
✅ Melhor escrita e nuance
❌ Caros
❌ Lentos

### Modelos pequenos (Haiku, GPT-4o-mini, Phi)

✅ Rápidos
✅ Baratos
✅ Bons pra tarefas específicas
❌ Pioram em raciocínio complexo

> 🎯 **Estratégia comum**: pequeno pra tarefas simples (classificar, formatar), grande pra tarefas complexas (raciocínio, geração criativa).

---

## 🎯 Reasoning Models (modelos de raciocínio)

Geração 2024-2025: modelos que **"pensam" antes de responder** (chain-of-thought visível).

- **OpenAI o1, o3**
- **DeepSeek R1**
- **Claude com extended thinking**

```
Pergunta: "Resolva: se tenho 23 maçãs, dou 7 e ganho 12, ..."

Modelo normal: resposta direta
Reasoning model:
  [pensamento interno]
  Tenho 23. Dou 7 → 16. Ganho 12 → 28.
  Verificando: 23-7+12 = 28 ✓
  [resposta final]
  28 maçãs.
```

✅ Muito melhores em matemática, lógica, código complexo
❌ Mais lentos, mais caros

---

## 🛡️ Safety e Limites

### Knowledge cutoff

Cada modelo "para" de aprender em uma data. Não sabe nada após.

### Refusals

Modelos modernos recusam:
- Instruções pra causar dano
- Conteúdo sexual/violento
- Ajudar em atividades ilegais
- Personificar pessoas reais maliciosamente

### Jailbreaks

Tentativas de **contornar** os limites. Cat-and-mouse contínuo.

### Bias

LLMs refletem **vieses dos dados de treino**:
- Estereótipos de gênero/raça
- Predominância cultural ocidental
- Preferências políticas sutis

---

## ✅ Resumo da página

- LLM = modelo treinado pra **prever próximo token**
- **Tokens** ≠ palavras; ~4 chars em inglês, mais em português
- **Context window** define quanto pode ler/gerar por vez (8k a 2M+)
- **Temperature** controla criatividade (0 = determinístico, alto = criativo)
- LLMs **alucinam**: mitigação principal é RAG + temperature baixa + verificação
- **Modelos grandes** são melhores mas caros e lentos; combine
- **Reasoning models** (o1, R1) são novidade — pensam antes de responder
- Sempre tem **knowledge cutoff** — info recente requer ferramentas externas

---

⬅️ [Anterior: Fundamentos de IA](./26-fundamentos-ia.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: RAG](./28-rag.md)
