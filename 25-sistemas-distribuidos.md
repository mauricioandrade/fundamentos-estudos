# 🌍 Sistemas Distribuídos

> Quando seu sistema cresce além de **uma máquina**, você entra no mundo dos sistemas distribuídos. Aqui as regras mudam: rede falha, relógios mentem, partes do sistema ficam fora enquanto outras seguem. Os "fundamentos" tradicionais não bastam.

---

## 🤔 O Que é um Sistema Distribuído?

Definição clássica de Leslie Lamport:

> "A distributed system is one in which the failure of a computer you didn't even know existed can render your own computer unusable."

Em outras palavras: **múltiplas máquinas trabalhando juntas**, mas com **falhas e atrasos** sendo a regra, não exceção.

### Exemplos do dia a dia

- 🌐 **Web**: cliente + servidor + DB + cache + CDN = sistema distribuído
- 📱 **WhatsApp**: mensagens trafegam por dezenas de servidores
- 🛒 **E-commerce**: pedido toca múltiplos serviços (catálogo, estoque, pagamento)
- ☁️ **Cloud**: literalmente um sistema distribuído gigante

---

## ⚠️ As 8 Falácias dos Sistemas Distribuídos

Erros que **todo dev iniciante** comete:

1. ❌ A rede é **confiável**
2. ❌ A latência é **zero**
3. ❌ A largura de banda é **infinita**
4. ❌ A rede é **segura**
5. ❌ A topologia **não muda**
6. ❌ Existe **um administrador**
7. ❌ O custo de transporte é **zero**
8. ❌ A rede é **homogênea**

> 🤔 Cada vez que você assume uma dessas, seu sistema vai falhar. Sistemas distribuídos forçam você a pensar em **degradação graciosa**: o que acontece quando essa parte cai?

---

## 🎯 Teorema CAP

Em qualquer sistema distribuído, você só pode garantir **2 das 3** propriedades:

```
        Consistency
              ▲
              |
              |
    ┌─────────┼─────────┐
    |         |         |
    ▼         |         ▼
Availability ─┴─ Partition Tolerance
```

| Propriedade | Significa |
|---|---|
| **C — Consistency** | Todos os nós veem o mesmo dado ao mesmo tempo |
| **A — Availability** | Sistema sempre responde (mesmo se desatualizado) |
| **P — Partition Tolerance** | Sistema funciona mesmo se rede particionar |

### Como escolher

Como **partições de rede acontecem** (rede falha, é fato da vida), na prática você escolhe entre:

- **CP** (Consistência + Tolerância): em caso de partição, prefere recusar requests
- **AP** (Disponibilidade + Tolerância): em caso de partição, responde com dado possivelmente desatualizado

### Exemplos

| Sistema | Trade-off |
|---|---|
| **Bancos de dados SQL clássicos** | CP (consistência forte) |
| **DynamoDB, Cassandra** | AP (eventual consistency) |
| **Redis (cluster)** | AP |
| **MongoDB (config padrão)** | AP, mas configurável |
| **Sistema bancário** | CP (não pode ter saldo errado!) |
| **Twitter/Instagram feed** | AP (quem se importa se falta 1 like?) |

---

## 🔄 Consistency Models

### Strong Consistency

Todo nó vê o mesmo dado **imediatamente** após escrita.

✅ Simples de raciocinar
❌ Lento (precisa coordenar)

### Eventual Consistency

Eventualmente todos os nós convergem pra mesma versão.

✅ Rápido, escalável
❌ Pode ler dado "velho" temporariamente

### Causal Consistency

Operações causalmente relacionadas têm ordem garantida; outras podem variar.

### Read-your-writes

Você sempre lê suas próprias escritas, mesmo que outros não vejam ainda.

> 💡 **Em apps modernos**: maioria das operações são **eventualmente consistentes**, com algumas críticas em **strong consistency** (saldo, estoque, leilões).

---

## 🤝 Consenso Distribuído

Como múltiplos nós **concordam** sobre algo (líder eleito, valor de variável, ordem de eventos)?

### O Problema

Em rede instável, com possíveis traições:
- Como decidir quem é o líder?
- Como acordar sobre uma transação?
- Como processar pedidos em ordem?

### Algoritmos clássicos

#### Paxos

Algoritmo seminal (Leslie Lamport, 1989). Garante consenso mesmo com falhas. Famoso por ser **difícil de entender e implementar**.

#### Raft

Criado pra ser **mais simples** que Paxos (Stanford, 2014). Mesmo objetivo, mais didático.

```
Estados:
- Follower: aceita comandos do líder
- Candidate: tenta virar líder
- Leader: coordena replicação
```

**Usado em**: etcd (Kubernetes), CockroachDB, Consul.

#### Byzantine Fault Tolerance (BFT)

Pra cenários onde nós podem ser **maliciosos** (não só falharem). Base de blockchain.

**Usado em**: blockchains (Bitcoin usa Proof of Work, outras usam BFT puro).

---

## 🕰️ Tempo em Sistemas Distribuídos

### Problema clássico

Se servidor A diz "12:30:00" e servidor B diz "12:30:01", qual evento aconteceu primeiro?

**Resposta**: você não sabe. Relógios físicos divergem. NTP ajuda mas não resolve.

### Soluções

#### Logical Clocks (Lamport)

Cada operação incrementa um contador. Mensagem entre nós sincroniza contadores.

#### Vector Clocks

Cada nó mantém um vetor com contador de cada outro nó. Permite detectar **concorrência** vs causalidade.

#### Hybrid Logical Clocks (HLC)

Combina relógio físico + lógico. Usado em CockroachDB.

> 💡 **Lição**: ordering em sistemas distribuídos é difícil. Quando precisar (logs, eventos, transações), use ferramentas que lidam com isso (Kafka mantém ordem por partição, por exemplo).

---

## 📦 Replicação

Cópias dos dados em múltiplos nós.

### Por quê?

- 🛡️ **Disponibilidade**: se 1 cai, outras servem
- 🚀 **Performance**: leituras locais (sem ir longe)
- 💾 **Durabilidade**: dado em múltiplos lugares = mais difícil perder

### Modelos

#### Single-leader (Master-Slave)

Um nó é primário (escritas), outros são réplicas (leituras).

```
Cliente escreve → Leader → replica → Followers (read replicas)
```

✅ Simples, evita conflitos
❌ Single point of failure

**Usado em**: PostgreSQL streaming replication, MySQL master-slave.

#### Multi-leader

Vários nós aceitam escritas, sincronizam entre si.

✅ Disponibilidade alta
❌ Conflitos podem ocorrer

#### Leaderless

Cliente escreve em vários nós direto. Quórum decide.

**Usado em**: Cassandra, DynamoDB.

### Quórum

Pra leitura/escrita ser bem-sucedida, **N nós** devem confirmar.

```
Se você tem 5 réplicas (R=5)
Write quorum (W) = 3
Read quorum (R) = 3
W + R > N → consistência forte (3 + 3 > 5 ✓)
```

---

## 💾 Sharding (Particionamento Horizontal)

Quando o dataset não cabe num nó só, **divide** entre múltiplos.

```
Tabela usuarios (10 milhões de registros)
├── Shard 1: usuários A-G
├── Shard 2: usuários H-N
├── Shard 3: usuários O-T
└── Shard 4: usuários U-Z
```

### Estratégias de sharding

| Estratégia | Como divide |
|---|---|
| **Range** | Por faixa de valores |
| **Hash** | Hash do ID escolhe shard |
| **Geográfico** | Por região do usuário |
| **Diretório** | Lookup table central |

### Trade-offs

✅ Escala horizontalmente
❌ Queries cross-shard são caras
❌ Rebalanceamento é complexo
❌ Joins entre shards são limitados

---

## 🌊 Event Sourcing e CQRS (revisitando)

Já vistos no [tópico de Arquitetura](./15-arquitetura-software.md), mas brilham em sistemas distribuídos:

### Event Sourcing

Em vez de salvar **estado**, salva **eventos**. Estado é derivado.

```
Eventos:
- ContaCriada
- DepositoFeito(100)
- SaqueFeito(30)

Estado atual = soma = 70
```

✅ Auditoria perfeita
✅ Replay/debugging
✅ Múltiplas projeções (vistas)
❌ Complexidade alta

### CQRS

Separa **escritas** (commands) de **leituras** (queries).

```
Write side: modelo otimizado pra mudanças
Read side: múltiplos modelos otimizados pra leituras específicas
```

---

## 🏗️ Padrões Distribuídos

### Circuit Breaker

Se um serviço dependente está fora, **abre o circuito** e falha rápido em vez de timeout.

```
Estados:
- Closed: tudo normal
- Open: muitas falhas, falha rápido
- Half-Open: testa periodicamente se voltou
```

**Implementação**: Hystrix (Netflix, descontinuado), Resilience4j, Polly (.NET).

### Retry com backoff

Tentar de novo, com espera crescente. Já visto em [Mensageria](./19-mensageria.md).

### Bulkhead

Isola recursos por tipo de cliente. Se um tipo sobrecarrega, outros não sofrem.

```
Pool de threads:
├── 50 threads pra clientes premium
└── 50 threads pra clientes free
```

### Idempotency

Operações que podem ser executadas N vezes com mesmo resultado. Crítico em retries.

### Two-Phase Commit (2PC)

Coordenador garante que todos os nós confirmam ou cancelam transação.

```
Fase 1 - Prepare: "Você pode fazer X?"
Fase 2 - Commit: "Faça X" (ou "Cancele")
```

⚠️ Bloqueante e frágil. Saga (já visto) é alternativa moderna.

---

## 🌐 Service Mesh

Camada de **infraestrutura** que cuida de comunicação entre serviços.

```
App ──→ Sidecar Proxy ──→ Network ──→ Sidecar Proxy ──→ Other App
              ↑                                    ↑
              └───────── Control Plane ───────────┘
```

**Funcionalidades:**
- 🔒 mTLS (mutual TLS) automático
- 🔄 Retry, timeout, circuit breaker
- 📊 Observabilidade
- 🚦 Roteamento avançado (canary, A/B)

**Implementações**: Istio, Linkerd, Consul Connect.

---

## ✅ Resumo da página

- Sistemas distribuídos = múltiplas máquinas com **falhas e atrasos** como regra
- Conheça as **8 falácias** — todo iniciante comete
- **CAP**: você escolhe entre Consistência e Disponibilidade quando há partição
- **Consistency models**: strong (lento, simples), eventual (rápido, complexo)
- **Consenso**: Raft é o algoritmo moderno mais usado
- **Tempo distribuído** é difícil — use logical clocks ou ferramentas que abstraem
- **Replicação**: single-leader (simples), multi-leader (HA), leaderless (escalável)
- **Sharding** divide dados horizontalmente quando crescem
- **Padrões essenciais**: Circuit Breaker, Retry+Backoff, Bulkhead, Idempotency, Saga
- **Service Mesh** abstrai comunicação entre microserviços

---

⬅️ [Anterior: Cloud](./24-cloud.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Fundamentos de IA/ML](./26-fundamentos-ia.md)
