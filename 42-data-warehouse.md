# 🌊 Modelagem Dimensional e Data Warehouses

> Banco transacional (OLTP) é otimizado pra **operações do dia a dia**: cadastros, pedidos, atualizações. Mas e quando você precisa **analisar 5 anos de dados** pra responder "qual foi a venda média por região no Q3?" — banco transacional sofre. Aí entra o **Data Warehouse** e **modelagem dimensional**.

---

## 🤔 OLTP vs OLAP

A primeira distinção que precisa ficar clara:

| Aspecto | OLTP (Online Transaction Processing) | OLAP (Online Analytical Processing) |
|---|---|---|
| **Pra quê** | Operações do dia a dia | Análise e relatórios |
| **Operações típicas** | INSERT, UPDATE, DELETE | SELECT com agregações |
| **Volume por operação** | Pequeno (1 pedido) | Gigante (milhões de pedidos) |
| **Latência aceitável** | Milissegundos | Segundos a minutos |
| **Modelagem** | Normalizada (3FN) | Desnormalizada (estrela) |
| **Exemplos** | PostgreSQL, MySQL | Snowflake, BigQuery, Redshift |

> 💡 **Em palavras simples**: OLTP é o caixa do supermercado registrando compras. OLAP é o gerente analisando relatórios de vendas mensais. Ferramentas diferentes pra problemas diferentes.

---

## 🏛️ Por Que Não Usar o Banco Transacional?

Imagine 10 milhões de pedidos no PostgreSQL de produção:

```sql
-- Query OLAP rodando em produção:
SELECT regiao, AVG(valor), COUNT(*)
FROM pedidos p
JOIN clientes c ON p.cliente_id = c.id
WHERE p.data >= '2024-01-01'
GROUP BY regiao;
```

**Problemas:**
- 🐌 **Lenta** (escaneia tabela inteira)
- 🔒 **Trava** writes durante o cálculo
- 😡 **Afeta usuários** que estão tentando comprar
- 💾 **Não foi otimizado** pra agregações massivas

A solução: **separar** dados analíticos em outro lugar.

---

## 🏗️ Arquitetura Clássica de Data Warehouse

```
   Fontes operacionais            ETL/ELT              Data Warehouse           BI/Analytics
   ┌─────────────────┐         ┌──────────┐          ┌───────────────┐       ┌─────────────┐
   │ PostgreSQL      │ ──────→ │          │          │  Star Schema  │       │  Tableau    │
   │ MySQL           │ ──────→ │  ETL     │ ────→    │  Fatos +      │ ────→ │  Power BI   │
   │ APIs externas   │ ──────→ │  Tools   │          │  Dimensões    │       │  Metabase   │
   │ Logs, CSVs      │ ──────→ │          │          │               │       │  SQL queries│
   └─────────────────┘         └──────────┘          └───────────────┘       └─────────────┘
```

### Componentes

1. **Fontes operacionais**: bancos OLTP, APIs, logs
2. **ETL/ELT**: extrai, transforma, carrega ([tópico 44](./44-etl-pipelines.md))
3. **Data Warehouse**: armazenamento otimizado
4. **BI/Analytics**: ferramentas pra explorar dados

---

## ⭐ Modelagem Dimensional: Star Schema

A forma mais clássica de modelar um DW. Inventada por **Ralph Kimball**.

### Conceito central

Você divide os dados em:
- 📊 **Fatos**: o que aconteceu (vendas, eventos, transações)
- 📐 **Dimensões**: contexto do fato (cliente, produto, tempo, região)

```
                   ┌─────────────┐
                   │ DIM_TEMPO   │
                   │  data       │
                   │  mes        │
                   │  trimestre  │
                   └──────┬──────┘
                          │
   ┌─────────────┐       │       ┌─────────────┐
   │ DIM_CLIENTE │       │       │ DIM_PRODUTO │
   │  cliente_id │       │       │  produto_id │
   │  nome       │───────┼───────│  categoria  │
   │  cidade     │       │       │  preco      │
   │  segmento   │       │       └─────────────┘
   └─────────────┘       │
                  ┌──────▼─────────┐
                  │  FATO_VENDAS   │
                  │                │
                  │  cliente_id FK │
                  │  produto_id FK │
                  │  tempo_id   FK │
                  │  loja_id    FK │
                  │  quantidade    │  ← métricas
                  │  valor_total   │
                  │  desconto      │
                  └──────┬─────────┘
                          │
                   ┌──────▼──────┐
                   │ DIM_LOJA    │
                   │  loja_id    │
                   │  cidade     │
                   │  estado     │
                   └─────────────┘
```

> 💡 **"Star schema"** porque parece uma estrela: tabela de fatos no centro, dimensões em volta.

### Tabela de Fatos

Contém:
- 🔑 **Chaves estrangeiras** pras dimensões
- 📊 **Métricas** numéricas (quantidade, valor, total)

```sql
CREATE TABLE fato_vendas (
    venda_id        BIGSERIAL PRIMARY KEY,
    -- chaves dimensionais
    cliente_id      INT REFERENCES dim_cliente,
    produto_id      INT REFERENCES dim_produto,
    tempo_id        INT REFERENCES dim_tempo,
    loja_id         INT REFERENCES dim_loja,
    -- métricas (sempre numéricas e agregáveis)
    quantidade      INT,
    valor_unitario  DECIMAL(10,2),
    valor_total     DECIMAL(10,2),
    desconto        DECIMAL(10,2)
);
```

### Tabelas de Dimensão

Contêm:
- 🆔 Chave própria (geralmente surrogate key)
- 📝 Atributos descritivos

```sql
CREATE TABLE dim_cliente (
    cliente_id      SERIAL PRIMARY KEY,
    cliente_natural_key VARCHAR,  -- ID original do sistema
    nome            VARCHAR,
    cidade          VARCHAR,
    estado          VARCHAR,
    segmento        VARCHAR,        -- B2B, B2C, etc
    data_cadastro   DATE
);
```

### Por que dimensão de tempo é tabela?

Você poderia ter `data_venda DATE` direto na fato. Mas com **dim_tempo** ganha:

```sql
CREATE TABLE dim_tempo (
    tempo_id        INT PRIMARY KEY,
    data            DATE,
    dia_semana      VARCHAR,
    mes             INT,
    nome_mes        VARCHAR,
    trimestre       INT,
    ano             INT,
    eh_feriado      BOOLEAN,
    eh_fim_semana   BOOLEAN
);
```

Queries ficam **muito mais simples**:

```sql
-- Sem dim_tempo:
SELECT EXTRACT(QUARTER FROM data_venda), SUM(valor)
FROM fato_vendas WHERE EXTRACT(YEAR FROM data_venda) = 2024
GROUP BY 1;

-- Com dim_tempo:
SELECT t.trimestre, SUM(f.valor_total)
FROM fato_vendas f
JOIN dim_tempo t ON f.tempo_id = t.tempo_id
WHERE t.ano = 2024
GROUP BY t.trimestre;
```

---

## ❄️ Snowflake Schema

Variação do star: dimensões podem ser **normalizadas** em sub-dimensões.

```
DIM_PRODUTO ──→ DIM_CATEGORIA ──→ DIM_DEPARTAMENTO
```

✅ Menos redundância
❌ Mais joins (queries mais lentas)

> 💡 **Use star como padrão**. Snowflake só quando dimensão é muito grande e complexa.

---

## 🐌 SCD: Slowly Changing Dimensions

E quando uma dimensão **muda**? Cliente mudou de cidade. Como tratar?

### Tipo 1: Sobrescreve

Atualiza o valor, perde histórico.

```sql
UPDATE dim_cliente SET cidade = 'Rio' WHERE cliente_id = 123;
```

✅ Simples
❌ "Quantos clientes de SP em 2020?" → resposta errada se ele mudou pro Rio

### Tipo 2: Versiona

Adiciona nova linha. Mantém histórico.

```sql
-- Linha original
| 100 | Mauricio | São Paulo | 2020-01-01 | 2024-03-15 | NÃO   |
-- Nova linha após mudança
| 101 | Mauricio | Rio       | 2024-03-15 | NULL       | SIM   |
```

```sql
-- Schema
CREATE TABLE dim_cliente (
    cliente_id          SERIAL PRIMARY KEY,
    cliente_natural_key VARCHAR,
    nome                VARCHAR,
    cidade              VARCHAR,
    valido_desde        TIMESTAMP,
    valido_ate          TIMESTAMP,
    eh_atual            BOOLEAN
);
```

✅ Histórico preservado
❌ Mais complexo

### Tipo 3: Atributo de mudança

Coluna extra "anterior":

```sql
| 100 | Mauricio | Rio | São Paulo (anterior) |
```

✅ Simples mas mantém última mudança
❌ Só guarda 1 mudança

> 💡 **Tipo 2 é o mais usado** quando histórico importa (na maioria dos DWs sérios).

---

## 🛠️ Ferramentas de Data Warehouse

### Cloud (mais usadas em 2026)

| DW | Provider | Pontos fortes |
|---|---|---|
| **Snowflake** | Multi-cloud | Performance, zero-management, marketplace |
| **BigQuery** | Google | Serverless, ML integrado, preço por query |
| **Redshift** | AWS | Integração AWS, RA3 nodes, Spectrum |
| **Synapse** | Azure | Integração Microsoft, dedicated/serverless |
| **Databricks SQL** | Multi-cloud | Lakehouse + DW unificado |

### Open Source / On-premise

- **ClickHouse** (column-oriented, super rápido)
- **Apache Druid** (analytics em tempo real)
- **Apache Pinot** (similar ao Druid)
- **DuckDB** ("SQLite analítico", rodando local)

### Características em comum dos DWs cloud

- 📦 **Storage e compute separados**: escala independente
- 🔥 **Columnar storage**: dados agrupados por coluna, não linha
- ⚡ **Query parallelization**: divide queries em múltiplos workers
- 💾 **Compressão pesada**: dados analíticos comprimem muito bem

---

## 📊 Columnar vs Row Storage

Diferença **crucial** entre OLTP e OLAP.

### Row-oriented (OLTP)

```
[id=1, nome=Ana, cidade=SP, valor=100]
[id=2, nome=Bob, cidade=RJ, valor=200]
[id=3, nome=Cid, cidade=SP, valor=150]
```

✅ Pegar 1 linha completa é rápido (`SELECT * WHERE id=2`)
❌ Pegar `SUM(valor)` precisa ler tudo

### Column-oriented (OLAP)

```
id:     [1, 2, 3]
nome:   [Ana, Bob, Cid]
cidade: [SP, RJ, SP]
valor:  [100, 200, 150]
```

✅ `SUM(valor)` lê só essa coluna (super rápido)
✅ Compressão alta (valores parecidos juntos)
❌ Pegar 1 linha completa exige montar de várias colunas

> 💡 **Por isso DWs são columnar**: você não busca pedidos individuais, busca métricas agregadas em milhões deles.

---

## 🧮 Star Schema na Prática

### Exemplo: e-commerce

```sql
-- Dimensões
CREATE TABLE dim_cliente (...);
CREATE TABLE dim_produto (...);
CREATE TABLE dim_tempo (...);
CREATE TABLE dim_loja (...);

-- Fato
CREATE TABLE fato_vendas (
    venda_id     BIGSERIAL,
    cliente_id   INT,
    produto_id   INT,
    tempo_id     INT,
    loja_id      INT,
    quantidade   INT,
    valor_total  DECIMAL(12,2)
);

-- Query típica: vendas por trimestre, região, categoria
SELECT
    t.ano,
    t.trimestre,
    l.estado,
    p.categoria,
    SUM(f.valor_total) AS receita,
    COUNT(*) AS num_vendas
FROM fato_vendas f
JOIN dim_tempo t  ON f.tempo_id = t.tempo_id
JOIN dim_loja l   ON f.loja_id = l.loja_id
JOIN dim_produto p ON f.produto_id = p.produto_id
WHERE t.ano = 2024
GROUP BY t.ano, t.trimestre, l.estado, p.categoria
ORDER BY receita DESC;
```

DWs cloud (Snowflake, BigQuery) executam isso em segundos mesmo com **bilhões** de linhas.

---

## 🎯 Conceitos Avançados

### Fatos sem fato (factless facts)

Tabelas de fato sem métricas, registrando **eventos**:

```sql
-- "Quem viu qual produto, quando"
CREATE TABLE fato_visualizacao (
    cliente_id INT,
    produto_id INT,
    tempo_id   INT
    -- sem métricas!
);
```

Útil pra contar ocorrências.

### Granularidade

Cada linha do fato representa **o que**?
- 1 venda = 1 linha (granular)
- 1 dia de vendas agregado = 1 linha (sumarizado)

> 💡 **Sempre prefira granularidade fina**. Você pode agregar depois, mas não pode "desagregar" o que já agregou.

### Materialized Views

Resultados de queries pré-calculadas:

```sql
CREATE MATERIALIZED VIEW vendas_diarias AS
SELECT
    t.data,
    SUM(f.valor_total) AS total
FROM fato_vendas f
JOIN dim_tempo t ON f.tempo_id = t.tempo_id
GROUP BY t.data;

-- Refresh periódico
REFRESH MATERIALIZED VIEW vendas_diarias;
```

✅ Queries instantâneas
❌ Dados podem estar atrasados (depende da frequência de refresh)

### Particionamento

Divide tabela em "pedaços" físicos por critério (geralmente data):

```sql
CREATE TABLE fato_vendas (
    ...
) PARTITION BY RANGE (tempo_id);

CREATE TABLE fato_vendas_2024 PARTITION OF fato_vendas
    FOR VALUES FROM (20240101) TO (20250101);
```

Queries com filtro de tempo só leem partições relevantes.

---

## 💡 Dicas Práticas

### 1. Comece pelo negócio, não pelo schema

Pergunte:
- Quais perguntas o time de negócios precisa responder?
- Quais métricas importam?
- Em qual granularidade?

A modelagem **sai naturalmente** das perguntas.

### 2. Surrogate keys > natural keys

```sql
-- ❌ Usar CPF/CNPJ como PK
cliente_id VARCHAR(14)

-- ✅ Usar ID gerado
cliente_id BIGSERIAL PRIMARY KEY,
cpf VARCHAR(14)  -- atributo, não chave
```

Por quê? CPF pode mudar (raro mas acontece), tem letras/símbolos, joins ficam pesados em DW.

### 3. Documente!

Cada métrica precisa ter definição clara:
```
receita_liquida = valor_total - desconto - imposto
```

Sem isso, equipes diferentes calculam "receita" de formas diferentes. **Pesadelo**.

### 4. Versionamento de schema

DWs evoluem. Use ferramentas como **dbt** ([visto no próximo tópico](./44-etl-pipelines.md)) pra versionar transformações.

### 5. Monitore custo

DWs cloud cobram por **query** (BigQuery) ou **compute time** (Snowflake). Queries ruins viram contas gigantes.

```
❌ SELECT * FROM fato_vendas (escaneia 10TB) → $50 numa query
✅ SELECT col1, col2 WHERE data > '2024-01-01' (1GB) → $0.005
```

---

## 🆚 Data Warehouse vs Data Lake vs Lakehouse

Spoiler do próximo tópico:

| Característica | Data Warehouse | Data Lake | Lakehouse |
|---|---|---|---|
| **Schema** | Estruturado, predefinido | Schema-on-read | Híbrido |
| **Dados** | Estruturados | Tudo (JSON, imagens, vídeos) | Tudo + estruturados |
| **Custo storage** | Caro | Barato | Barato |
| **Performance query** | 🔥 | OK | 🔥 |
| **Casos** | BI, relatórios | ML, dados brutos | Tudo unificado |

> 💡 **Lakehouse** (Databricks, Snowflake) é a tendência: tenta ter o melhor dos dois.

---

## ✅ Resumo da página

- **OLTP** (transacional) e **OLAP** (analítico) são otimizados pra coisas diferentes
- **Data Warehouse** = banco otimizado pra análise (colunar, agregações)
- **Modelagem dimensional**: tabelas de **fatos** (métricas) + **dimensões** (contexto)
- **Star Schema**: padrão. **Snowflake** é variação normalizada
- **SCD**: como lidar com mudanças nas dimensões (Tipo 2 mais comum)
- DWs cloud principais: **Snowflake, BigQuery, Redshift, Synapse**
- **Columnar storage** é o que faz DW ser rápido em agregações
- **Materialized views, particionamento** otimizam ainda mais
- Use **surrogate keys**, documente métricas, monitore custos
- **Lakehouse** está unificando DW + Data Lake

---

⬅️ [Anterior: Java Completo](./41-java-completo.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Data Lakes e Lakehouses](./43-data-lakes.md)
