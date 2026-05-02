# 🏞️ Data Lakes e Lakehouses

> Data Warehouse é ótimo, mas tem limitações: caro, schema rígido, ruim pra dados não-estruturados (JSON, imagens, áudio). **Data Lakes** resolveram isso. Mas trouxeram outros problemas. Aí surgiu o **Lakehouse** — tentando ter o melhor dos dois mundos.

---

## 🌊 A Evolução

### Era 1: Só Data Warehouse (1990s-2010s)

```
Fontes → ETL → Data Warehouse → BI
```

✅ Estruturado, performance ótima
❌ Caro
❌ Schema rígido (precisa decidir tudo antes)
❌ Não aguenta dados não-estruturados (JSONs, imagens, vídeos)

### Era 2: Data Lake (2010s)

```
Fontes → Storage barato (S3, GCS) → Spark/Hadoop → Insights
```

✅ Barato (S3 cobra centavos por GB)
✅ Aceita qualquer formato
✅ Schema-on-read (decide depois)
❌ Vira **data swamp** (pântano de dados sem organização)
❌ Performance ruim sem indexação
❌ Difícil garantir qualidade

### Era 3: Lakehouse (2020s)

```
Fontes → Storage barato + Metadados/Transações + Engine analítica
```

✅ Barato como lake
✅ Performance próxima de DW
✅ ACID em cima de arquivos
✅ Suporta dados estruturados E não-estruturados

> 💡 **Em palavras simples**: Lakehouse = Data Lake com superpoderes de Data Warehouse.

---

## 🏞️ Data Lake: Conceito

Data Lake é, no fundo, **um bucket de object storage** com arquivos.

```
s3://meu-data-lake/
├── raw/                    ← dados brutos como vieram
│   ├── eventos/
│   │   ├── 2024-01-01.json
│   │   └── 2024-01-02.json
│   ├── pedidos/
│   └── usuarios/
├── processed/              ← dados limpos
│   ├── eventos/
│   └── pedidos/
└── curated/                ← dados prontos pra análise
    └── vendas_consolidadas/
```

### As 3 zonas (data lake bem organizado)

#### 1. Raw / Bronze
Dados **como vieram**, sem transformação. Preserva fonte original.

```
raw/api_pedidos/2024-01-15/abc123.json
{
  "id": 12345,
  "items": [...],
  "createdAt": "2024-01-15T10:30:00Z"
}
```

#### 2. Processed / Silver
Dados **limpos e estruturados**. Tipos corrigidos, duplicatas removidas.

```parquet
processed/pedidos/year=2024/month=01/day=15/
  → arquivos Parquet
```

#### 3. Curated / Gold
Dados **prontos pra consumo**. Agregações, joins, modelagem dimensional.

```
curated/vendas_diarias/year=2024/
  → tabelas otimizadas pra análise
```

> 💡 **Padrão "Medallion"** (Bronze → Silver → Gold) é da Databricks, virou padrão da indústria.

---

## 💾 Formatos de Arquivo

A escolha do formato é **crítica** pra performance.

### CSV / JSON

```csv
id,nome,valor
1,Ana,100
2,Bob,200
```

✅ Universal, legível
❌ Verboso, sem tipos, sem compressão eficiente
❌ Performance péssima em DW (sem schema, sem column-pruning)

### Parquet ⭐

Formato **colunar binário**. Padrão de fato em Big Data.

```
Estrutura:
- Metadados (schema)
- Coluna 1: [valores comprimidos]
- Coluna 2: [valores comprimidos]
- ...
```

✅ **Compressão** alta (10-100x menor que CSV)
✅ **Column pruning**: lê só as colunas que precisa
✅ **Predicate pushdown**: filtra durante leitura
✅ **Schema embutido**

```python
# Salvando com pandas
df.to_parquet('vendas.parquet', compression='snappy')

# Lendo
df = pd.read_parquet('vendas.parquet')
```

### ORC

Similar ao Parquet, criado pelo Hortonworks.

- ✅ Levemente melhor pra Hive
- ❌ Menos suporte fora do ecossistema Hadoop

### Avro

Formato **row-oriented** (linha, não coluna).

```
Schema (separado) + dados binários
```

✅ Bom pra **streaming** (Kafka usa)
✅ Schema evolution forte
❌ Não bom pra analytics (linha, não coluna)

### Comparação

| Formato | Tipo | Compressão | Bom pra | Quando usar |
|---|---|---|---|---|
| **CSV** | Linha | Ruim | Compatibilidade | Export pro Excel |
| **JSON** | Linha | Ruim | Hierarquia | APIs, debug |
| **Avro** | Linha | OK | Streaming, Kafka | Pipelines de eventos |
| **Parquet** | Coluna | Ótima | Analytics | DW, lakes ⭐ |
| **ORC** | Coluna | Ótima | Hive | Ecossistema Hadoop |

> 💡 **Regra de ouro**: dados em movimento → Avro. Dados em descanso pra analytics → Parquet.

---

## 🏘️ Lakehouse: A Evolução

Data Lake puro tem problemas:
- 🌪️ **Sem ACID**: dois jobs gravando no mesmo arquivo = caos
- 🔄 **Sem updates eficientes**: mudar 1 linha em 1TB de Parquet = horror
- 📜 **Sem time travel**: como ver como o dado estava ontem?
- 🗂️ **Sem schema enforcement**: schema pode mudar e quebrar leitores

**Lakehouse** adiciona uma **camada transacional** sobre os arquivos.

### As 3 grandes implementações

#### Delta Lake (Databricks)

```python
# Escrita transacional
df.write.format("delta").save("s3://lake/vendas")

# Update (impossível em Parquet puro)
deltaTable.update(
    condition = "id = 123",
    set = {"valor": "200"}
)

# Time travel
spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-15") \
    .load("s3://lake/vendas")
```

✅ Mais maduro
✅ Backed by Databricks
❌ Originalmente acoplado a Spark

#### Apache Iceberg (Netflix → Apache)

```sql
-- Iceberg via SQL
CREATE TABLE vendas (
    id BIGINT,
    valor DECIMAL(10,2)
) USING iceberg;

UPDATE vendas SET valor = 200 WHERE id = 123;

-- Time travel
SELECT * FROM vendas TIMESTAMP AS OF '2024-01-15';
```

✅ Mais "padrão aberto"
✅ Suporta múltiplas engines (Spark, Trino, Flink)
✅ Hidden partitioning (não precisa especificar coluna de partição em queries)

#### Apache Hudi (Uber → Apache)

✅ Bom pra **upserts** em larga escala
✅ Streaming nativo
❌ Menos popular que Delta/Iceberg

### Comparação

| Feature | Delta | Iceberg | Hudi |
|---|---|---|---|
| **ACID** | ✅ | ✅ | ✅ |
| **Time travel** | ✅ | ✅ | ✅ |
| **Schema evolution** | ✅ | ✅ | ✅ |
| **Engine principal** | Spark | Multi | Spark |
| **Padrão aberto** | OK | ⭐ | OK |
| **Comunidade** | Grande | Crescendo | Menor |

> 💡 **Em 2026**: Iceberg está ganhando força como padrão aberto. Snowflake, BigQuery, Databricks suportam. Mas Delta ainda é dominante onde Databricks é usado.

---

## 🏗️ Arquitetura Lakehouse Moderna

```
   Storage (barato, escalável)
   ┌──────────────────────────────────┐
   │  S3 / GCS / Azure Blob           │
   │  (arquivos Parquet)              │
   └──────────────────────────────────┘
                    ↑
   Camada transacional (metadados, ACID)
   ┌──────────────────────────────────┐
   │  Delta / Iceberg / Hudi          │
   └──────────────────────────────────┘
                    ↑
   Engines de query
   ┌──────────────────────────────────┐
   │  Spark / Trino / Flink / Athena  │
   └──────────────────────────────────┘
                    ↑
   Consumidores
   ┌──────────────────────────────────┐
   │  BI tools / Notebooks / Apps     │
   └──────────────────────────────────┘
```

---

## 🎯 Particionamento

Divide dados por colunas (geralmente data) pra acelerar queries.

```
s3://lake/vendas/
├── year=2024/
│   ├── month=01/
│   │   ├── day=01/
│   │   │   ├── parte_001.parquet
│   │   │   └── parte_002.parquet
│   │   └── day=02/
│   └── month=02/
└── year=2025/
```

Query com filtro `WHERE year=2024 AND month=01` lê **só esse diretório**.

### Boas práticas

✅ Particione por colunas frequentemente filtradas (geralmente **data**)
✅ Tamanho razoável por partição (100MB - 1GB cada)
❌ NÃO particione por colunas de alta cardinalidade (millions de partições = pesadelo)
❌ NÃO super-particione (year/month/day/hour gera muitos arquivos pequenos)

---

## 🔥 Pequeno problema: Small Files

Lakes ingerindo dados em streaming criam **milhares de arquivos pequenos** por dia.

```
year=2024/month=01/day=15/
├── 00:00:01.parquet (50KB)
├── 00:00:05.parquet (30KB)
├── 00:00:12.parquet (45KB)
... (milhares deles)
```

Engines de query gastam mais tempo abrindo arquivos do que lendo dados.

### Soluções

#### Compactação periódica
Job que **junta arquivos pequenos** em maiores:

```python
# Delta Lake
deltaTable.optimize().executeCompaction()

# Iceberg
spark.sql("CALL system.rewrite_data_files('vendas')")
```

#### Z-ordering / clustering
Reorganiza dados fisicamente pra co-localizar registros similares (Delta/Iceberg suportam).

---

## 🎨 Casos de Uso

### Lake é melhor pra

- 🤖 **ML training**: precisa dos dados brutos completos
- 📊 **Data science**: exploração com formato livre
- 📦 **Arquivamento**: dados raros mas precisam ser guardados
- 🌊 **Streaming**: armazenar eventos contínuos
- 💰 **Custo**: PB de dados ficam baratos no S3

### Warehouse é melhor pra

- 📈 **BI tradicional**: dashboards, relatórios padronizados
- 🔥 **Performance crítica**: respostas em ms
- 👨‍💼 **Usuários SQL**: analistas que só sabem SQL
- 🛡️ **Governança forte**: catálogo, permissões, auditoria

### Lakehouse: tenta ambos

- 📊 BI + ML no mesmo lugar
- 💰 Custo de lake, performance de DW
- 🔄 ACID em cima de arquivos

---

## 🌐 Plataformas Lakehouse

### Databricks

Inventou o termo "Lakehouse". Plataforma completa:
- Delta Lake (storage)
- Spark (compute)
- SQL Analytics (BI)
- ML, AI tools

### Snowflake

Tradicional DW, agora suporta tabelas externas (Iceberg) e formatos abertos.

### Amazon Athena + S3 + Glue

Stack lakehouse "DIY" na AWS:
- S3 (storage)
- Athena (query engine sem servidor)
- Glue (catálogo + ETL)
- Iceberg (formato)

### Google BigQuery + BigLake

Similar pra GCP.

### Trino / Starburst

Query engine open source pra lakehouse.

---

## 📋 Catálogos de Dados

Em lakehouse com bilhões de arquivos, **descoberta** vira problema. Catálogos resolvem.

### Apache Hive Metastore

Padrão clássico. Guarda metadata (schema, partições, localização).

### AWS Glue

Metastore gerenciado da AWS. Auto-descobre schemas com **crawlers**.

### Unity Catalog (Databricks)

Catálogo unificado com governança, permissões, lineage.

### Apache Polaris (Iceberg)

Catálogo aberto pra Iceberg, recém-lançado.

> 💡 **Sem catálogo, lakehouse vira swamp**. Comece com catálogo desde o dia 1.

---

## 🚨 Anti-Patterns

### 1. "Lake como SaaS de logs"

```
Time joga TUDO no S3 sem padrão → ninguém consegue achar nada
```

✅ **Adote zonas (Bronze/Silver/Gold)** desde o início.

### 2. Sem catálogo

Sem metastore, queries precisam saber path exato. Inviável em escala.

### 3. Particionamento errado

```
WHERE user_id = 123    ← particionou por isso → 1 milhão de partições, caos
```

Particione por **data**, não por chaves de alta cardinalidade.

### 4. Não compactar arquivos pequenos

Streaming joga 1 arquivo por minuto → após 1 ano você tem 500k arquivos por tabela. Performance afundada.

### 5. Querer tudo "tempo real"

Streaming é caro e complexo. Maioria dos relatórios funciona com **micro-batches** (a cada 5 min, 1h, 1 dia).

---

## 🛠️ Ferramentas Úteis

### Engines de query

- **Apache Spark** (mais usado)
- **Trino / Presto** (SQL distribuído)
- **Apache Flink** (streaming + batch)
- **DuckDB** (local, super rápido)
- **Polars** (Python, rápido)

### Notebooks

- **Databricks Notebooks**
- **Jupyter / JupyterHub**
- **Hex, Deepnote, Mode**

### Frameworks

- **PySpark** (Python pra Spark)
- **Pandas** (datasets pequenos)
- **Polars** (alternativa moderna ao Pandas)
- **Dask** (Pandas distribuído)

---

## ✅ Resumo da página

- **Data Lake** = bucket de object storage com arquivos brutos. Barato, flexível.
- Sem organização vira **data swamp** — use zonas (Bronze/Silver/Gold)
- **Parquet** é o formato padrão (colunar, comprimido, performante)
- **Lakehouse** = lake + camada transacional (ACID, time travel, updates)
- 3 implementações principais: **Delta Lake, Apache Iceberg, Apache Hudi**
- **Iceberg** está virando padrão aberto da indústria
- **Particione por data**, evite small files
- Use **catálogos** (Hive Metastore, Glue, Unity) desde o início
- **Lakehouse > Lake puro** pra maioria dos casos modernos
- Plataformas: **Databricks**, **Snowflake**, **AWS Athena+Glue**, **BigQuery**

---

⬅️ [Anterior: Modelagem Dimensional e DW](./42-data-warehouse.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: ETL/ELT e Pipelines](./44-etl-pipelines.md)
