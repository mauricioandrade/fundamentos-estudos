# 📈 Performance e Profiling

> "Premature optimization is the root of all evil" — Donald Knuth. Mas otimização **na hora certa** transforma apps lentas em rápidas. A chave: **medir antes de otimizar**.

---

## 🎯 Princípios Fundamentais

### 1. Meça, não adivinhe

```
❌ "Acho que esse loop é lento"
✅ "Profiling mostrou que esse loop é 80% do tempo"
```

### 2. Otimize o gargalo certo

Lei de Amdahl: melhorar 90% de algo que ocupa 10% do tempo total = melhoria geral de 9%. **Foco no que mais consome.**

### 3. Performance é trade-off

| Você ganha em | Geralmente perde em |
|---|---|
| Velocidade | Memória ou complexidade |
| Memória | Velocidade ou flexibilidade |
| Throughput | Latência |
| Latência | Throughput |

> 🤔 **Não existe almoço grátis.** Toda otimização tem um custo.

---

## 📊 Métricas que Importam

### Latência (latency)

Quanto tempo **uma operação** leva. Medida em ms, μs.

- **p50** (mediana): metade das requests é mais rápida
- **p95**: 95% das requests são mais rápidas
- **p99**: 99% das requests são mais rápidas
- **p999**: 99.9% das requests são mais rápidas

> 💡 **Por que olhar percentis altos?** Média esconde problemas. Você pode ter média de 100ms mas p99 de 5 segundos — significa que 1% dos usuários espera 50x mais. Em apps com milhões de usuários, isso é muita gente.

### Throughput

Quantas operações por segundo (RPS — Requests Per Second).

### Saturação

Quão sobrecarregado está o recurso (CPU, memória, disco, rede). >80% sustentado = problema.

### Erros

Taxa de falhas. Performance perfeita com 50% de erros é inútil.

> 🎯 Os **4 sinais dourados** ([visto em Observabilidade](./18-observabilidade.md)): Latency, Traffic, Errors, Saturation.

---

## 🔍 Profiling: Onde Está o Gargalo?

### CPU Profiling

Mostra **onde a CPU gasta tempo**. Identifica:
- Funções lentas
- Loops ineficientes
- Algoritmos errados

### Memory Profiling

Mostra **alocação e liberação** de memória. Identifica:
- Memory leaks
- Allocations excessivas
- Objetos grandes desnecessários

### I/O Profiling

Mostra tempo gasto em **disco e rede**. Identifica:
- Queries lentas
- Falta de cache
- Chamadas síncronas mal feitas

---

## ☕ Profiling em Java (JVM)

### Ferramentas

| Ferramenta | Quando usar |
|---|---|
| **JFR (Java Flight Recorder)** | Profiling em produção (low overhead) |
| **VisualVM** | Profiling local fácil |
| **async-profiler** | Profiling preciso, baixo overhead |
| **YourKit / JProfiler** | Pagas, super completas |

### Exemplo: JFR

```bash
# Habilitar JFR
java -XX:StartFlightRecording=duration=60s,filename=profile.jfr MyApp

# Analisar com JDK Mission Control (JMC)
jmc profile.jfr
```

### Heap Dump (memória)

```bash
# Captura heap quando app está rodando
jmap -dump:live,format=b,file=heap.hprof <PID>

# Analisa com Eclipse MAT (Memory Analyzer Tool)
```

### Garbage Collection Tuning

```bash
# Ver logs do GC
java -Xlog:gc*:file=gc.log MyApp

# Coletores principais (Java moderno)
-XX:+UseG1GC          # padrão moderno (Java 9+)
-XX:+UseZGC           # baixíssima latência
-XX:+UseShenandoahGC  # alternativa low-latency
```

> 💡 **Regra**: não tune GC sem profiling. Defaults são bons pra 90% dos casos.

---

## 🟢 Profiling em outras stacks

### Node.js

```bash
# Built-in profiler
node --prof app.js
node --prof-process isolate-*.log > profile.txt

# Clinic.js (mais visual)
npx clinic doctor -- node app.js
```

Chrome DevTools também aceita conexão remota pra profiling.

### Python

```python
# cProfile (built-in)
python -m cProfile -o profile.out app.py

# Visualizar
pip install snakeviz
snakeviz profile.out

# py-spy (sampling profiler, baixo overhead)
py-spy record -o profile.svg -- python app.py
```

### .NET

- **dotnet-trace**, **PerfView**, **dotMemory** (JetBrains)

### Go

```go
// pprof built-in
import _ "net/http/pprof"
// Acesse /debug/pprof/profile
```

```bash
go tool pprof http://localhost:6060/debug/pprof/profile
```

---

## 🗄️ Performance de Banco de Dados

Geralmente o gargalo principal de apps web.

### Identificar queries lentas

#### PostgreSQL
```sql
-- Habilitar log de queries lentas (>500ms)
ALTER SYSTEM SET log_min_duration_statement = 500;

-- pg_stat_statements pra estatísticas agregadas
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC LIMIT 20;
```

#### MySQL
```sql
-- Slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.5;
```

### EXPLAIN ANALYZE (já vimos no [tópico 8](./08-banco-de-dados.md))

```sql
EXPLAIN ANALYZE
SELECT * FROM pedidos WHERE usuario_id = 123 ORDER BY data DESC;
```

Procure por:
- ❌ **Seq Scan** em tabela grande (precisa índice)
- ❌ **Sort** sem index (custoso em muitos dados)
- ❌ **Hash Join** quando tem dados pequenos (deveria ser nested loop)

### Otimizações comuns

| Problema | Solução |
|---|---|
| Query escaneando tabela inteira | Adicionar índice |
| N+1 queries | JOIN ou eager loading |
| Muitas joins | Desnormalização ou materialized view |
| ORDER BY lento | Índice na coluna de ordenação |
| LIKE '%termo%' lento | Full-text search (tsvector no Postgres) |
| Conexões muito altas | Connection pool (HikariCP) |

---

## 🌐 Performance Web (Frontend)

### Core Web Vitals (Google)

3 métricas que afetam SEO e UX:

| Métrica | O que mede | Bom |
|---|---|---|
| **LCP** (Largest Contentful Paint) | Tempo até carregar conteúdo principal | < 2.5s |
| **INP** (Interaction to Next Paint) | Latência de interação | < 200ms |
| **CLS** (Cumulative Layout Shift) | Layout pulando enquanto carrega | < 0.1 |

### Ferramentas de medição

- **Lighthouse** (Chrome DevTools)
- **PageSpeed Insights** ([pagespeed.web.dev](https://pagespeed.web.dev))
- **WebPageTest** ([webpagetest.org](https://webpagetest.org))

### Otimizações comuns

#### Bundles e assets
- 🗜️ Minificação (Webpack, Vite, esbuild)
- 📦 Tree-shaking (remove código não usado)
- 🔪 Code splitting (carrega só o que precisa)
- 🗂️ Lazy loading de imagens (`loading="lazy"`)
- 🎨 Otimizar imagens (WebP, AVIF, dimensões corretas)

#### Cache HTTP
```http
Cache-Control: public, max-age=31536000, immutable
```
Cache de 1 ano pra assets com hash no nome.

#### CDN
Já visto. Aproxima conteúdo do usuário.

---

## 🔥 Anti-Patterns Clássicos

### 1. Premature Optimization

```java
// ❌ Otimização antes de medir
// Cache de tudo, código complexo, sem ganho real

// ✅ Mede primeiro, otimiza onde dói
```

### 2. Micro-otimizações inúteis

```java
// Não importa se é for ou foreach
// Não importa se é if-else ou ternário
// Otimize ALGORITMO antes (O(n²) → O(n log n) é 1000x melhor)
```

### 3. N+1 (recorrente)

```java
List<Usuario> users = repo.findAll();
users.forEach(u -> System.out.println(u.getPedidos().size()));
// 1 query + N queries = lento
```

### 4. Loops aninhados desnecessários

```java
// ❌ O(n²)
for (Pedido p : pedidos) {
    for (Item i : itens) {
        if (i.pedidoId.equals(p.id)) { ... }
    }
}

// ✅ O(n) — agrupa primeiro
Map<Long, List<Item>> itensPorPedido = itens.stream()
    .collect(groupingBy(Item::getPedidoId));

for (Pedido p : pedidos) {
    List<Item> seus = itensPorPedido.get(p.id);
}
```

### 5. Logs em produção sem nível adequado

```java
log.debug("Detalhes: " + complexObject.toString()); // toString() roda mesmo em INFO
```

```java
// ✅ Lazy evaluation
log.debug("Detalhes: {}", complexObject); // só serializa se DEBUG ativo
```

---

## 🎯 Estratégias por Camada

### Aplicação
- Algoritmos eficientes (Big O importa)
- Caching agressivo (Redis, in-memory)
- Async / non-blocking I/O quando faz sentido
- Connection pooling

### Banco
- Índices certos
- Queries otimizadas (EXPLAIN)
- Particionamento de tabelas grandes
- Read replicas pra leituras pesadas

### Rede
- HTTP/2 ou HTTP/3
- Compressão (gzip, brotli)
- CDN
- Reduzir round-trips

### Cliente
- Bundle splitting
- Imagens otimizadas
- Lazy loading
- Service Workers (PWA)

---

## 🚀 Load Testing

Testar performance **antes** de produção.

### Ferramentas

| Ferramenta | Linguagem |
|---|---|
| **k6** | JavaScript |
| **Apache JMeter** | GUI Java |
| **Gatling** | Scala/Java |
| **Locust** | Python |
| **wrk** | CLI super simples |

### Exemplo: k6

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 100,         // 100 usuários virtuais
  duration: '30s',  // por 30 segundos
};

export default function () {
  const res = http.get('https://api.exemplo.com/usuarios');
  check(res, {
    'status 200': (r) => r.status === 200,
    'duration < 500ms': (r) => r.timings.duration < 500,
  });
}
```

```bash
k6 run script.js
```

### Tipos de teste

- **Load test**: carga normal esperada
- **Stress test**: aumenta até quebrar (encontrar limite)
- **Spike test**: pico súbito de tráfego
- **Soak test**: carga sustentada por horas (detecta memory leaks)

---

## ✅ Resumo da página

- **Meça antes de otimizar** — sempre
- Métricas chave: **latência (p50, p95, p99), throughput, saturação, erros**
- **Profiling** identifica gargalos: CPU, memória, I/O
- Em Java: **JFR + JMC** pra produção, VisualVM local
- **Banco** geralmente é o gargalo principal — use EXPLAIN, índices certos, evite N+1
- **Frontend**: Core Web Vitals (LCP, INP, CLS) afetam SEO e UX
- Anti-patterns: premature optimization, micro-otimizações, N+1, loops O(n²)
- **Load testing** antes de produção (k6, JMeter, Gatling)
- Otimização é trade-off: ganha velocidade, perde memória/complexidade

---

⬅️ [Anterior: Segurança](./22-seguranca.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Cloud](./24-cloud.md)
