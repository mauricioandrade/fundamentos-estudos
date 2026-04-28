# 🔍 Observabilidade: Logs, Métricas e Tracing

> Sua aplicação está em produção. Algo deu errado. Como você descobre **o quê**, **onde**, **quando** e **por quê**? Observabilidade é a resposta.

---

## 🤔 Monitoramento ≠ Observabilidade

Diferença sutil mas importante:

- **Monitoramento**: você sabe **quais perguntas fazer** ("CPU está alta?", "tem erro 500?"). Você configura alertas pra essas perguntas.
- **Observabilidade**: você consegue **fazer perguntas que ainda não imaginou** investigando os dados depois.

> 💡 **Em palavras simples**: monitoramento é checklist. Observabilidade é detetive. Sistemas distribuídos modernos exigem observabilidade — você não consegue prever todos os problemas.

---

## 🎯 Os 3 Pilares

```
        OBSERVABILIDADE
        /      |      \
       /       |       \
      LOGS  MÉTRICAS  TRACING
   (eventos)  (números)  (jornadas)
```

| Pilar | Pergunta que responde |
|---|---|
| 📜 **Logs** | "O que aconteceu nesse momento exato?" |
| 📊 **Métricas** | "Como está se comportando ao longo do tempo?" |
| 🛤️ **Traces** | "Por onde passou essa requisição específica?" |

Você precisa dos 3 pra ter cobertura completa.

---

## 📜 Logs

Registros de eventos em texto. O mais antigo dos 3, ainda essencial.

### Níveis de log

```
TRACE   → super detalhado (dev local)
DEBUG   → detalhes pra investigação
INFO    → eventos normais ("usuário cadastrado")
WARN    → algo estranho, mas funcionou
ERROR   → algo falhou
FATAL   → app vai morrer
```

### Logs Estruturados (JSON)

Em vez de texto livre, use JSON. Ferramentas conseguem **filtrar e agregar**.

```java
// ❌ Ruim - texto livre
log.info("Usuário " + email + " logou às " + horario);

// ✅ Bom - estruturado
log.info("usuario_logou", Map.of(
    "email", email,
    "horario", horario,
    "ip", request.getRemoteAddr()
));
```

Output:
```json
{
  "timestamp": "2026-01-15T10:30:00Z",
  "level": "INFO",
  "event": "usuario_logou",
  "email": "mau@example.com",
  "horario": "10:30",
  "ip": "192.168.1.1",
  "trace_id": "abc-123"
}
```

### O que NÃO logar

- 🔐 Senhas, tokens, chaves de API
- 💳 Cartões de crédito, CPF completo, dados sensíveis
- 📦 Payloads gigantes (use IDs e busque depois)

> ⚠️ **GDPR e LGPD se importam com isso.** Logar dados pessoais sem necessidade é violação de lei.

### Onde os logs vão?

Em produção, **não basta** escrever em arquivo. Você quer agregar logs de **N instâncias** num lugar central:

| Stack | Componentes |
|---|---|
| **ELK** | Elasticsearch + Logstash + Kibana |
| **EFK** | Elasticsearch + Fluentd + Kibana |
| **Loki** | Grafana Loki (mais leve) |
| **Cloud** | AWS CloudWatch, GCP Logging, Datadog Logs |

---

## 📊 Métricas

Números agregados ao longo do tempo. Eficiente em armazenamento, ideal pra dashboards e alertas.

### Tipos de métricas

| Tipo | O que mede | Exemplo |
|---|---|---|
| **Counter** | Só sobe | Total de requisições |
| **Gauge** | Sobe e desce | Memória atual, conexões ativas |
| **Histogram** | Distribuição | Latência (p50, p95, p99) |
| **Summary** | Similar ao histogram | Quantis pré-calculados |

### Os "Quatro Sinais Dourados" (Google SRE)

Pra qualquer serviço, monitore:

1. **Latency** — quanto tempo demora
2. **Traffic** — quantas requisições
3. **Errors** — taxa de erro
4. **Saturation** — quão sobrecarregado está

### USE Method (pra recursos)

Pra cada recurso (CPU, RAM, disco):
- **U**tilization
- **S**aturation
- **E**rrors

### RED Method (pra serviços)

- **R**ate
- **E**rrors
- **D**uration

### Stack típica

```
[Sua App] → métricas → [Prometheus] → [Grafana]
                                ↓
                          [Alertmanager]
                                ↓
                        Slack / Email / PagerDuty
```

### Métricas em frameworks modernos

A maioria dos frameworks tem instrumentação built-in pra Prometheus:

- **Spring Boot (Java)**: Actuator + Micrometer (exemplo abaixo)
- **Node.js**: `prom-client`
- **Python**: `prometheus_client` (FastAPI tem integração nativa)
- **.NET**: `prometheus-net`
- **Go**: `prometheus/client_golang`

### Exemplo: Spring Boot Actuator + Micrometer

Spring Boot tem suporte built-in:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=health,info,prometheus,metrics
```

Acesse `/actuator/prometheus` e o Prometheus coleta automaticamente.

### Métricas customizadas

```java
@Service
public class PedidoService {
    private final Counter pedidosCriados;

    public PedidoService(MeterRegistry registry) {
        this.pedidosCriados = Counter.builder("pedidos.criados")
            .description("Total de pedidos criados")
            .register(registry);
    }

    public void criar(Pedido p) {
        // ...
        pedidosCriados.increment();
    }
}
```

---

## 🛤️ Tracing Distribuído

Em microserviços, uma requisição passa por **vários serviços**. Como saber onde está a lentidão?

### O conceito

Cada requisição recebe um **trace ID** único, propagado por todos os serviços. Cada operação dentro de um serviço vira um **span**.

```
Trace ID: abc-123
│
├─ Span: API Gateway        [10ms]
│   │
│   ├─ Span: User Service   [50ms]
│   │   └─ Span: DB query   [40ms] ← gargalo!
│   │
│   └─ Span: Order Service  [200ms]
│       ├─ Span: DB query   [30ms]
│       └─ Span: Payment    [150ms]
│
└─ Total: 260ms
```

### Stack

- **OpenTelemetry**: padrão aberto pra coleta
- **Jaeger** ou **Zipkin**: backend de visualização
- **Datadog APM**, **New Relic**: alternativas comerciais

### Em frameworks modernos

- **Spring Boot (Java)**: Micrometer Tracing + OpenTelemetry
- **Node.js**: `@opentelemetry/sdk-node`
- **Python**: `opentelemetry-api` + `opentelemetry-sdk`
- **.NET**: `OpenTelemetry.Extensions.Hosting`
- **Go**: `go.opentelemetry.io/otel`

### Exemplo: Spring Boot

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
```

Spring propaga o trace ID **automaticamente** entre chamadas HTTP. Você só precisa configurar exportador.

> 💡 **Junte os 3 pilares**: log estruturado com `trace_id` permite que você clique num trace no Jaeger e veja **logs daquela requisição específica** no Kibana.

---

## 🚨 Alertas

Métrica passou de um limite → alguém é notificado.

### Boas práticas

- ✅ **Alerte só em coisas acionáveis**. "CPU 80%" sem ação a tomar = ruído
- ✅ **Severidade clara**: P1 (acordar à noite), P2 (horário comercial), P3 (FYI)
- ✅ **Runbook**: cada alerta deve ter "o que fazer quando dispara"
- ❌ **Alert fatigue**: muitos alertas → ninguém olha → o real é perdido

### Exemplos de alertas úteis

```yaml
# Prometheus AlertManager
- alert: HighErrorRate
  expr: rate(http_requests_total{status="500"}[5m]) > 0.05
  annotations:
    summary: "Mais de 5% das requisições estão dando 500"

- alert: APIHighLatency
  expr: histogram_quantile(0.95, http_duration_seconds) > 1
  annotations:
    summary: "p95 de latência acima de 1s"

- alert: DatabaseDown
  expr: up{job="postgres"} == 0
  annotations:
    summary: "Postgres caiu"
```

---

## 💚 Health Checks

Endpoints simples pra verificar se a app está viva.

### Liveness vs Readiness

| Tipo | Pergunta | Se falhar |
|---|---|---|
| **Liveness** | "App está viva?" | Reinicia o container |
| **Readiness** | "App está pronta pra receber tráfego?" | Tira do load balancer |

### Spring Boot Actuator

```
GET /actuator/health
```

```json
{
  "status": "UP",
  "components": {
    "db": { "status": "UP" },
    "redis": { "status": "UP" },
    "diskSpace": { "status": "UP" }
  }
}
```

> 💡 **Por que separar?** App pode estar viva mas não pronta (ex: ainda carregando cache). Reiniciar não resolveria.

---

## 🎯 Erros e Exception Tracking

Logar exceções no log é OK, mas ferramentas dedicadas são melhores.

### Sentry (popular)

- Agrupa exceções iguais
- Stack trace completo
- Contexto da request
- Alerta no Slack/email

```java
// Spring Boot
Sentry.captureException(exception);
```

> 💡 **Diferença pra logs**: Sentry agrega, deduplica e gera dashboard de "top exceptions". Você não fica caçando em GB de logs.

---

## 📈 SLI, SLO, SLA

Vocabulário de SRE (Site Reliability Engineering):

| Sigla | O que é |
|---|---|
| **SLI** (Indicator) | Métrica que mede qualidade. Ex: "% de requests com sucesso" |
| **SLO** (Objective) | Meta interna. Ex: "99.9% sucesso no mês" |
| **SLA** (Agreement) | Contrato com cliente. Ex: "99.5% ou multa" |

Hierarquia: **SLI < SLO < SLA**. SLA sempre pior que SLO (margem de segurança).

### Error Budget

Se SLO é 99.9%, você tem **0.1% de erro budget** por mês. Quando acaba, **PARA de fazer features** e investe em estabilidade.

> 💡 É um equilibrio: 100% de uptime é caro e desnecessário. Definir SLO honesto evita over-engineering.

---

## 🔧 Ferramentas Mencionadas

| Categoria | Ferramentas |
|---|---|
| **Logs** | ELK Stack, Loki, CloudWatch, Datadog Logs |
| **Métricas** | Prometheus + Grafana, Datadog, New Relic |
| **Tracing** | Jaeger, Zipkin, OpenTelemetry |
| **Erros** | Sentry, Rollbar, Bugsnag |
| **APM completo** | Datadog, New Relic, Dynatrace, AppDynamics |
| **Alertas** | Alertmanager, PagerDuty, OpsGenie |

> 💡 **Pra projetos pequenos/médios**, Grafana + Prometheus + Loki + Jaeger (todos open-source) cobrem tudo.

---

## ✅ Resumo da página

- 3 pilares: **logs** (eventos), **métricas** (números), **traces** (jornadas)
- Use **logs estruturados** (JSON), nunca logue dados sensíveis
- 4 sinais dourados: **latency, traffic, errors, saturation**
- **Prometheus + Grafana** é a stack open-source padrão
- **OpenTelemetry** é o padrão emergente pra coletar tudo
- **Health checks** distinguem **liveness** (vivo?) de **readiness** (pronto?)
- **Sentry** > log pra exceptions (agrupa e deduplica)
- **SLI/SLO/SLA + error budget** equilibram velocidade vs confiabilidade
- Frameworks modernos integram com Prometheus/OpenTelemetry quase de graça (Spring Actuator + Micrometer, prom-client em Node, prometheus_client em Python, etc)

---

⬅️ [Anterior: Testes](./17-testes.md) | 🔙 [Voltar ao índice](./README.md)
