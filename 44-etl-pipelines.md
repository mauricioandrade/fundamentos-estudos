# 🔄 ETL/ELT e Pipelines de Dados

> Você tem dados em **vários lugares**: banco transacional, APIs, planilhas, logs. Pra analisar, precisa **mover, transformar e carregar** num lugar centralizado (DW ou lake). Esse processo é **ETL/ELT** — e fazer bem feito é o que separa pipelines confiáveis de pesadelos noturnos.

---

## 🎯 ETL vs ELT

### ETL (Extract, Transform, Load)

**Padrão clássico**, dominante até ~2015.

```
Fontes → Extract → Transform (em servidor dedicado) → Load → DW
```

```
PostgreSQL ─┐
MySQL ──────┼─→ [Servidor ETL: Informatica] ─→ Data Warehouse
APIs ───────┘     (transforma na memória)
```

**Características:**
- 🔧 Transformação **antes** de carregar
- 💪 Servidor ETL faz o trabalho pesado
- 🎯 Carrega só dados **prontos** no DW

### ELT (Extract, Load, Transform)

**Padrão moderno**, viabilizado por DWs cloud.

```
Fontes → Extract → Load (raw) → Transform (no DW) → DW limpo
```

```
PostgreSQL ─┐
MySQL ──────┼─→ Snowflake (raw) → SQL (transforma) → Snowflake (final)
APIs ───────┘
```

**Características:**
- 📦 Carrega **bruto** primeiro
- 🔥 Transformação no **próprio DW** (compute potente)
- 🔄 Mais flexível: pode retransformar com lógica nova

### Comparação

| Aspecto | ETL | ELT |
|---|---|---|
| **Transformação** | Antes do load | Depois do load |
| **Compute** | Servidor dedicado | DW (Snowflake, BigQuery) |
| **Custo storage** | Menor (só dados finais) | Maior (raw + finais) |
| **Flexibilidade** | Menor | Maior (pode refazer transforms) |
| **Boa pra** | Sistemas legados | Cloud-native moderno |
| **Ferramentas** | Informatica, Talend, SSIS | dbt, Airflow + DW |

> 💡 **Em 2026**: ELT venceu pra projetos novos. Storage barato + DWs poderosos tornaram inviável o overhead de ETL clássico. Mas ETL ainda existe em muitos sistemas legados.

---

## 🧩 Anatomia de um Pipeline

Pipelines de dados típicos têm 4 partes:

### 1. Ingestão (Extract)

Tira dados das fontes. Modos:

#### Full load
Extrai **tudo** toda vez.

```sql
-- Tira tabela inteira
SELECT * FROM pedidos;
```

✅ Simples
❌ Lento, caro com tabelas grandes

#### Incremental
Só pega o que **mudou**.

```sql
-- Só novos/atualizados desde último run
SELECT * FROM pedidos WHERE updated_at > '2024-01-15 10:00:00';
```

✅ Eficiente
❌ Precisa de coluna confiável de modificação
❌ Não pega DELETEs

#### Change Data Capture (CDC)

Lê o **log de mudanças** do banco.

```
PostgreSQL WAL → Debezium → Kafka → Pipeline
MySQL binlog →     ↓
                 captura todos INSERT/UPDATE/DELETE
```

✅ Pega TUDO em tempo real, incluindo DELETEs
✅ Mínimo impacto no banco
❌ Setup mais complexo

> 💡 **Ferramentas CDC**: Debezium (open source), Fivetran, Airbyte, AWS DMS.

### 2. Transformação

Onde a mágica acontece. Tarefas comuns:

#### Limpeza
```sql
-- Remove duplicatas
SELECT DISTINCT * FROM pedidos;

-- Trata nulls
SELECT COALESCE(cidade, 'Desconhecido') FROM clientes;

-- Padroniza formatos
SELECT LOWER(TRIM(email)) FROM usuarios;
```

#### Enriquecimento
```sql
-- Junta dados de várias fontes
SELECT
    p.id,
    p.valor,
    c.nome AS cliente_nome,
    pr.categoria AS produto_categoria
FROM pedidos p
JOIN clientes c ON p.cliente_id = c.id
JOIN produtos pr ON p.produto_id = pr.id;
```

#### Agregação
```sql
-- Sumariza
SELECT data, SUM(valor), COUNT(*)
FROM pedidos
GROUP BY data;
```

#### Modelagem dimensional
Constrói as tabelas de fato e dimensão ([visto no tópico 42](./42-data-warehouse.md)).

### 3. Carga (Load)

Insere dados no destino:
- Append (adiciona novos)
- Truncate + insert (substitui tudo)
- Merge / Upsert (atualiza existentes, insere novos)

### 4. Orquestração

**Quem dispara o quê, quando, e em que ordem?**

```
Diariamente às 2h:
  1. Extrair dados do PostgreSQL
  2. Extrair dados de APIs
  3. (espera ambos) → Transformar
  4. (após sucesso) → Carregar no DW
  5. (após sucesso) → Atualizar dashboards
  6. Notificar time
```

Sem orquestrador = scripts cron caóticos.

---

## 🛠️ Apache Airflow

Orquestrador **mais popular** do mundo.

### Conceito: DAG

**Directed Acyclic Graph** — grafo direcionado sem ciclos.

```
extrair_postgres ─┐
                  ├─→ transformar ─→ carregar ─→ notificar
extrair_apis    ──┘
```

### Exemplo simples

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

def extrair():
    # tira dados do banco
    pass

def transformar():
    # processa
    pass

def carregar():
    # insere no DW
    pass

with DAG(
    'pipeline_diario',
    start_date=datetime(2024, 1, 1),
    schedule='0 2 * * *',  # todo dia às 2h (cron)
    catchup=False,
) as dag:

    t1 = PythonOperator(task_id='extrair', python_callable=extrair)
    t2 = PythonOperator(task_id='transformar', python_callable=transformar)
    t3 = PythonOperator(task_id='carregar', python_callable=carregar)

    t1 >> t2 >> t3  # define dependências
```

### Features importantes

- 🔄 **Retry automático** em falhas
- ⏰ **Schedule flexível** (cron, intervalos)
- 📊 **UI web** pra ver execuções
- 🚨 **Alertas** por email/Slack
- 🔌 **Operadores prontos** pra DBs, APIs, cloud
- 🐍 **Python nativo** (DAGs são código)

### Airflow vs alternativas modernas

| Tool | Linguagem | Pontos fortes |
|---|---|---|
| **Airflow** | Python | Mais maduro, comunidade gigante |
| **Prefect** | Python | API mais moderna, dynamic DAGs |
| **Dagster** | Python | Asset-based, type checking, melhor DX |
| **Mage** | Python | UI-first, mais simples |
| **Argo Workflows** | YAML | K8s-native |
| **AWS Step Functions** | JSON | Serverless, integração AWS |

> 💡 **Pra começar projeto novo em 2026**: considere **Dagster** ou **Prefect**. Airflow ainda é o padrão da indústria, mas tem decisões de design antigas que pesam.

---

## 🔥 dbt: Transformação Moderna

**dbt (data build tool)** revolucionou a parte "T" do ELT.

### O que é

Você escreve **SQL com Jinja** (templating), dbt cuida de:
- 📊 Compilar SQL final
- 📈 Executar na ordem certa
- 📚 Gerar documentação
- 🧪 Rodar testes
- 🔄 Versionar transformações

### Exemplo

```sql
-- models/staging/stg_pedidos.sql

WITH source AS (
    SELECT * FROM {{ source('raw', 'pedidos') }}
),

renamed AS (
    SELECT
        id AS pedido_id,
        customer_id AS cliente_id,
        total AS valor_total,
        created_at AS data_criacao
    FROM source
)

SELECT * FROM renamed
```

```sql
-- models/marts/fact_vendas.sql

SELECT
    p.pedido_id,
    p.cliente_id,
    c.cidade,
    p.valor_total,
    p.data_criacao
FROM {{ ref('stg_pedidos') }} p
LEFT JOIN {{ ref('stg_clientes') }} c
    ON p.cliente_id = c.cliente_id
```

```bash
# Rodar transformações na ordem correta
dbt run

# Testes (NULL, único, FK, custom)
dbt test

# Documentação automática
dbt docs generate
dbt docs serve  # abre site interativo
```

### Por que dbt vendeu tanto

✅ **Engenharia em SQL**: Git, testes, docs, modular
✅ **Testes built-in**: garantia de qualidade
✅ **Documentação automática** com lineage
✅ **Modularidade**: refs entre models
✅ **CI/CD friendly**: roda em pipelines

> 💡 **dbt + Airflow** é combo dominante: Airflow orquestra, dbt transforma.

---

## 🎯 Padrões e Boas Práticas

### 1. Idempotência

Pipeline que roda **2 vezes** deve ter **mesmo resultado** que 1.

```sql
-- ❌ Não idempotente: cada execução adiciona
INSERT INTO vendas SELECT * FROM staging;

-- ✅ Idempotente: substitui ou faz upsert
MERGE INTO vendas USING staging
ON vendas.id = staging.id
WHEN MATCHED THEN UPDATE
WHEN NOT MATCHED THEN INSERT;
```

> 🤔 **Por que importa?** Pipeline falhou no meio? Precisa reprocessar? Sem idempotência, vira pesadelo.

### 2. Particionamento de tempo

Processe **por janela de tempo**, não tudo de uma vez.

```python
# Em vez de "processar TUDO"
data_alvo = "{{ ds }}"  # Airflow injeta data da execução

SELECT *
FROM eventos
WHERE date(created_at) = '{{ data_alvo }}'
```

✅ Reprocessar dia específico é trivial
✅ Falhas afetam menos dados
✅ Backfill (preencher histórico) funciona

### 3. Backfill estratégico

Você criou métrica nova, precisa calcular pros últimos 2 anos.

```bash
# Airflow CLI
airflow dags backfill pipeline_vendas \
    --start-date 2022-01-01 \
    --end-date 2024-01-01
```

Pipeline particionado por data faz isso fácil.

### 4. Data Quality testing

```yaml
# dbt schema.yml
version: 2

models:
  - name: stg_pedidos
    columns:
      - name: pedido_id
        tests:
          - unique
          - not_null
      - name: cliente_id
        tests:
          - relationships:
              to: ref('stg_clientes')
              field: cliente_id
      - name: valor_total
        tests:
          - dbt_utils.expression_is_true:
              expression: ">= 0"
```

Testes rodam após transformação. Pipeline para se algo está errado.

### 5. Schema evolution

Tabela fonte ganhou coluna nova. Como pipeline reage?

**Estratégias:**
- ✅ **Schema-on-read**: aceita qualquer formato (data lake)
- ✅ **Schema enforcement com fallback**: novos campos mapeados, antigos preservados
- ❌ **Schema rígido sem alerta**: pipeline quebra silenciosamente

### 6. Logging e observabilidade

```python
import logging

def transformar():
    logging.info(f"Processando {len(df)} linhas")
    df_clean = limpar(df)
    logging.info(f"{len(df_clean)} linhas após limpeza")

    if len(df_clean) < len(df) * 0.5:
        logging.warning("Mais de 50% dos dados foram filtrados!")
```

### 7. Monitoramento de freshness

```sql
-- Quão antigo é o dado mais recente?
SELECT
    MAX(created_at) AS ultimo_dado,
    NOW() - MAX(created_at) AS atraso
FROM vendas;
```

Se atraso > esperado, **alertar**.

---

## 📡 Streaming vs Batch

### Batch (lote)

Processa em **janelas** (1h, 1 dia).

```
Coletados 24h de dados → Processa de madrugada → DW pronto às 6h
```

✅ Simples
✅ Barato
✅ Eficiente pra agregações
❌ Latência alta (resultado de hoje só amanhã)

**Quando usar**: relatórios diários/semanais, BI, ML batch training.

### Streaming

Processa **continuamente**, evento por evento.

```
Evento chega → Processado em segundos → Resultado disponível
```

✅ Latência baixa (segundos)
✅ Bom pra detecção em tempo real
❌ Mais complexo
❌ Mais caro

**Quando usar**: detecção de fraude, monitoramento, real-time analytics.

> 💡 **Mais detalhes no [tópico 45 (Streaming)](./45-streaming-dados.md)**.

### Lambda Architecture

Combina batch + streaming:

```
Eventos ─→ Batch layer  → Resultados precisos (atraso)
       └→ Speed layer  → Resultados aproximados (rápido)
       
Aplicação combina os dois
```

### Kappa Architecture

**Só streaming**, batch é caso especial.

```
Eventos → Stream processor → Tudo
```

Mais simples, mas exige stream processing maduro.

---

## 🚨 Anti-Patterns

### 1. Pipelines em scripts cron sem orquestrador

```bash
# Cron
0 2 * * * /scripts/etl.sh
```

Não tem retry, não tem dependências, não tem visibilidade. Ruim em qualquer escala.

### 2. Lógica de negócio espalhada

Transformações em vários lugares (script Python, view SQL, função do Airflow). **Centralize** com dbt ou similar.

### 3. Sem testes de qualidade

Pipeline rodou? Sucesso? **Não significa que dado está certo**. Testes de data quality são essenciais.

### 4. Não-idempotência

Pipeline reprocessa = duplica dados. Receita pra desastre.

### 5. Pipeline frágil que processa "tudo"

Falhou no dia 30? Reprocessa **30 dias inteiros**. Particione!

### 6. Sem documentação

Equipe nova entra. Pipelines rodam. Ninguém sabe o que cada um faz.

```sql
-- ✅ dbt models documentados
-- models/marts/fact_vendas.yml
version: 2

models:
  - name: fact_vendas
    description: |
      Fato consolidado de vendas, granularidade pedido.
      Receita líquida = valor_total - desconto - imposto.
      Atualizado diariamente às 2h.
    columns:
      - name: receita_liquida
        description: "Valor após descontos e impostos"
```

---

## 📊 Tipos de Pipelines Modernos

### EL puro
Só extrair e carregar. Sem transformação.

```
PostgreSQL → Snowflake (raw)
```

Útil quando time de analytics faz transformação separadamente.

**Ferramentas**: Fivetran, Airbyte, Stitch.

### Reverse ETL

Pega do DW e **manda de volta** pra ferramentas operacionais.

```
DW (segmentos calculados) → HubSpot, Salesforce, Mailchimp
```

> 💡 Cresceu muito porque DW virou "single source of truth", mas times de marketing/sales precisam dos dados em ferramentas deles.

**Ferramentas**: Hightouch, Census, RudderStack.

### Reverse ETL + ELT

```
Fontes → DW (raw) → DW (limpo) → Reverse ETL → Ferramentas
```

Stack moderna completa.

---

## 🎯 Stack Moderno Típico

```
INGESTÃO:        Fivetran / Airbyte / Debezium (CDC)
                          ↓
STORAGE:         Snowflake / BigQuery / Databricks
                          ↓
TRANSFORMAÇÃO:   dbt
                          ↓
ORQUESTRAÇÃO:    Airflow / Dagster / Prefect
                          ↓
QUALIDADE:       dbt tests / Great Expectations / Monte Carlo
                          ↓
BI:              Tableau / Looker / Metabase / Mode
                          ↓
REVERSE ETL:     Hightouch / Census
```

---

## ✅ Resumo da página

- **ETL** (Extract, Transform, Load) é clássico; **ELT** (Extract, Load, Transform) venceu em cloud
- Ingestão: **full load**, **incremental**, **CDC** (Change Data Capture)
- **Airflow** é o orquestrador mais usado; **Dagster** e **Prefect** são alternativas modernas
- **dbt** revolucionou transformação: SQL + Jinja + testes + docs + Git
- Princípios: **idempotência**, **particionamento por tempo**, **backfill estratégico**
- **Data quality testing** é obrigatório (dbt tests, Great Expectations)
- **Batch** vs **Streaming**: latência vs simplicidade
- **Reverse ETL** manda dados do DW pra ferramentas operacionais
- Stack moderno: Fivetran/Airbyte → Snowflake → dbt → Airflow → BI tools

---

⬅️ [Anterior: Data Lakes e Lakehouses](./43-data-lakes.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Streaming de Dados](./45-streaming-dados.md)
