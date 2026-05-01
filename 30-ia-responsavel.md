# 🛡️ IA com Responsabilidade

> Construir com IA é fácil. Construir com IA **de forma responsável** é difícil. Este tópico cobre os principais riscos, vieses e cuidados que separam projetos sérios de armadilhas que viram problema legal, financeiro ou ético.

---

## 🎯 Por Que Importa

IA não é "neutra". Decisões automatizadas têm impactos reais:

- ❌ Sistemas de crédito recusando empréstimos a minorias
- ❌ Reconhecimento facial errando 10x mais com pessoas negras
- ❌ Chatbots dando conselhos médicos perigosos
- ❌ Modelos de RH preferindo nomes masculinos
- ❌ Geração de conteúdo difamatório sobre pessoas reais
- ❌ Vazamento de dados confidenciais via APIs de IA

> 💡 **Como dev**, você é o responsável técnico. "A IA decidiu" não é defesa válida.

---

## 👻 Hallucination (Alucinação)

LLMs **inventam informação** com confiança. Mais grave em domínios críticos.

### Onde dói mais

- ⚕️ **Medicina**: "tome 500mg de X" (inventa dose, doença, contraindicação)
- ⚖️ **Direito**: cita jurisprudência que não existe (caso real em 2023: advogado dos EUA usou ChatGPT, citou casos inexistentes em corte)
- 💰 **Financeiro**: estatísticas, números, recomendações inventadas
- 📚 **Educação**: fatos históricos errados, conceitos confundidos
- 🔬 **Pesquisa**: citações de papers que não existem

### Mitigações

| Estratégia | Como funciona |
|---|---|
| 🗂️ **RAG** | Força resposta baseada em docs reais |
| 🌡️ **Temperature 0** | Respostas mais determinísticas |
| 📋 **Structured output** | JSON schema valida formato |
| ✅ **Verificação** | Pedir fontes verificáveis |
| 🤝 **Human-in-the-loop** | Humano valida antes de agir |
| 🧪 **Validação cruzada** | Outro modelo verifica primeiro |

### O que NÃO fazer

❌ **Confiar 100%** em qualquer output de IA pra decisão crítica
❌ **Apresentar como fato** sem checar
❌ **Automatizar ações irreversíveis** sem validação humana

---

## ⚖️ Bias (Vieses)

LLMs herdam vieses dos dados de treino.

### Tipos comuns

#### Vieses de gênero
```
"O médico disse que..."           → assume homem
"A enfermeira chegou..."          → assume mulher
"A engenheira de software..."     → menos comum nos dados
```

#### Vieses raciais e étnicos
```
- Reconhecimento facial mais preciso em peles claras
- Modelos associam crimes a certos grupos étnicos
- Tradução muda gênero dependendo de profissão (turco → inglês)
```

#### Vieses culturais
```
- Predominância de cultura ocidental/anglófona
- Padrões norte-americanos como "neutros"
- Subrepresentação de outras línguas e contextos
```

#### Vieses históricos
```
- Dados de currículos antigos refletem discriminação histórica
- IA aprende padrões que **devemos** evitar repetir
```

### Caso famoso: Amazon

Em 2014-2018, Amazon construiu um sistema de IA pra triar currículos. Treinou com dados históricos. Resultado: **rejeitava currículos de mulheres** porque o histórico era predominantemente masculino. Projeto cancelado.

### Mitigações

- 📊 **Auditar datasets** de treino e fine-tuning
- 🧪 **Testes de equidade** (fairness tests) com grupos diversos
- 👥 **Diversidade no time** que constrói (vê problemas que outros não veem)
- 📈 **Métricas por subgrupo** (não só média)
- 📝 **Documentar limitações** conhecidas

---

## 🔐 Privacidade e Vazamento de Dados

### Riscos principais

#### 1. Dados enviados pra APIs

Quando você manda dados pro ChatGPT/Claude:
- Provedor pode (ou não) usar pra treinar modelos futuros
- Logs podem ser auditados
- Vazamento se provedor for hackeado

#### 2. Memorização de treinos

Modelos treinados com dados privados podem **vazar** esses dados:
```
Prompt: "Continue: 'meu CPF é..."
Modelo (treinado com dados sensíveis): vaza CPF real
```

#### 3. Prompt injection extraindo dados

Atacante manipula LLM pra revelar **system prompt** com info confidencial.

### Boas práticas

#### Para APIs comerciais (OpenAI, Anthropic, etc)

- ✅ Use modos **enterprise** (não usam dados pra treino)
- ✅ Anonimize dados sensíveis **antes** de enviar
- ✅ Use **Zero Data Retention** quando disponível
- ✅ Leia ToS sobre data usage
- ❌ **Nunca** envie senhas, tokens, dados de cartão

#### Para LLMs próprios (open-source)

- ✅ Cuidado com dados de fine-tuning (cuidado com PII)
- ✅ Testes pra garantir que modelo não vaza dados sensíveis
- ✅ Logs com retenção limitada

#### Para apps de RAG

- ✅ Permissionamento: usuário só vê docs que tem acesso
- ✅ Filtra PII antes de embeddar
- ✅ Audit logs

### Compliance

- 🇪🇺 **GDPR** (Europa): direito de explicação, deleção, portabilidade
- 🇧🇷 **LGPD** (Brasil): similar ao GDPR
- 🇺🇸 **CCPA** (Califórnia): direitos do consumidor
- 🇪🇺 **EU AI Act**: regulamentação específica de IA (em vigor)

---

## 💰 Custos e Sustentabilidade

### Custos que escalam rápido

```
1.000 requests/dia × $0.01 = $10/dia = $300/mês
1.000.000 requests/dia × $0.01 = $10.000/dia = $300.000/mês
```

### Estratégias pra controlar custos

#### 1. Cache agressivo

Pergunta repetida? Não chama o LLM, devolve resposta cacheada.

#### 2. Use modelo certo pro problema

Não use Opus pra coisa simples. Use Haiku/Mini.

#### 3. Streaming + early stop

Pare de gerar quando achar a resposta.

#### 4. Limite de tokens (max_tokens)

Evita respostas gigantes desnecessárias.

#### 5. Batch processing

Quando possível, agrupar requests.

#### 6. Self-hosted pra alto volume

Acima de certo volume, vale rodar modelo open-source em GPU própria.

#### 7. Monitore por usuário

Detectar abuse precocemente.

### Sustentabilidade ambiental

- 🌍 Treino de LLMs grandes consome energia equivalente a centenas de casas/ano
- ⚡ Inferência em escala também é significativa
- 💚 Considere quando IA realmente agrega valor vs solução simples

---

## ⚠️ Limites do Que Construir

Nem tudo deve ser IA. Pergunte-se:

### Deve ser IA?

✅ **Sim, IA agrega**:
- Tarefas com input não-estruturado (texto livre, imagem)
- Padrões complexos, difíceis de codificar em regras
- Personalização em escala
- Tradução, sumário, criação de conteúdo

❌ **Não, use código tradicional**:
- Cálculos exatos (matemática, financeiro)
- Decisões que precisam ser auditáveis 100%
- Lógica simples e bem definida
- Quando custo > benefício

### Existe risco de dano?

🚨 **Áreas que requerem cuidado extra**:
- Saúde, direito, finanças (impacto direto na vida)
- Sistemas que negam serviços (crédito, emprego, seguro)
- Decisões automatizadas sem revisão humana
- Sistemas voltados a crianças

> 💡 **Princípio**: quanto mais dano potencial, mais salvaguardas. Sempre tenha **humano no loop** em decisões críticas.

---

## 🤖 Agentes Autônomos: Cuidado Redobrado

Agentes que **executam ações** (não só geram texto) trazem novos riscos.

### Exemplos

- Agente que envia emails
- Agente que faz transações financeiras
- Agente que modifica código em produção
- Agente que interage com APIs externas

### Riscos

1. **Loop infinito**: agente fica gastando dinheiro/recursos
2. **Ações destrutivas**: deletar dados, fazer transações erradas
3. **Cascata de erros**: 1 erro vira 10, vira 100
4. **Prompt injection**: usuário malicioso "sequestra" agente
5. **Responsabilidade**: quem responde quando agente faz besteira?

### Salvaguardas mínimas

- ✅ **Sandbox**: ambiente isolado primeiro
- ✅ **Limites duros**: máximo de ações por execução
- ✅ **Aprovação humana**: pra ações irreversíveis
- ✅ **Audit logs**: tudo registrado
- ✅ **Rollback**: poder desfazer
- ✅ **Rate limits**: não saturar recursos
- ✅ **Dry run**: testar sem executar de verdade
- ✅ **Monitoramento**: alertas pra comportamento anômalo

---

## 📜 Transparência e Direitos do Usuário

### Comunicar claramente

- ✅ Informe quando usuário está falando com IA, não humano
- ✅ Documente capacidades e limitações
- ✅ Explique como decisões automatizadas funcionam
- ✅ Forneça canal pra contestação

### Direito de explicação

Em decisões automatizadas com impacto:
- Por que essa decisão?
- Quais fatores pesaram?
- Como contestar?

> 💡 **GDPR e LGPD exigem isso** pra decisões automatizadas. Considere desde o design.

---

## 🧪 Testes e Avaliação

### Não basta "deu certo no teste manual"

Sistemas de IA precisam:

#### 1. Test sets diversos

Casos que cobrem:
- Casos típicos
- Casos extremos
- Grupos minoritários (fairness)
- Inputs adversariais
- Erros comuns esperados

#### 2. Métricas relevantes

Não só acurácia. Considere:
- **Acurácia por subgrupo** (gênero, raça, etc)
- **Precision/Recall** específicos do problema
- **Calibração** (modelo "sabe quanto não sabe"?)

#### 3. Avaliação contínua

Modelos podem **degradar com o tempo**:
- Distribuição dos dados muda
- Comportamento de usuários muda
- Modelo desatualizado

→ **Monitore performance em produção**.

---

## 🔓 Open Source vs Commercial

### Modelos abertos (Llama, Mistral, etc)

✅ **Privacidade**: dados não saem da empresa
✅ **Customização**: pode fine-tunar
✅ **Sem vendor lock-in**
✅ **Auditável** (em teoria — pesos são públicos)
❌ **Custo de infra** (GPUs)
❌ **Geralmente menos performantes** que SOTA comercial
❌ **Time pra operar**

### Modelos comerciais (GPT, Claude)

✅ **Performance estado-da-arte**
✅ **Sem infra pra gerenciar**
✅ **Atualizações automáticas**
❌ **Privacidade** (dados saem)
❌ **Custos imprevisíveis**
❌ **Dependência do provedor**
❌ **Mudanças unilaterais** (provedor pode mudar/descontinuar)

### Como escolher

```
Volume baixo + protótipo → Comercial (rápido)
Volume médio + sensibilidade → Comercial enterprise
Volume alto + privacidade crítica → Self-hosted
Compliance estrito (saúde, gov) → Self-hosted
```

---

## 🎯 Checklist Mínimo de IA Responsável

Antes de colocar sistema com IA em produção:

- [ ] Identificou **casos de erro** principais e mitigou?
- [ ] Testou com **dados diversos** (gênero, raça, idade, idioma)?
- [ ] Definiu **limites duros** (max custos, max ações)?
- [ ] Tem **humano no loop** pra decisões críticas?
- [ ] **Documentou limitações** conhecidas pros usuários?
- [ ] **Privacidade**: dados sensíveis estão protegidos?
- [ ] **Compliance**: GDPR/LGPD/setor específico?
- [ ] **Audit logs**: pode auditar decisões depois?
- [ ] **Rollback**: pode desfazer ações erradas?
- [ ] **Alertas**: monitora comportamento anômalo?
- [ ] **Custo**: entende e controla gastos?
- [ ] **Plano de contestação** pra usuários afetados?
- [ ] **Plano de incidente** se algo der errado?

---

## 📚 Recursos Pra Ir Mais Fundo

### Frameworks

- 📋 **NIST AI Risk Management Framework**
- 🌍 **EU AI Act**
- 📜 **OECD AI Principles**

### Livros

- 📕 *Weapons of Math Destruction* — Cathy O'Neil
- 📗 *Atlas of AI* — Kate Crawford
- 📘 *The Alignment Problem* — Brian Christian

### Sites

- 🎓 [AI Ethics Lab](https://aiethicslab.com/)
- 🔍 [Algorithmic Justice League](https://www.ajl.org/)
- 📊 [Partnership on AI](https://partnershiponai.org/)

---

## ✅ Resumo da página

- IA não é neutra: **decisões têm impactos reais** em pessoas
- **Hallucination** é risco principal de LLMs — mitigue com RAG, validação, humano no loop
- **Bias** vem dos dados; teste com **subgrupos diversos**
- **Privacidade**: nunca envie dados sensíveis sem cuidado; conheça GDPR/LGPD
- **Custos crescem rápido** — cache, modelos certos, monitoramento
- **Nem tudo deve ser IA** — solução simples às vezes é melhor
- **Agentes autônomos** = cuidados extras (sandbox, limites, aprovação humana)
- **Transparência**: avise quando é IA, explique decisões, permita contestação
- **Open vs Commercial**: trade-off de privacidade, custo, performance
- Use o **checklist** antes de produção

---

⬅️ [Anterior: Engenharia de Prompt](./29-engenharia-prompt.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Integração com APIs de LLMs](./31-integracao-llms.md)
