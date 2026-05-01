# 📨 Mensageria: Comunicação Assíncrona entre Sistemas

> Quando você tem múltiplos serviços que precisam conversar, chamadas síncronas (HTTP) acoplam tudo. Mensageria desacopla — um serviço **dispara um evento** e segue a vida; outros consomem quando puderem.

---

## 🤔 O Problema das Chamadas Síncronas

Imagine um e-commerce. Quando o usuário finaliza um pedido, você precisa:

1. ✅ Validar pagamento
2. 📦 Reservar estoque
3. 📧 Enviar email de confirmação
4. 📊 Atualizar analytics
5. 🚚 Notificar transportadora

**Síncrono (cada serviço chamando o próximo):**
```
Pedido → Pagamento → Estoque → Email → Analytics → Transportadora
   |________________________________________________|
                  Se UM falhar, TUDO falha
```

**Problemas:**
- 🐢 Latência acumula (soma de todos os tempos)
- 💥 Falha em cascata (email fora? pedido inteiro falha)
- 🔗 Acoplamento forte (Pedido **conhece** todos os outros)

---

## ✅ A Solução: Mensageria

Em vez de chamar diretamente, o Pedido **publica um evento** num **broker** (intermediário). Quem se interessar consome.

```
Pedido publica → [Broker] → Pagamento (consome)
                         → Estoque (consome)
                         → Email (consome)
                         → Analytics (consome)
                         → Transportadora (consome)
```

> 💡 **Em palavras simples**: o Pedido grita "criei um pedido!" no megafone, e todos os interessados ouvem e fazem sua parte. O Pedido nem precisa saber **quem** está ouvindo.

### Vantagens

- 🚀 **Performance**: Pedido responde rápido, não espera os outros
- 🛡️ **Resiliência**: se Email estiver fora, mensagens ficam guardadas
- 🔄 **Desacoplamento**: novo serviço se inscreve sem mudar o Pedido
- 📈 **Escalabilidade**: cada consumidor escala independente

---

## 🧱 Conceitos Centrais

### Producer (Produtor)

Quem **publica** mensagens. No exemplo, o serviço de Pedido.

### Consumer (Consumidor)

Quem **lê e processa** mensagens. Email, Estoque, etc.

### Broker

Software intermediário que armazena e roteia mensagens. Ex: RabbitMQ, Kafka, AWS SQS.

### Queue (Fila)

Estrutura **FIFO** onde mensagens aguardam consumo. Cada mensagem é entregue **a um consumer** (compartilhada entre instâncias do mesmo serviço).

### Topic (Tópico)

Como uma fila, mas com **múltiplos consumidores independentes**. Cada serviço recebe sua cópia.

> 🤔 **Quando usar fila vs tópico?**
> - **Fila**: trabalho compartilhado (ex: 5 workers de email dividindo carga)
> - **Tópico**: notificação broadcast (ex: todos os serviços precisam saber do novo pedido)

---

## 🐰 RabbitMQ

Broker tradicional, baseado no protocolo **AMQP**. Maduro, estável, fácil de operar.

### Modelo

```
Producer → Exchange → Queue → Consumer
            (roteador)
```

**Tipos de Exchange:**
- `direct`: roteia por chave exata
- `fanout`: broadcast pra todas as queues
- `topic`: roteia por padrões (ex: `pedido.*`)
- `headers`: roteia por metadados

### Exemplo: Spring Boot

```java
// Configuração
@Configuration
public class RabbitConfig {
    @Bean
    Queue pedidosQueue() {
        return new Queue("pedidos.criados", true);  // durable
    }
}

// Producer
@Service
public class PedidoService {
    private final RabbitTemplate rabbit;

    public void criar(Pedido p) {
        // ... salva no banco
        rabbit.convertAndSend("pedidos.criados", p);
    }
}

// Consumer
@Service
public class EmailConsumer {
    @RabbitListener(queues = "pedidos.criados")
    public void onPedidoCriado(Pedido p) {
        emailService.enviarConfirmacao(p);
    }
}
```

### Equivalentes em outras stacks

```typescript
// Node.js + amqplib
const channel = await connection.createChannel();
await channel.assertQueue('pedidos.criados');
channel.sendToQueue('pedidos.criados', Buffer.from(JSON.stringify(pedido)));
```

```python
# Python + pika
channel.queue_declare(queue='pedidos.criados')
channel.basic_publish(exchange='', routing_key='pedidos.criados', body=json.dumps(pedido))
```

---

## 🌊 Apache Kafka

Diferente do RabbitMQ. Kafka é uma **plataforma de streaming** — pense num "log distribuído" infinito.

### Diferenças cruciais

| Aspecto | RabbitMQ | Kafka |
|---|---|---|
| **Modelo** | Mensagem some após consumir | Mensagem fica no log |
| **Replay** | ❌ Não dá pra reprocessar | ✅ Releia o que quiser |
| **Throughput** | ~50k msg/s | Milhões msg/s |
| **Latência** | Muito baixa | Baixa |
| **Uso típico** | Tarefas, comandos | Eventos, analytics, streams |

### Conceitos do Kafka

- **Topic**: o "log" onde mensagens são escritas
- **Partition**: divisão do tópico pra paralelismo
- **Offset**: posição da mensagem no log
- **Consumer Group**: grupo de consumers que dividem partições

```
Topic: pedidos
├── Partition 0: [msg1] [msg2] [msg3] ...
├── Partition 1: [msg4] [msg5] [msg6] ...
└── Partition 2: [msg7] [msg8] [msg9] ...
```

> 💡 **Quando escolher Kafka?** Quando você precisa de:
> - Volumes massivos (logs, telemetria, IoT)
> - Replay de eventos (event sourcing, debugging)
> - Múltiplos consumers independentes lendo o mesmo dado

---

## ⚖️ Garantias de Entrega

Trade-off importante:

| Garantia | Significa | Custo |
|---|---|---|
| **At most once** | No máximo 1 vez (pode perder) | Mais rápido, menos confiável |
| **At least once** | Ao menos 1 vez (pode duplicar) | Confiável, exige idempotência |
| **Exactly once** | Exatamente 1 vez | Lento, complexo, raramente real |

> ⚠️ **Importante**: "exactly once" é difícil/caro. Maioria dos sistemas usa "at least once" + **idempotência** no consumer.

### Idempotência

Operação que pode ser executada **várias vezes** com o mesmo resultado:

```java
// ❌ Não-idempotente
saldo += 100;  // executar 2x = +200

// ✅ Idempotente
if (!processedIds.contains(msg.id)) {
    saldo += 100;
    processedIds.add(msg.id);
}
```

---

## 🚨 Padrões e Problemas Comuns

### Dead Letter Queue (DLQ)

Fila pra mensagens que **falharam** repetidamente. Em vez de descartar, guarda pra análise.

```
[Queue Principal] → 3 falhas → [DLQ] → analista investiga
```

### Retry com backoff

Tentar de novo, mas esperando cada vez mais:

```
Tentativa 1: imediato
Tentativa 2: após 1s
Tentativa 3: após 5s
Tentativa 4: após 30s
Tentativa 5: → DLQ
```

### Outbox Pattern

Garante que evento é publicado **junto** com mudança no DB:

```
1. INSERT no Pedido + INSERT na tabela "outbox" (mesma transação)
2. Worker lê tabela "outbox" e publica no broker
3. Marca como publicado
```

> 🤔 **Por que esse padrão existe?** Sem ele, você pode salvar pedido no banco mas falhar ao publicar evento. Ou publicar e falhar ao salvar. Outbox garante atomicidade.

### Saga Pattern

Pra transações **distribuídas**. Em vez de transação única (impossível entre serviços), você faz uma sequência com **compensações**:

```
Pedido criado → Pagamento aprovado → Estoque reservado
                                      ↓ FALHA
                              Estornar pagamento ← Cancelar pedido
                              (compensação)
```

---

## 🎯 Tabela: RabbitMQ vs Kafka vs Redis Pub/Sub vs SQS

| Critério | RabbitMQ | Kafka | Redis Pub/Sub | AWS SQS |
|---|---|---|---|---|
| **Persistência** | ✅ | ✅ | ❌ (perde se cair) | ✅ |
| **Throughput** | Médio | 🔥 Altíssimo | Médio | Médio |
| **Replay** | ❌ | ✅ | ❌ | ❌ |
| **Operação** | Média | Complexa | Simples | Gerenciada |
| **Caso típico** | Tarefas, RPC | Streaming, eventos | Notificações leves | Filas simples |

---

## ✅ Resumo da página

- Mensageria **desacopla** serviços e melhora resiliência
- **Producer publica** no broker, **consumers consomem**
- **Filas** distribuem trabalho; **tópicos** fazem broadcast
- **RabbitMQ**: broker maduro, baseado em AMQP
- **Kafka**: log distribuído, alto throughput, suporta replay
- Garantia mais comum: **at-least-once + idempotência**
- Padrões essenciais: **DLQ**, **retry com backoff**, **Outbox**, **Saga**
- Em microserviços, mensageria é quase obrigatória pra escala

---

⬅️ [Anterior: Observabilidade](./18-observabilidade.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Caching](./20-caching.md)
