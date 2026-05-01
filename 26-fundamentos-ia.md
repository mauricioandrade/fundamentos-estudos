# 🤖 Fundamentos de IA/ML para Devs

> Você não precisa virar cientista de dados, mas precisa entender o que IA é, o que **não é**, e como integrar com aplicações reais. Este tópico cobre o mínimo conceitual pra você não ficar perdido nas reuniões.

---

## 🎯 Desmistificando: O que é IA?

### IA não é mágica

Não é consciência, não "pensa", não tem "intenção". É **matemática estatística rodando em GPUs**.

> 💡 **Em palavras simples**: IA é um programa que aprendeu padrões a partir de dados, e usa esses padrões pra fazer previsões em dados novos. Tipo um aluno que decora muitas provas resolvidas e tenta resolver provas novas.

### IA, ML, Deep Learning, LLMs — diferenças

```
┌─────────────────────────────────────────┐
│ IA (Inteligência Artificial)            │
│ ┌─────────────────────────────────────┐ │
│ │ ML (Machine Learning)               │ │
│ │ ┌─────────────────────────────────┐ │ │
│ │ │ Deep Learning (redes neurais)   │ │ │
│ │ │ ┌─────────────────────────────┐ │ │ │
│ │ │ │ LLMs (GPT, Claude, etc)     │ │ │ │
│ │ │ └─────────────────────────────┘ │ │ │
│ │ └─────────────────────────────────┘ │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

| Termo | Significa |
|---|---|
| **IA** | Qualquer técnica que faz máquina parecer inteligente (incluindo regras hardcoded) |
| **ML** | Subset da IA: máquina aprende **a partir de dados**, sem programação explícita |
| **Deep Learning** | Subset do ML: usa **redes neurais profundas** (muitas camadas) |
| **LLMs** | Modelos de linguagem grandes (Deep Learning especializado em texto) |

---

## 📊 Tipos de Machine Learning

### 1️⃣ Supervised Learning (Supervisionado)

Aprende com **exemplos rotulados** (input + output esperado).

```
Dados de treino:
Input: foto de gato → Output: "gato"
Input: foto de cachorro → Output: "cachorro"
Input: foto de pássaro → Output: "pássaro"
... (milhares de exemplos)

Depois: foto nova → modelo prevê classe
```

**Casos de uso:**
- Classificação (spam vs não-spam, diagnóstico médico)
- Regressão (prever preço de imóvel, demanda)

### 2️⃣ Unsupervised Learning (Não-supervisionado)

Aprende **padrões sem rótulos**.

**Casos de uso:**
- Clustering (segmentação de clientes)
- Redução de dimensionalidade
- Detecção de anomalias

### 3️⃣ Reinforcement Learning (Por Reforço)

Aprende por **tentativa e erro**, recebendo recompensas.

**Casos de uso:**
- Jogos (AlphaGo)
- Robótica
- Sistemas de recomendação

### 4️⃣ Self-Supervised (Auto-supervisionado)

Cria seus próprios rótulos a partir dos dados. **É como GPT/Claude foram treinados.**

```
Texto: "O gato sentou no _____"
Modelo aprende a prever a palavra escondida.
```

---

## 🧠 Como Modelos Aprendem (sem cálculo, prometo)

### Conceito básico

1. Modelo começa **aleatório** (chuta tudo errado)
2. Vê um exemplo de treino
3. Compara previsão vs resposta correta
4. Ajusta seus parâmetros pra **errar menos**
5. Repete bilhões de vezes

### Termos importantes

| Termo | Significa |
|---|---|
| **Training (treino)** | Processo de aprendizado |
| **Inference (inferência)** | Usar o modelo treinado pra fazer previsões |
| **Model weights** | Os números que o modelo aprendeu |
| **Loss function** | Mede o quão errado o modelo está |
| **Gradient descent** | Método matemático de ajustar pesos |
| **Epoch** | Uma passada completa pelos dados de treino |
| **Batch** | Subconjunto de exemplos processados juntos |

> 💡 **Inferência** é o que você faz quando chama uma API de IA. **Treino** é o que custou milhões pra OpenAI/Anthropic fazerem.

---

## 🎲 Conceitos Estatísticos Essenciais

### Overfitting

Modelo "decorou" os dados de treino e não generaliza pra dados novos.

```
Treino: 99% acerto
Teste (dados novos): 50% acerto

→ Overfitting clássico
```

**Como evitar**: mais dados, regularização, validação cruzada, dropout.

### Underfitting

Modelo é simples demais pra capturar padrões.

```
Treino: 60% acerto
Teste: 60% acerto
→ Modelo subaproveitou
```

### Bias e Variance

Trade-off clássico:
- **Bias alto** = underfitting (suposições simples demais)
- **Variance alta** = overfitting (sensível demais a ruído nos dados)

### Train/Validation/Test Split

```
Dados (100%)
├── Train (60-70%): pra modelo aprender
├── Validation (15-20%): pra ajustar hiperparâmetros
└── Test (15-20%): pra avaliar performance final
```

> ⚠️ **Nunca** treine no conjunto de teste. É como decorar a prova.

### Métricas

Não basta "acertar X%". Depende do problema:

| Métrica | Quando usar |
|---|---|
| **Accuracy** | Classes balanceadas |
| **Precision** | Quando falso positivo é caro (spam, fraude) |
| **Recall** | Quando falso negativo é caro (câncer, ataques) |
| **F1 Score** | Equilíbrio entre precision e recall |
| **AUC-ROC** | Comparar classificadores |
| **MSE / RMSE** | Regressão (prever números) |

---

## 🧩 Redes Neurais (resumido)

### A unidade básica: Neurônio

Inspirado no cérebro, mas **muito mais simples**:

```
inputs → multiplica por pesos → soma → função de ativação → output
   x₁ ─→ w₁ ─┐
   x₂ ─→ w₂ ─┼─→ Σ → ativação → y
   x₃ ─→ w₃ ─┘
```

### Rede neural

Múltiplos neurônios em **camadas**:

```
Input Layer ─→ Hidden Layer 1 ─→ Hidden Layer 2 ─→ Output Layer
   (dados)        (aprende)        (aprende)         (predição)
```

### Deep Learning

Quando tem **muitas camadas escondidas** (deep = profundo).

### Tipos comuns

| Tipo | Pra quê |
|---|---|
| **CNN** (Convolutional NN) | Imagens, visão |
| **RNN/LSTM** | Sequências, texto antigo |
| **Transformer** | Texto moderno (GPT, Claude, BERT) |
| **GAN** | Geração (imagens, dados sintéticos) |
| **Diffusion** | Geração de imagem (Stable Diffusion, DALL-E) |

---

## 🛠️ Ferramentas e Frameworks

### Para usar IA pronta

- **OpenAI API** (GPT, DALL-E, Whisper)
- **Anthropic API** (Claude)
- **Google Gemini API**
- **Hugging Face** (milhares de modelos open-source)
- **Replicate** (vários modelos via API)

### Para treinar (se quiser ir fundo)

| Framework | Linguagem | Uso |
|---|---|---|
| **PyTorch** | Python | Pesquisa, indústria moderna |
| **TensorFlow** | Python | Indústria (mais "enterprise") |
| **JAX** | Python | Performance extrema (Google) |
| **scikit-learn** | Python | ML clássico (não Deep Learning) |

> 💡 **Pra dev backend**: você vai usar APIs. Treinar do zero é trabalho de cientista de dados.

---

## 💼 Onde IA Aparece em Apps Reais

Não é só "chatbot". IA está em:

| Aplicação | Tipo de IA |
|---|---|
| Detecção de fraude | Classificação supervisionada |
| Recomendações (Netflix, Spotify) | Filtering, redes neurais |
| Busca semântica | Embeddings + similarity |
| Tradução automática | LLMs / Transformer |
| Reconhecimento de imagem | CNN |
| Reconhecimento de voz (Alexa, Siri) | Modelos acústicos + NLP |
| Filtro de spam | Naive Bayes, redes neurais |
| Auto-complete (IDE) | LLMs |
| Análise de sentimento | NLP |
| Predição de demanda | Regressão, séries temporais |

---

## 🎯 Limitações Importantes

### 1. Garbage In, Garbage Out

Modelo só é tão bom quanto os dados de treino. Dados ruins/enviesados = modelo ruim/enviesado.

### 2. Não generaliza fora da distribuição

Modelo treinado em fotos de gatos domésticos pode falhar com leões.

### 3. Black box

Difícil entender **por que** o modelo decidiu X. Área chamada **Explainability/XAI**.

### 4. Custos

- 💰 Treino de LLM grande: milhões de dólares
- 💰 Inferência: significativa em escala
- ⚡ Energia: data centers de IA são consumidores massivos

### 5. Vieses (bias)

Modelos refletem **vieses dos dados**. Famosos casos:
- Reconhecimento facial pior pra peles escuras
- Modelos de RH preferindo nomes masculinos
- Sistemas de crédito discriminando bairros

---

## 🚀 Por Onde Começar Aprender

### Pra dev curioso (sem virar especialista)

1. **Andrew Ng — Machine Learning Specialization** (Coursera) — clássico, didático
2. **Fast.ai** (curso prático) — ensina a usar ML rápido
3. **3Blue1Brown — Neural Networks** (YouTube) — visualização brilhante

### Pra ir mais fundo

1. *Hands-On Machine Learning* — Aurélien Géron
2. *Deep Learning* — Goodfellow, Bengio, Courville (gratuito online)
3. **CS231n** (Stanford) — Visão computacional
4. **CS224n** (Stanford) — NLP

### Pra brincar

- **Hugging Face** — testar modelos no browser
- **Kaggle** — competições e datasets
- **Google Colab** — GPU grátis pra experimentar

---

## ✅ Resumo da página

- IA ⊃ ML ⊃ Deep Learning ⊃ LLMs (cada um é caso específico do anterior)
- **Supervised** (com rótulos), **Unsupervised** (sem), **RL** (recompensas), **Self-supervised** (cria rótulos)
- Modelos aprendem **ajustando pesos** pra minimizar erro
- Cuidado com **overfitting**: bom no treino, ruim no teste
- Métricas variam por problema: accuracy, precision, recall, F1, AUC
- **Deep Learning** = redes neurais profundas; Transformer é o tipo mais moderno
- Como dev: você **usa APIs**, raramente treina do zero
- IA não é mágica: limitações reais (bias, custos, black box)
- Comece pelo **Andrew Ng** ou **Fast.ai** se quiser aprender de verdade

---

⬅️ [Anterior: Sistemas Distribuídos](./25-sistemas-distribuidos.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: LLMs](./27-llms.md)
