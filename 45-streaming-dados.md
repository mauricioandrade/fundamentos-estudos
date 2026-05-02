# 🌀 Streaming de Dados

> Em batch processing, você espera juntar 1 dia de dados pra processar. Em **streaming**, você processa **cada evento à medida que chega** — em milissegundos. É a diferença entre saber o que vendeu **ontem** vs saber o que está vendendo **agora**. Crítico pra detecção de fraude, monitoramento, real-time analytics.

---

## 🎯 Streaming vs Batch

Já vimos rapidamente em [ETL/ELT](./44-etl-pipelines.md), mas vamos aprofundar.

### Batch

```
00:00 ─────────────── 23:59
   coletando dados o dia inteiro
                      ↓
00:00 do dia seguinte
   processa tudo
                      ↓
06:00 - resultados prontos
```

### Streaming

```
Evento ─→ processa ─→ resultado    (segundos)
Evento ─→ processa ─→ resultado    (segundos)
Evento ─→ processa ─→ resultado    (segundos)
   contínuo, sem pausa
```

> 💡 **Em palavras simples**: batch é fotografia (snapshot de um momento), streaming é vídeo (fluxo contínuo).

---

## 🤔 Quando Streaming Vale a Pena

### ✅ Bons casos pra streaming

- 🚨 **Detecção de fraude**: precisa bloquear transação suspeita imediatamente
- 📊 **Real-time dashboards**: traders, monitoramento de produção
- 🎯 **Recomendações em tempo real**: "você curtiu X, veja Y agora"
- 🔍 **Monitoramento de aplicação**: alertas baseados em eventos
- 🌐 **IoT**: milhões de sensores reportando continuamente
- 📍 **Geolocalização**: rastreamento de motoristas, frotas

### ❌ Casos onde batch é melhor

- 📈 **Relatórios financeiros mensais**
- 🎓 **Treinamento de modelos ML**
- 📊 **BI tradicional** com dashboards diários
- 💾 **Histórico longo** pra análises

> 💡 **Regra de ouro**: streaming é **mais caro e complexo**. Use só quando a latência justifica. Batch micro (a cada 5-15 min) já resolve 80% das "necessidades de tempo real".

---

## 🧱 Conceitos Centrais

### Evento (Event)

Mensagem **imutável** representando algo que aconteceu.

```json
{
  "evento_id": "abc123",
  "tipo": "pedido_criado",
  "timestamp": "2026-04-28T10:30:00Z",
  "dados": {
    "pedido_id": 12345,
    "cliente_id": 678,
    "valor": 299.90
  }
}
```

### Stream

Fluxo **contínuo e ordenado** de eventos.

```
[evento1] → [evento2] → [evento3] → [evento4] → ...
```

### Producer

Quem **publica** eventos no stream. Ex: serviço de pedidos quando alguém compra.

### Consumer

Quem **lê e processa** eventos. Ex: serviço de detecção de fraude.

### Topic / Stream

Onde os eventos ficam. **Categoria lógica**.

```
Topic: pedidos_criados
Topic: pagamentos_aprovados
Topic: usuarios_logaram
```

### Partition

Topic dividido em **partições** pra paralelismo.

```
Topic: pedidos_criados
├── Partition 0: [evt1, evt2, evt3, ...]
├── Partition 1: [evt4, evt5, evt6, ...]
└── Partition 2: [evt7, evt8, evt9, ...]
```

### Offset

Posição de um evento na partição.

```
Partition 0:
[evt1: offset=0, evt2: offset=1, evt3: offset=2, ...]
```

Consumer guarda **até onde leu** (offset).

---

## 🌊 Apache Kafka

**Padrão de fato** pra streaming. Originalmente do LinkedIn, agora Apache.

### Conceito-chave

Kafka é um **log distribuído**. Eventos são **anexados** ao final, **nunca modificados**.

```
[evt1] [evt2] [evt3] [evt4] [evt5] ← novos chegam aqui
  ↑                       ↑
  consumer A             consumer B
  (lendo do início)      (lendo eventos novos)
```

### Por que isso é genial

- 📜 **Múltiplos consumers** leem o mesmo dado
- 🔄 **Replay**: pode ler do início de novo
- ⚡ **Performance**: append-only é super rápido
- 💾 **Durável**: persiste em disco

### Componentes

```
   Producers              Kafka Cluster              Consumers
   ┌─────────┐         ┌─────────────┐         ┌─────────┐
   │ App A   │ ─────→  │  Topic 1    │  ─────→ │ Service X│
   │ App B   │ ─────→  │  Topic 2    │  ─────→ │ Service Y│
   │ App C   │ ─────→  │  Topic 3    │  ─────→ │ Service Z│
   └─────────┘         └─────────────┘         └─────────┘
                       Brokers (servers)
                       Partitioned + Replicated
```

### Producer (Java)

```java
Properties props = new Properties();
props.put("bootstrap.servers", "kafka:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

ProducerRecord<String, String> record = new ProducerRecord<>(
    "pedidos_criados",
    "pedido_12345",
    "{\"id\": 12345, \"valor\": 299.90}"
);

producer.send(record);
producer.close();
```

### Consumer (Java)

```java
Properties props = new Properties();
props.put("bootstrap.servers", "kafka:9092");
props.put("group.id", "fraude-detector");
props.put("key.deserializer", "...StringDeserializer");
props.put("value.deserializer", "...StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("pedidos_criados"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        processar(record.value());
    }
}
```

### Consumer Groups

Múltiplos consumers no **mesmo grupo** dividem partições:

```
Topic: pedidos (3 partições)
Consumer Group: fraude

Consumer A → lê Partition 0
Consumer B → lê Partition 1
Consumer C → lê Partition 2

Cada evento é processado por UM dos consumers do grupo.
```

Múltiplos **grupos diferentes** leem os mesmos eventos:

```
Group fraude:    Consumer A, B, C → leem todos os eventos
Group analytics: Consumer X       → leem todos os eventos (independente do fraude)
```

---

## ⚡ Kafka Streams

Biblioteca pra **processar streams** dentro da própria app Java.

### Exemplo: contar pedidos por categoria em tempo real

```java
StreamsBuilder builder = new StreamsBuilder();

KStream<String, Pedido> pedidos = builder.stream("pedidos_criados");

KTable<String, Long> contagemPorCategoria = pedidos
    .groupBy((key, pedido) -> pedido.getCategoria())
    .count();

contagemPorCategoria.toStream().to("contagem_categorias");

KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

### Conceitos importantes

#### KStream
Fluxo de eventos (cada um único).

#### KTable
Estado **agregado** (último valor por chave).

```
KStream eventos:
  (cat=eletronicos, 100)
  (cat=eletronicos, 200)
  (cat=livros, 50)

Vira KTable:
  eletronicos: 300 (soma)
  livros: 50
```

#### Windowing

Operações em **janelas de tempo**:

```java
// Vendas por minuto
pedidos
    .groupBy((k, v) -> v.getCategoria())
    .windowedBy(TimeWindows.of(Duration.ofMinutes(1)))
    .aggregate(...);
```

Tipos:
- **Tumbling**: janelas fixas, não sobrepõem (00:00-00:01, 00:01-00:02)
- **Hopping**: sobrepõem (00:00-00:05, 00:01-00:06, ...)
- **Sliding**: similar a hopping mas com lógica específica
- **Session**: janelas baseadas em atividade do usuário

---

## 🔥 Apache Flink

Concorrente forte do Kafka Streams. **Stream processing engine** completa.

### Diferenças vs Kafka Streams

| Aspecto | Kafka Streams | Flink |
|---|---|---|
| **Tipo** | Biblioteca em sua app | Cluster/engine separada |
| **Setup** | Simples (lib Java) | Mais complexo (cluster) |
| **Performance** | Boa | Excelente em escala |
| **Estado** | Distribuído junto da app | Gerenciado pela Flink |
| **Linguagens** | Java, Scala | Java, Scala, Python (PyFlink), SQL |
| **Quando usar** | Apps Java em escala média | Workloads pesados, multi-team |

### Flink SQL (super conveniente)

```sql
-- Stream de pedidos
CREATE TABLE pedidos (
    id BIGINT,
    cliente_id BIGINT,
    valor DECIMAL(10,2),
    timestamp TIMESTAMP(3),
    WATERMARK FOR timestamp AS timestamp - INTERVAL '5' SECOND
) WITH (
    'connector' = 'kafka',
    'topic' = 'pedidos_criados',
    'format' = 'json'
);

-- Janela de 1 minuto
SELECT
    TUMBLE_START(timestamp, INTERVAL '1' MINUTE) AS minuto,
    COUNT(*) AS qtd_pedidos,
    SUM(valor) AS receita
FROM pedidos
GROUP BY TUMBLE(timestamp, INTERVAL '1' MINUTE);
```

> 💡 SQL pra streaming! Reduz drasticamente complexidade.

---

## ⏰ Tempo em Streaming

Talvez o conceito **mais difícil** de streaming. Existem 3 noções de tempo:

### Event Time

Quando o evento **aconteceu** (timestamp original).

```json
{ "evento": "compra", "event_time": "2026-04-28T10:30:00Z" }
```

### Processing Time

Quando o sistema **recebeu** o evento.

```
Evento criado: 10:30:00
Recebido pelo Kafka: 10:30:02 (2s de delay de rede)
Processado: 10:30:05 (3s na fila)
```

### Ingestion Time

Quando entrou no sistema de streaming.

### Por que isso importa?

Eventos chegam **fora de ordem**.

```
Ordem real:      [A: 10:00] [B: 10:01] [C: 10:02]
Ordem que chega: [B: 10:01] [A: 10:00] [C: 10:02]
                      ↑
                 chegou antes!
```

Causas: latência de rede, falhas, retries, fusos horários.

### Watermarks

Mecanismo do Flink/Kafka Streams pra dizer "todos eventos até timestamp X já chegaram".

```
Watermark = 10:05 → "considero todos eventos até 10:05 como recebidos"
Janela 10:00-10:05 pode ser fechada e calculada
```

> 🤔 **Trade-off**: watermark agressivo (espera pouco) é rápido mas perde eventos atrasados. Watermark conservador (espera muito) é mais correto mas mais lento.

---

## 🎯 Padrões Importantes

### Exactly-once processing

Já vimos em [Mensageria](./19-mensageria.md). Em streaming, é especialmente difícil.

#### Estratégias
- ✅ **Idempotência** no consumer
- ✅ **Transações** (Kafka Transactions)
- ✅ **Exactly-once com Kafka Streams** (configurar `processing.guarantee=exactly_once_v2`)

### State Management

Pra agregações, joins, etc., você precisa **manter estado**.

```
"Pra cada usuário, contar quantas compras ele fez no último mês"
                                   ↑
                                  estado
```

Onde guardar?
- **Em memória**: rápido mas volátil
- **RocksDB embutido**: Kafka Streams faz isso
- **Banco externo**: Redis, Cassandra
- **Flink State Backend**: gerenciado pela Flink

### Backpressure

Producer publica mais rápido que consumer aguenta. Como reagir?

- 🚨 **Buffer cresce** até estourar
- 📦 **Drop messages** (perde dados)
- 🐌 **Slow down producer** (idealmente)

Stream processors modernos lidam com isso.

### Late Events

Evento chegou depois da janela ter sido fechada. Opções:

- 🚮 **Ignorar** (perde dados)
- 🔄 **Atualizar resultado** (mais complexo, mas correto)
- 📦 **Side output** (envia pra fluxo separado)

---

## 🆚 Streaming Engines: Comparação

| Engine | Tipo | Quando usar |
|---|---|---|
| **Kafka Streams** | Lib Java | App em produção, escala média |
| **Apache Flink** | Engine | Workloads complexos, baixa latência |
| **Apache Spark Streaming** | Engine | Times que já usam Spark batch |
| **Apache Beam** | API unificada | Multi-runner (Flink, Spark, Dataflow) |
| **AWS Kinesis Data Streams** | Cloud | AWS-native |
| **Google Dataflow** | Cloud | GCP-native |
| **Materialize** | DB streaming | SQL incremental, baixa latência |

---

## 🌐 Alternativas ao Kafka

### AWS Kinesis

Versão gerenciada da AWS. Similar ao Kafka mas:
- ✅ Sem operação (gerenciado)
- ❌ Vendor lock-in
- ❌ Menos flexível

### Apache Pulsar

Concorrente moderno. Diferenciais:
- ✅ Storage e compute separados
- ✅ Multi-tenancy nativo
- ✅ Geo-replication melhor
- ❌ Comunidade menor

### Redpanda

Reescrita do Kafka em C++. Mais rápido, sem JVM.
- ✅ Compatible com API Kafka
- ✅ Performance melhor
- ❌ Mais novo, menos maduro

### NATS / RabbitMQ Streams

Pra casos mais simples (não competem direto com Kafka em escala).

---

## 📊 CDC + Streaming

Combinação poderosa: usa **Change Data Capture** ([visto em ETL](./44-etl-pipelines.md)) pra colocar mudanças do banco em stream.

```
PostgreSQL → Debezium → Kafka → Flink → DW (atualizado em segundos)
                                       → Cache (Redis)
                                       → Search (Elastic)
```

> 💡 **Padrão "outbox"** ([visto em Mensageria](./19-mensageria.md)) é caso especial disso.

---

## 🚨 Anti-Patterns

### 1. Streaming pra tudo

Adotar streaming end-to-end sem necessidade real. Custo + complexidade altíssimos.

### 2. Sem ordering quando necessário

Stream sem partition key adequado = eventos do mesmo usuário em partições diferentes = ordering quebrada.

```java
// ❌ Sem chave
producer.send(new ProducerRecord<>(topic, evento));

// ✅ Com chave (mesmo usuário sempre na mesma partição)
producer.send(new ProducerRecord<>(topic, usuarioId, evento));
```

### 3. Janelas mal dimensionadas

- Pequena demais: muito processamento, pouco insight
- Grande demais: latência alta

### 4. Sem monitoramento de lag

```
"Quantos eventos atrasados o consumer está?"
Lag = 1.000 → ok
Lag = 10.000.000 → 🚨 alerta
```

Métricas: Kafka Lag Exporter, Burrow.

### 5. Ignorar exactly-once

"At-least-once + idempotência" funciona. Mas se precisar de exactly-once, configure direito ou bug sutis aparecem.

---

## ✅ Resumo da página

- **Streaming** processa eventos individualmente, em **tempo real**
- **Batch** é mais simples e barato — use streaming só se latência justifica
- **Kafka** é padrão de fato: log distribuído, append-only, replay
- Conceitos: **producer, consumer, topic, partition, offset, consumer group**
- **Kafka Streams** = biblioteca pra processar streams na sua app Java
- **Flink** = engine completa, mais escalável, suporta SQL
- **Tempo em streaming** é complexo: event time, processing time, watermarks
- **State management** crítico pra agregações e joins
- **Late events** e **out-of-order** são realidade — projete pra eles
- **CDC + streaming** = atualização quase real-time entre sistemas
- Alternativas: **Pulsar, Redpanda, Kinesis, Dataflow**

---

⬅️ [Anterior: ETL/ELT e Pipelines](./44-etl-pipelines.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Bancos NoSQL Específicos](./46-nosql-especifico.md)
