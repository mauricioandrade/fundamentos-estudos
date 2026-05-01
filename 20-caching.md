# ⚡ Caching: Acelerando Aplicações

> Cache é a forma mais rápida de **acelerar** uma aplicação. Em vez de buscar dado caro toda vez (banco, API externa), guarda o resultado em memória e devolve quase instantaneamente. Mas tem um ditado famoso: "só existem dois problemas difíceis em ciência da computação: cache invalidation e nomear coisas". 🤣

---

## 🤔 O Que é Cache?

Armazenamento **temporário** de dados frequentemente acessados, em local **mais rápido** que a fonte original.

### Analogia

Você lê um livro pesquisando algo. Em vez de ir na biblioteca toda vez (lento), você guarda o livro na sua mesa (rápido). Quando precisar de novo, está ali.

### Onde cache aparece (já está em todo lugar)

```
Browser (cache de imagens) → CDN → Reverse Proxy (Nginx)
   → Application Cache (Redis) → Database Cache → CPU L1/L2/L3
```

Cada camada acelera a próxima. O dado mais "quente" fica mais perto do usuário.

---

## 📊 Tipos de Cache

### 💾 Em memória (in-process)

Dado guardado **dentro** do processo da aplicação. Super rápido (acesso de RAM local).

```java
// Java + Caffeine
Cache<String, Usuario> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build();

cache.put("user:1", usuario);
Usuario u = cache.getIfPresent("user:1");
```

✅ Latência **mínima** (nanosegundos)
❌ Não compartilhado entre instâncias (cada réplica tem o seu)
❌ Some quando reinicia

### 🌐 Distribuído (out-of-process)

Cache **separado**, compartilhado entre todas as instâncias da aplicação.

```
   App #1 ──┐
   App #2 ──┼── Redis (cache compartilhado)
   App #3 ──┘
```

✅ Compartilhado entre instâncias
✅ Sobrevive a restarts
❌ Latência maior (roundtrip de rede ~1ms)

**Mais usado**: **Redis**, Memcached, Hazelcast.

### 🌍 CDN (Content Delivery Network)

Cache de **assets estáticos** distribuído globalmente. Já vimos no [tópico de Redes](./07-redes.md).

```
Usuário no Brasil → CDN em São Paulo (cache hit, 5ms)
                    └─→ Origem nos EUA (somente se cache miss, 200ms)
```

---

## 🔴 Redis: O Padrão da Indústria

Redis é um **banco de dados em memória** que serve perfeitamente como cache. Mas é mais que isso — é uma "Swiss Army knife" pra dados em memória.

### Estruturas de dados suportadas

| Estrutura | Uso típico |
|---|---|
| **String** | Cache simples chave-valor |
| **Hash** | Objetos com múltiplos campos |
| **List** | Filas, timelines |
| **Set** | Tags únicas, membros de grupos |
| **Sorted Set** | Rankings, leaderboards |
| **Stream** | Logs de eventos |
| **HyperLogLog** | Contagem aproximada |

### Comandos básicos

```bash
# Strings
SET user:123 "Mauricio"
GET user:123
EXPIRE user:123 60        # expira em 60 segundos
SET user:123 "Mauricio" EX 60   # set + expire em 1 comando

# Hashes
HSET user:123 nome "Mauricio" idade 30
HGET user:123 nome
HGETALL user:123

# Sorted Set (ranking)
ZADD ranking 100 "alice"
ZADD ranking 250 "bob"
ZRANGE ranking 0 -1 WITHSCORES   # top scores

# Verificação rápida
EXISTS user:123
DEL user:123
```

### Em Spring Boot

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```java
@Service
public class UsuarioService {
    private final RedisTemplate<String, Usuario> redis;

    public Usuario buscar(Long id) {
        String key = "user:" + id;

        // Tenta no cache primeiro
        Usuario cached = redis.opsForValue().get(key);
        if (cached != null) return cached;

        // Cache miss: busca no banco
        Usuario u = repo.findById(id).orElseThrow();

        // Salva no cache (10 min)
        redis.opsForValue().set(key, u, Duration.ofMinutes(10));

        return u;
    }
}
```

### Anotações do Spring Cache (mais elegante)

```java
@Service
public class UsuarioService {
    @Cacheable(value = "usuarios", key = "#id")
    public Usuario buscar(Long id) {
        return repo.findById(id).orElseThrow();
    }

    @CacheEvict(value = "usuarios", key = "#id")
    public void atualizar(Long id, Usuario u) {
        repo.save(u);
    }
}
```

---

## 🎯 Padrões de Caching

### 1️⃣ Cache-Aside (Lazy Loading)

A app **gerencia** o cache manualmente.

```
1. App pede dado
2. Verifica no cache
3. Se MISS, busca no DB e popula o cache
4. Retorna
```

✅ Simples, controle total
❌ Primeira request sempre mais lenta (cache miss)

### 2️⃣ Read-Through

Cache **busca sozinho** no DB se não tiver.

```
App → Cache (que busca no DB se necessário)
```

✅ App não conhece DB
❌ Requer biblioteca/middleware suportando

### 3️⃣ Write-Through

Toda escrita vai pra **cache E DB simultaneamente**.

```
Write → Cache → DB (síncrono)
```

✅ Cache sempre consistente
❌ Escritas mais lentas

### 4️⃣ Write-Behind (Write-Back)

Escrita vai pro **cache primeiro**; DB é atualizado **assíncrono**.

```
Write → Cache (rápido) → DB (depois, async)
```

✅ Escritas rapidíssimas
❌ Risco de perda se cache cair

### 5️⃣ Refresh-Ahead

Cache se **renova sozinho** antes de expirar.

```
TTL: 10min, refresh-ahead em 8min
8min: pré-carrega versão nova em background
10min: cache expira (mas já tem versão atualizada)
```

✅ Evita cache miss em itens populares
❌ Mais complexo de implementar

---

## ⏰ TTL: Tempo de Vida

Toda chave deve ter um **TTL** (Time To Live). Sem TTL, o cache cresce até estourar memória.

### Estratégias de TTL

| Estratégia | Quando usar |
|---|---|
| **TTL fixo** | Maioria dos casos (5min, 1h, 1 dia) |
| **TTL randomizado** | Evita "thundering herd" (todos expirando juntos) |
| **TTL baseado em modificação** | Atualiza quando o dado real muda |

```java
// TTL randomizado pra evitar thundering herd
int baseTTL = 600;  // 10 min
int jitter = ThreadLocalRandom.current().nextInt(60);  // ±60s
redis.opsForValue().set(key, value, Duration.ofSeconds(baseTTL + jitter));
```

> 🤔 **Thundering herd**: 10.000 requests batem no banco simultaneamente porque todos os caches expiraram ao mesmo tempo. Banco morre.

---

## 💀 Problemas Clássicos

### 1. Cache Stampede

Cache expira → 1000 requests batem no DB simultaneamente.

**Solução**: lock + apenas 1 thread renova:

```java
public Usuario buscar(Long id) {
    Usuario cached = cache.get(id);
    if (cached != null) return cached;

    synchronized (lock(id)) {
        cached = cache.get(id);  // double-check
        if (cached != null) return cached;

        Usuario u = repo.findById(id).orElseThrow();
        cache.put(id, u);
        return u;
    }
}
```

### 2. Cache Penetration

Atacante consulta IDs que **não existem**, batendo no DB toda vez.

**Solução**: cachear `null` também (com TTL menor):

```java
if (cached != null) {
    return cached.equals("NULL_MARKER") ? null : cached;
}
Usuario u = repo.findById(id).orElse(null);
cache.put(id, u != null ? u : "NULL_MARKER", Duration.ofMinutes(1));
```

### 3. Cache Invalidation

O dado mudou no DB mas o cache ainda tem o antigo. **Como invalidar?**

Estratégias:
- ✏️ **Write-through**: atualiza cache na mesma operação
- 🗑️ **Invalidate on write**: deleta a chave; próxima leitura recarrega
- ⏰ **TTL curto**: aceita inconsistência de até X segundos
- 📡 **Pub/sub**: outros serviços avisam de mudanças

> 💡 **A regra de ouro**: o cache mais simples de invalidar é o que tem **TTL pequeno**. "Inconsistência por 30 segundos é aceitável?" Sim → use TTL.

---

## 🎯 O que cachear?

### ✅ Bom pra cachear

- Dados **lidos muito mais que escritos** (catálogo de produtos, perfil de usuário)
- Resultados de queries **caras** (relatórios, agregações)
- Dados **imutáveis** ou que mudam raramente (configurações, listas de cidades)
- Tokens, sessões
- Fragments de HTML (Server-Side Cache)

### ❌ Ruim pra cachear

- Dados que mudam **a todo segundo** (estoque tempo-real, cotação de ações)
- Dados **pequenos e baratos** (não vale a complexidade)
- Dados **sensíveis** (cuidado com cache de dados pessoais)
- Dados **únicos por usuário** se o universo de usuários for enorme

---

## 🔍 Métricas Importantes

| Métrica | Significa |
|---|---|
| **Hit Rate** | % de requests servidos do cache |
| **Miss Rate** | % que precisou ir ao DB |
| **Eviction Rate** | Itens removidos por falta de espaço |
| **Avg Latency** | Tempo médio de leitura |

> 🎯 **Meta razoável**: hit rate > 80% pra que o cache "compense".

```bash
# Redis stats
redis-cli INFO stats | grep -E "keyspace_hits|keyspace_misses"
```

---

## ✅ Resumo da página

- **Cache** acelera apps guardando dados em locais mais rápidos
- **In-memory** é mais rápido; **distribuído (Redis)** é compartilhado
- **Redis** suporta strings, hashes, sets, sorted sets, streams
- Padrões: **cache-aside, read-through, write-through, write-behind, refresh-ahead**
- Sempre usa **TTL**; randomize pra evitar thundering herd
- **Cache invalidation** é difícil — TTL curto é o atalho
- **Cache stampede, penetration, invalidation** — conheça e mitigue
- **Hit rate > 80%** = cache compensando

---

⬅️ [Anterior: Mensageria](./19-mensageria.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: APIs Avançadas](./21-apis-avancadas.md)
