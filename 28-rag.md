# 🗂️ RAG: Retrieval-Augmented Generation

> RAG é a técnica que mais resolve o problema de **alucinação e conhecimento limitado** dos LLMs. Em vez de confiar no que o modelo "lembra", você **busca documentos relevantes** e passa pra ele. Resultado: respostas baseadas em fontes reais e atualizáveis.

---

## 🤔 O Problema Que RAG Resolve

### Cenário 1: Conhecimento desatualizado

```
Você: "Qual a versão mais nova do Spring Boot?"

LLM (knowledge cutoff em 2023):
"Spring Boot 3.2 é a mais recente."
                     ↑ desatualizado
```

### Cenário 2: Conhecimento privado

```
Você: "Qual a política de férias da minha empresa?"

LLM: "Não tenho acesso a essa informação..."
```

### Cenário 3: Alucinação

```
Você: "Cite o paper X sobre tópico Y"

LLM: "Sure! Smith et al. (2019) publicou..."
       ↑ paper inventado
```

---

## 💡 A Solução: RAG

```
1. Usuário faz pergunta
2. Sistema busca documentos relevantes na base de conhecimento
3. Anexa esses documentos ao prompt
4. LLM responde baseado nesses documentos
```

```
┌─────────────────────────┐
│ "Qual a política de     │
│  férias da empresa?"    │
└───────────┬─────────────┘
            ↓
   ┌────────────────┐
   │ Busca em docs  │ → encontra doc.pdf relevante
   └────────┬───────┘
            ↓
   ┌──────────────────────────────┐
   │ Prompt enviado ao LLM:        │
   │                               │
   │ "Baseado nestes documentos:   │
   │ [conteúdo do doc.pdf]         │
   │                               │
   │ Responda: Qual a política     │
   │ de férias da empresa?"        │
   └──────────┬───────────────────┘
              ↓
   ┌────────────────────────┐
   │ Resposta fundamentada  │
   │ em documento real      │
   └────────────────────────┘
```

> 💡 **Em palavras simples**: RAG dá ao LLM uma "memória externa" de documentos. Em vez de inventar, ele responde com base no que você forneceu.

---

## 🧱 Componentes de um Sistema RAG

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  📚 Documentos                                  │
│       ↓                                         │
│  ✂️  Chunking (divide em pedaços)               │
│       ↓                                         │
│  🔢 Embedding (vetorização)                     │
│       ↓                                         │
│  🗄️  Vector Database (armazena)                 │
│                                                 │
└─────────────────────────────────────────────────┘

           Pergunta do usuário
                    ↓
            🔢 Embedding
                    ↓
        🔍 Busca vetorial (top K)
                    ↓
        📋 Prompt com contexto
                    ↓
                🧠 LLM
                    ↓
                 Resposta
```

---

## 🔢 Embeddings: O Coração do RAG

### O que são?

**Vetores numéricos** que representam o significado de um texto. Textos similares têm vetores próximos.

```
"gato"      → [0.2, 0.8, 0.1, ..., 0.5]  (1536 dimensões)
"felino"    → [0.21, 0.79, 0.12, ..., 0.49]  (próximo de "gato")
"automóvel" → [0.7, 0.1, 0.6, ..., 0.2]  (longe de "gato")
```

### Como funciona

Modelos especializados (não LLMs comuns) convertem texto → vetor:
- **OpenAI**: `text-embedding-3-small`, `text-embedding-3-large`
- **Cohere**: `embed-multilingual-v3`
- **Sentence Transformers** (open source): `all-MiniLM-L6-v2`

### Buscar similaridade

```python
from numpy import dot
from numpy.linalg import norm

def cosine_similarity(a, b):
    return dot(a, b) / (norm(a) * norm(b))

# 1.0 = idêntico, 0 = não relacionado, -1 = oposto
```

> 💡 **Em apps reais**, você usa um **vector database** que faz isso eficientemente em milhões de vetores.

---

## ✂️ Chunking: Dividindo Documentos

LLMs têm limites de contexto, e busca vetorial funciona melhor em **pedaços pequenos**.

### Estratégias

#### 1. Fixed-size chunks
```
Divide em pedaços de N tokens (ex: 512)
Simples, mas pode quebrar no meio de uma frase
```

#### 2. Sentence-based
```
Divide por sentenças completas
Melhor que fixed-size
```

#### 3. Recursive (LangChain)
```
Tenta dividir por parágrafos. Se muito grande, divide por sentenças.
Se ainda grande, por palavras.
```

#### 4. Semantic chunking
```
Usa embeddings pra detectar mudanças de tópico e dividir lá
Mais sofisticado, melhor qualidade
```

### Overlap

Importante manter **overlap** entre chunks pra não perder contexto:

```
Doc completo: "...frase A. frase B. frase C. frase D..."

Chunk 1: "frase A. frase B. frase C."
Chunk 2: "frase B. frase C. frase D."  ← overlap mantém contexto
```

> 🎯 **Tamanho típico**: 256-1024 tokens por chunk, overlap de 10-20%.

---

## 🗄️ Vector Databases

Bancos otimizados pra **busca por similaridade vetorial**.

### Principais opções

| Database | Tipo | Notas |
|---|---|---|
| **Pinecone** | Cloud SaaS | Mais popular, gerenciado |
| **Weaviate** | Open / Cloud | GraphQL, multi-modal |
| **Qdrant** | Open / Cloud | Rust, performante |
| **Chroma** | Open | Simples, ótimo pra começar |
| **Milvus** | Open | Distribuído, escalável |
| **pgvector** | Extensão do Postgres | ✨ Sem precisar de DB extra |
| **Redis Vector** | Extensão do Redis | Bom se já usa Redis |

### Exemplo: pgvector (PostgreSQL)

```sql
-- Habilitar extensão
CREATE EXTENSION vector;

-- Tabela com coluna vetor
CREATE TABLE documentos (
    id SERIAL PRIMARY KEY,
    conteudo TEXT,
    embedding vector(1536)
);

-- Inserir
INSERT INTO documentos (conteudo, embedding)
VALUES ('Política de férias...', '[0.1, 0.2, ...]');

-- Buscar 5 mais similares
SELECT conteudo
FROM documentos
ORDER BY embedding <=> '[0.15, 0.18, ...]'
LIMIT 5;
```

> 💡 **Recomendação**: se você já usa Postgres, comece com **pgvector**. Evita adicionar nova infra.

---

## 🔍 Tipos de Busca

### Dense Retrieval (vetorial)

Usa embeddings. Encontra significado **semelhante** mesmo com palavras diferentes.

```
Pergunta: "Como faço home office?"
Encontra: "trabalho remoto", "trabalho de casa"
```

### Sparse Retrieval (BM25, keyword)

Busca tradicional por palavras-chave (algoritmo BM25).

```
Pergunta: "Como faço home office?"
Encontra: textos com "home office" exato
```

### Hybrid Retrieval (recomendado)

Combina os dois. **Geralmente dá melhores resultados**.

```
1. Busca BM25 → top 50 docs
2. Busca vetorial → top 50 docs
3. Combina e re-ranqueia
4. Retorna top 5
```

### Re-ranking

Após primeira busca, usa modelo mais lento/preciso pra **reordenar** os top resultados.

- **Cohere Rerank**, **Cross-encoders**

---

## 🛠️ Implementação: Exemplo Simples

### Stack Python

```python
# pip install langchain openai chromadb

from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.document_loaders import TextLoader

# 1. Carrega documento
loader = TextLoader("politicas-empresa.txt")
docs = loader.load()

# 2. Divide em chunks
splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = splitter.split_documents(docs)

# 3. Embeddings + indexa em Chroma
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)

# 4. Busca
question = "Qual a política de férias?"
relevant_docs = vectorstore.similarity_search(question, k=3)

# 5. Monta prompt e envia ao LLM
context = "\n\n".join([doc.page_content for doc in relevant_docs])
prompt = f"""Baseado nestes documentos:

{context}

Responda: {question}"""

# Chama LLM (OpenAI, Claude, etc) com esse prompt
```

### Stack Java/Spring

```xml
<!-- Spring AI -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
</dependency>
```

```java
@Service
public class RagService {
    private final VectorStore vectorStore;
    private final ChatClient chatClient;

    public String responder(String pergunta) {
        // 1. Busca docs relevantes
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(pergunta).withTopK(3)
        );

        // 2. Monta contexto
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));

        // 3. Chama LLM
        return chatClient.prompt()
            .user(u -> u.text("""
                Baseado nestes documentos:
                {context}

                Responda: {pergunta}
                """).param("context", context).param("pergunta", pergunta))
            .call()
            .content();
    }
}
```

---

## 🚀 Padrões Avançados

### Multi-Query RAG

LLM gera **múltiplas variações** da pergunta antes de buscar.

```
Pergunta: "Como economizar no AWS?"

LLM gera:
- "Reduzir custos AWS"
- "Otimização de infraestrutura cloud"
- "Best practices billing AWS"

Busca cada variação, combina resultados.
```

### Hypothetical Document Embeddings (HyDE)

Em vez de embeddar a pergunta, LLM **imagina uma resposta**, embedda essa resposta hipotética e busca.

> 🤔 **Por quê funciona?** Pergunta e resposta têm embeddings diferentes. Resposta hipotética se parece mais com docs reais que a resposta deve ter.

### Contextual Retrieval (Anthropic)

Antes de embeddar cada chunk, adiciona contexto sobre **de onde ele veio**.

```
Chunk original: "A taxa é de 15%."

Chunk com contexto: "Documento: Política de Comissões 2024.
Seção: Vendas Internacionais. A taxa é de 15%."
```

Resultado: 49% menos falhas de retrieval.

### Agentic RAG

LLM decide **quando buscar**, **o que buscar** e **se precisa de mais buscas**.

```
1. Pergunta chega
2. LLM decide: "preciso buscar X"
3. Busca, lê resultado
4. LLM decide: "agora preciso de Y"
5. ... até ter resposta completa
```

---

## ⚠️ Desafios Reais

### 1. Qualidade dos chunks

Chunks ruins = retrieval ruim = respostas ruins. **Investe nessa parte**.

### 2. Embeddings multilíngues

Modelo OpenAI funciona OK em português, mas modelos especializados (Cohere multilingual) podem ser melhores.

### 3. Atualização dos dados

Como manter o vector DB sincronizado com mudanças nos documentos?

### 4. Custos

Embedding de milhões de chunks custa. Embedding na busca também (cada query é uma chamada).

### 5. Avaliação

Como saber se o RAG está bom? Métricas:
- **Faithfulness**: resposta é baseada nos docs?
- **Relevance**: docs recuperados são relevantes?
- **Recall@K**: % de queries com doc certo nos top K

Frameworks: **RAGAS**, **LangSmith**, **TruLens**.

---

## 🎯 RAG vs Fine-tuning

Pergunta comum: "devo fazer RAG ou fine-tunar o modelo?"

| Aspecto | RAG | Fine-tuning |
|---|---|---|
| **Atualização** | ✅ Fácil (atualiza docs) | ❌ Re-treina |
| **Custo** | Médio | Alto |
| **Conhecimento dinâmico** | ✅ | ❌ |
| **Estilo / formato** | Limitado | ✅ Pode ensinar |
| **Domínios novos** | ✅ | ✅ (caro) |

> 💡 **Regra prática**: comece com RAG. Fine-tuning só se RAG não for suficiente.

---

## ✅ Resumo da página

- **RAG** mitiga alucinação e adiciona conhecimento privado/atualizado a LLMs
- Componentes: **chunking, embeddings, vector DB, retrieval, prompt augmentation**
- **Embeddings** transformam texto em vetores; similaridade = cosseno
- **Chunking** importa muito; use overlap; semantic chunking é melhor
- **Vector DBs**: Pinecone, Weaviate, Qdrant, Chroma, **pgvector** (recomendado se já usa Postgres)
- **Hybrid retrieval** (vetorial + keyword) > só vetorial
- **Re-ranking** melhora qualidade dos top resultados
- Stacks: **LangChain (Python)**, **Spring AI (Java)**, **LlamaIndex** (Python)
- Padrões avançados: Multi-query, HyDE, Contextual Retrieval, Agentic RAG
- **RAG > fine-tuning** na maioria dos casos

---

⬅️ [Anterior: LLMs](./27-llms.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Engenharia de Prompt](./29-engenharia-prompt.md)
