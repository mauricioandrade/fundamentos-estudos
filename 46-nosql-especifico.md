# 🗃️ Bancos NoSQL Específicos

> "NoSQL" é um termo guarda-chuva pra **vários tipos** de bancos diferentes que **não são relacionais**. Cada um tem casos onde brilha e casos onde fracassa. Entender as diferenças entre **MongoDB, Cassandra, DynamoDB, Redis, Neo4j, Elasticsearch** te dá superpoderes pra escolher a ferramenta certa pro problema certo.

---

## 🤔 NoSQL: Quando e Por Que

### Limitações dos bancos relacionais

- 🐢 **Schema rígido**: mudanças são caras
- 📈 **Escala vertical**: aumentar máquina, não horizontal
- 🔗 **Joins** ficam lentos com bilhões de linhas
- 🌍 **Distribuição** é complicada (transações ACID em múltiplos nós = lento)

### NoSQL surgiu pra resolver

- ✅ **Schema flexível** (ou ausente)
- ✅ **Escala horizontal** (adicione nodes)
- ✅ **Casos específicos** (grafos, busca, time-series, etc)

> ⚠️ **Não é "melhor" que SQL**. É **diferente**. Postgres bem-feito ainda resolve 90% dos casos. Use NoSQL quando o problema **realmente** justifica.

---

## 📊 Categorias de NoSQL

| Tipo | Exemplos | Caso típico |
|---|---|---|
| **Documento** | MongoDB, CouchDB | Dados semi-estruturados, JSONs |
| **Key-Value** | Redis, DynamoDB | Cache, sessões, lookups rápidos |
| **Wide-column** | Cassandra, ScyllaDB, HBase | Big data, write-heavy |
| **Grafo** | Neo4j, ArangoDB | Relacionamentos complexos |
| **Time-series** | InfluxDB, TimescaleDB | Métricas, IoT, monitoramento |
| **Search** | Elasticsearch, Solr | Busca full-text, logs |
| **Vector** | Pinecone, Milvus, pgvector | Embeddings, IA semântica |

---

## 📄 MongoDB (Documento)

### Conceito

Armazena **documentos JSON** (na verdade BSON — Binary JSON).

```javascript
// Coleção "usuarios"
{
  "_id": ObjectId("507f..."),
  "nome": "Mauricio",
  "email": "mauricio@example.com",
  "enderecos": [
    { "tipo": "casa", "rua": "Rua A" },
    { "tipo": "trabalho", "rua": "Rua B" }
  ],
  "preferencias": {
    "idioma": "pt-BR",
    "tema": "escuro"
  }
}
```

> 💡 **Diferença chave do SQL**: aqui você tem **arrays e objetos aninhados** dentro do documento. Sem precisar de tabelas separadas com JOINs.

### Operações básicas

```javascript
// Criar
db.usuarios.insertOne({ nome: "Ana", email: "ana@x.com" });

// Ler
db.usuarios.findOne({ email: "ana@x.com" });
db.usuarios.find({ idade: { $gt: 18 } });

// Atualizar
db.usuarios.updateOne(
  { _id: ObjectId("...") },
  { $set: { nome: "Ana Maria" } }
);

// Deletar
db.usuarios.deleteOne({ _id: ObjectId("...") });

// Aggregation Pipeline (poderoso!)
db.pedidos.aggregate([
  { $match: { status: "concluido" } },
  { $group: { _id: "$cliente_id", total: { $sum: "$valor" } } },
  { $sort: { total: -1 } },
  { $limit: 10 }
]);
```

### Quando usar MongoDB

✅ **Dados semi-estruturados** que mudam (perfis, configurações, logs)
✅ **Catálogos** com atributos variáveis (cada produto tem campos diferentes)
✅ **APIs JSON** (mapeamento direto)
✅ **Prototipagem rápida** (sem schema fixo)

### Quando NÃO usar

❌ **Transações complexas multi-documento** (existe mas é limitado)
❌ **Joins frequentes** ($lookup é lento)
❌ **Reports analíticos** (use DW)
❌ **Schema realmente fixo** (Postgres + jsonb pode ser melhor)

### Em Java

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
</dependency>
```

```java
MongoClient mongo = MongoClients.create("mongodb://localhost:27017");
MongoDatabase db = mongo.getDatabase("minha_db");
MongoCollection<Document> usuarios = db.getCollection("usuarios");

usuarios.insertOne(new Document("nome", "Ana").append("idade", 30));
Document user = usuarios.find(eq("nome", "Ana")).first();
```

Com Spring Data MongoDB:

```java
@Document("usuarios")
public record Usuario(
    @Id String id,
    String nome,
    String email,
    List<Endereco> enderecos
) {}

public interface UsuarioRepository extends MongoRepository<Usuario, String> {
    Optional<Usuario> findByEmail(String email);
}
```

---

## 🚀 Redis (Key-Value)

Já visto em [Caching](./20-caching.md), mas vale mais profundidade aqui.

### Tipos de dados

```bash
# String
SET nome "Mauricio"
GET nome

# Hash (objeto)
HSET usuario:123 nome "Ana" email "ana@x.com"
HGET usuario:123 nome

# List (fila)
LPUSH fila_emails "email1"
RPOP fila_emails

# Set (conjunto único)
SADD tags "java" "spring" "redis"
SMEMBERS tags

# Sorted Set (ranking)
ZADD ranking 100 "alice"
ZADD ranking 250 "bob"
ZRANGE ranking 0 -1 WITHSCORES

# Stream (eventos)
XADD eventos * tipo "login" usuario "alice"

# HyperLogLog (contagem aproximada de únicos)
PFADD visitantes "user1" "user2" "user1"
PFCOUNT visitantes  # 2

# Pub/Sub
SUBSCRIBE canal:notificacoes
PUBLISH canal:notificacoes "olá!"

# Geospatial
GEOADD lugares -23.5 -46.6 "São Paulo"
GEORADIUS lugares -23.5 -46.6 100 km
```

### Quando usar Redis

✅ **Cache** (caso clássico)
✅ **Sessões de usuário**
✅ **Rate limiting**
✅ **Filas leves**
✅ **Leaderboards** (sorted sets)
✅ **Pub/Sub** simples

### Limitações

❌ Tudo em memória (caro pra TBs)
❌ Não é DB primário (use pra acelerar/complementar)

---

## 🌐 Apache Cassandra (Wide-Column)

Criado pelo Facebook, agora Apache. Usado por Netflix, Apple, Uber.

### Conceito

Combina ideias de DynamoDB (distribuição) e BigTable (modelo). **Otimizado pra writes em massa**.

### Modelo de dados

```cql
-- CQL (Cassandra Query Language) — parecido com SQL
CREATE TABLE eventos_usuario (
    usuario_id UUID,
    timestamp TIMESTAMP,
    evento_tipo TEXT,
    dados TEXT,
    PRIMARY KEY (usuario_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

### Particionamento

```
PRIMARY KEY (usuario_id, timestamp)
              ↑              ↑
         partition key    clustering key
         (qual node)      (ordem dentro do node)
```

Eventos do mesmo usuário ficam **juntos no mesmo node**, ordenados por tempo.

### Pontos fortes

- ⚡ **Writes super rápidos**
- 🌍 **Multi-datacenter** nativo
- 🔄 **Sem master**, todos os nodes são iguais
- 📈 **Escala linear** (dobra nodes = dobra capacidade)
- 🛡️ **Alta disponibilidade**

### Pontos fracos

- ❌ Queries **limitadas** (precisa modelar pelas queries)
- ❌ Sem JOINs
- ❌ ORDER BY só dentro da partition
- ❌ Eventually consistent (configurável)

### Quando usar

✅ **Time-series massivo** (logs, eventos, IoT)
✅ **Write-heavy** workloads (milhões de writes/s)
✅ **Multi-região** (replicação geográfica)
✅ **Histórico imutável** (append-only)

### ScyllaDB

Reescrita do Cassandra em C++. **10x mais rápido** com mesmo modelo.

> 💡 **Nova realidade**: muitos times migrando Cassandra → ScyllaDB pra performance.

---

## ☁️ DynamoDB (AWS)

NoSQL key-value/documento gerenciado da AWS. **Sem servidor**, escala automática.

### Modelo

```
Tabela "usuarios":
├── Partition Key: usuario_id
└── Sort Key (opcional): timestamp

Item exemplo:
{
  "usuario_id": "abc123",     ← partition key
  "timestamp": 1735689600,     ← sort key
  "nome": "Mauricio",
  "email": "...",
  "preferencias": { ... }
}
```

### Acesso

```javascript
// Put
await dynamodb.put({
  TableName: 'usuarios',
  Item: { usuario_id: 'abc', nome: 'Ana' }
});

// Get
await dynamodb.get({
  TableName: 'usuarios',
  Key: { usuario_id: 'abc' }
});

// Query (mais flexível)
await dynamodb.query({
  TableName: 'eventos',
  KeyConditionExpression: 'usuario_id = :u AND timestamp > :t',
  ExpressionAttributeValues: { ':u': 'abc', ':t': 1735000000 }
});
```

### Pontos fortes

- ☁️ **Totalmente gerenciado** (sem ops)
- 📈 **Escala automática** (provisioned ou on-demand)
- 🌍 **Global Tables** (multi-região com 1 click)
- ⚡ **Latência consistente** (single-digit ms)

### Pontos fracos

- 💰 **Custo alto** em escala
- 🔒 **Vendor lock-in** AWS
- ❌ Queries **muito limitadas** (precisa secondary indexes)
- 📐 **Modelagem é arte**: errou design, sofre depois

### DynamoDB single-table design

Padrão polêmico mas poderoso: **uma tabela** pra **toda aplicação**.

```
PK              | SK              | Attrs...
USER#abc        | PROFILE         | nome, email
USER#abc        | ORDER#123       | total, items
ORDER#123       | ITEM#1          | produto, qtd
```

✅ Performance ótima (1 query = todos os dados)
❌ Curva de aprendizado brutal

> 💡 Pra começar com DynamoDB, leia "DynamoDB Book" do Alex DeBrie. É praticamente obrigatório.

---

## 🕸️ Neo4j (Grafos)

Banco **otimizado pra relacionamentos**.

### Conceito

Em SQL, relacionamentos são via **JOINs**. Em grafos, são **first-class citizens**.

```
(Pessoa { nome: "Ana" }) -[CONHECE { desde: 2020 }]-> (Pessoa { nome: "Bob" })
       nó                    relacionamento                    nó
```

### Cypher (linguagem)

```cypher
// Criar
CREATE (a:Pessoa {nome: "Ana"})
CREATE (b:Pessoa {nome: "Bob"})
CREATE (a)-[:CONHECE {desde: 2020}]->(b)

// Buscar
MATCH (a:Pessoa)-[:CONHECE]->(b:Pessoa)
WHERE a.nome = "Ana"
RETURN b.nome

// Pergunta clássica de grafo: amigos de amigos
MATCH (eu:Pessoa {nome: "Ana"})-[:CONHECE]->(amigo)-[:CONHECE]->(amigo_de_amigo)
WHERE NOT (eu)-[:CONHECE]->(amigo_de_amigo)
RETURN DISTINCT amigo_de_amigo.nome
```

### Quando usar

✅ **Redes sociais** (LinkedIn, Facebook)
✅ **Recomendações baseadas em conexões**
✅ **Detecção de fraude** (padrões de transações)
✅ **Knowledge graphs**
✅ **Roteamento, GPS**

### Quando NÃO

❌ Casos sem relacionamentos complexos
❌ Volumes massivos de dados simples
❌ Workloads transacionais comuns

> 💡 Em SQL, "amigos de amigos de amigos" é JOIN aninhado horrível. Em grafos, é trivial. Essa é a vantagem.

---

## 📈 InfluxDB / TimescaleDB (Time-Series)

Otimizados pra **dados com timestamp**.

### Casos típicos

- 📊 Métricas de aplicação
- 🌡️ Sensores IoT
- 💹 Cotações de ações
- 📡 Logs com timestamp

### TimescaleDB

Extension do **PostgreSQL**. Mantém compatibilidade SQL total + otimizações pra time-series.

```sql
CREATE TABLE metricas (
    tempo TIMESTAMPTZ NOT NULL,
    sensor_id TEXT,
    valor DOUBLE PRECISION
);

-- Transforma em hypertable (otimizado pra time-series)
SELECT create_hypertable('metricas', 'tempo');

-- Queries SQL normais, mas otimizadas
SELECT
    time_bucket('5 minutes', tempo) AS intervalo,
    AVG(valor)
FROM metricas
WHERE tempo > NOW() - INTERVAL '1 hour'
GROUP BY intervalo
ORDER BY intervalo DESC;
```

### InfluxDB

Especializado, com query language própria (Flux).

```flux
from(bucket: "metricas")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> aggregateWindow(every: 5m, fn: mean)
```

> 💡 **TimescaleDB é mais "SQL-friendly"**, InfluxDB é mais especializado. Pra times já com Postgres, TimescaleDB ganha.

---

## 🔍 Elasticsearch (Search)

Não é exatamente "NoSQL", mas é o **search engine** mais usado.

### Pra quê serve

- 🔎 **Full-text search** (Google-like)
- 📜 **Logs centralizados** (ELK stack: Elasticsearch + Logstash + Kibana)
- 📊 **Analytics em tempo real**
- 🛒 **Search em e-commerce** (relevância, filtros, fuzzy)

### Modelo

```json
// Índice "produtos"
{
  "id": "p123",
  "nome": "Camiseta polo azul masculina",
  "descricao": "Camiseta de algodão, modelo polo, cor azul",
  "preco": 89.90,
  "categorias": ["roupas", "masculino", "casual"]
}
```

### Query (DSL)

```json
GET produtos/_search
{
  "query": {
    "multi_match": {
      "query": "camiseta polo azul",
      "fields": ["nome^2", "descricao"]   // ^2 = nome 2x mais importante
    }
  },
  "filter": [
    { "range": { "preco": { "lte": 100 } } },
    { "term": { "categorias": "masculino" } }
  ]
}
```

### Por que Elasticsearch é poderoso

- ✅ **Fuzzy matching** (encontra mesmo com erros de digitação)
- ✅ **Relevance scoring** (ordena por relevância)
- ✅ **Aggregations** (analytics)
- ✅ **Realtime** (segundos pra indexar)

### Alternativas

- **OpenSearch**: fork do ES (após mudança de licença)
- **Algolia**: SaaS, mais fácil mas pago
- **Typesense**: open source, mais simples
- **Meilisearch**: moderno, leve

---

## 🧠 Bancos Vetoriais

Categoria que explodiu com **LLMs e RAG** ([visto no tópico 28](./28-rag.md)).

### O que armazenam

**Embeddings**: vetores de números que representam significado.

```python
"gato" → [0.2, 0.8, -0.3, 0.5, ...]   (1536 dimensões em GPT)
"felino" → [0.21, 0.79, -0.28, 0.51, ...] (vetor parecido, similar)
"avião" → [-0.5, 0.1, 0.9, ...] (vetor bem diferente)
```

### Operação principal: similarity search

```python
# Busca os 5 mais similares ao vetor de "filhote de cachorro"
resultados = db.query(
    vector=embedding("filhote de cachorro"),
    top_k=5
)
# Retorna: cachorrinho, cão, pet, animal de estimação, husky
```

### Principais ferramentas

| Tool | Tipo |
|---|---|
| **Pinecone** | SaaS, gerenciado |
| **Weaviate** | Open source, completo |
| **Milvus / Zilliz** | Open source, escalável |
| **Qdrant** | Open source, Rust |
| **Chroma** | Open source, embarcado |
| **pgvector** | Extension do Postgres |

> 💡 **pgvector** está virando default: você usa o Postgres que já tem, com superpoderes vetoriais.

---

## 🎯 Como Escolher

### Decisão prática

```
Preciso buscar texto com relevância?
  → Elasticsearch / OpenSearch

Tenho dados com timestamp e séries temporais?
  → TimescaleDB / InfluxDB

Modelo dados altamente relacional (graph)?
  → Neo4j

Cache, sessões, rankings?
  → Redis

Embeddings pra IA?
  → pgvector (simples) / Pinecone (SaaS)

Documentos JSON com schema variado?
  → MongoDB
  (mas considere Postgres + JSONB primeiro!)

Write-heavy massivo, multi-região?
  → Cassandra / ScyllaDB

AWS, sem ops, escala automática?
  → DynamoDB

Caso comum, transações ACID, queries flexíveis?
  → PostgreSQL (sério, considere isso primeiro)
```

### Polyglot Persistence

Apps modernas usam **múltiplos bancos** pra casos diferentes:

```
PostgreSQL   → dados transacionais
Redis        → cache + sessões
Elasticsearch → busca full-text
S3           → arquivos grandes
TimescaleDB  → métricas
pgvector     → embeddings
```

> ⚠️ **Cuidado com complexidade**: cada banco a mais é mais ops, mais expertise, mais ponto de falha. Comece simples, adicione conforme necessário.

---

## 📊 Comparação resumida

| Banco | Tipo | Latência | Escala | Casos |
|---|---|---|---|---|
| **PostgreSQL** | Relacional | Baixa | Vertical+ | Universal |
| **MongoDB** | Documento | Baixa | Horizontal | JSONs, semi-estruturado |
| **Redis** | Key-Value | <1ms | Horizontal | Cache, sessões |
| **Cassandra** | Wide-column | Média | Linear | Big writes, time-series |
| **DynamoDB** | KV/Doc | Baixa | Auto | AWS, serverless |
| **Neo4j** | Grafo | Variável | Vertical+ | Relacionamentos |
| **TimescaleDB** | Time-series | Baixa | Horizontal | Métricas, IoT |
| **Elasticsearch** | Search | Média | Horizontal | Busca, logs |
| **pgvector** | Vetorial | Baixa | Vertical+ | Embeddings, RAG |

---

## 🚨 Anti-Patterns

### 1. "Vou usar MongoDB porque está na moda"

90% dos casos onde MongoDB foi usado, **PostgreSQL** funcionaria igual ou melhor. Avalie de verdade.

### 2. SQL em DB NoSQL

Tentar fazer JOIN em MongoDB com `$lookup` ou em DynamoDB com queries complexas. Se você precisa MUITO disso, escolheu errado.

### 3. Não modelar pelo padrão de query

Em SQL você modela e depois pensa nas queries. Em Cassandra/DynamoDB é o oposto: **modela PELAS queries**. Quem ignora, sofre.

### 4. Polyglot prematuro

Time pequeno + 5 bancos diferentes = pesadelo de ops. **Use 1 ou 2** até precisar de mais.

### 5. NoSQL pra "evitar schema"

Schema **existe** mesmo em NoSQL — só está implícito. Dados sem schema = caos. Tools como **JSON Schema validation no MongoDB** ajudam.

---

## ✅ Resumo da página

- "NoSQL" não é um banco — são **categorias diferentes** pra problemas diferentes
- **Documento (MongoDB)**: dados semi-estruturados, JSONs
- **Key-Value (Redis, DynamoDB)**: lookups rápidos, cache
- **Wide-column (Cassandra)**: writes massivos, time-series
- **Grafo (Neo4j)**: relacionamentos complexos
- **Time-series (TimescaleDB, InfluxDB)**: métricas com timestamp
- **Search (Elasticsearch)**: full-text, relevance, logs
- **Vetorial (pgvector, Pinecone)**: embeddings pra IA
- **Postgres** ainda é a melhor escolha **default** — só vá pra NoSQL com motivo claro
- **Polyglot persistence** é poderoso mas adiciona complexidade
- Em DBs distribuídos, **modele pelas queries**, não pelo "modelo perfeito"

---

⬅️ [Anterior: Streaming de Dados](./45-streaming-dados.md) | 🔙 [Voltar ao índice](./README.md)
