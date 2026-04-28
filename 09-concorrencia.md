# 🧵 Concorrência e Paralelismo

> CPUs modernas têm múltiplos núcleos. Programas precisam aproveitar isso. Mas multitarefa traz problemas novos: race conditions, deadlocks, bugs que só aparecem 1 vez em 1000.

---

## 🤔 Concorrência ≠ Paralelismo

A confusão é tão comum que vale começar daqui.

### Concorrência: lidar com várias coisas

Várias tarefas em **andamento simultâneo**, alternando entre elas. Pode ser num único núcleo.

> 💡 **Analogia**: um chef cozinhando 3 pratos. Ele coloca arroz no fogo, vai fritar a carne, volta pro arroz mexer, prepara o molho. Tarefas intercaladas, **mas só 1 mão por vez**.

### Paralelismo: fazer várias coisas ao mesmo tempo

Tarefas executando **literalmente simultaneamente**, em núcleos diferentes.

> 💡 **Analogia**: 3 chefs cozinhando 1 prato cada. **Verdadeiramente simultâneo**.

```
CONCORRÊNCIA (1 CPU):
Tarefa A: ──█──── ──█── ──█──────
Tarefa B: ──── ──█── ──── ──█────
Tarefa C: ──── ──── ──█── ──── ──█──

PARALELISMO (3 CPUs):
CPU 1 (Tarefa A): ████████████████
CPU 2 (Tarefa B): ████████████████
CPU 3 (Tarefa C): ████████████████
```

> 🤔 **Por que importa?** Em I/O bound (esperando rede, disco), **concorrência** já resolve — enquanto uma tarefa espera, outra trabalha. Em CPU bound (cálculo pesado), você precisa de **paralelismo real** pra ganhar performance.

---

## 🧱 Threads vs Processos

### 📦 Processo

- Programa em execução com **memória própria isolada**
- Pesado pra criar
- Não compartilha memória diretamente com outros processos

### 🧵 Thread

- Unidade de execução **dentro** de um processo
- Threads do mesmo processo **compartilham memória**
- Leves pra criar e trocar

> 💡 **Analogia**: o processo é uma casa. Threads são pessoas dentro da casa — todas compartilham geladeira, sofá, banheiro (memória). Já outros processos são outras casas — separadas.

```
PROCESSO 1                    PROCESSO 2
┌────────────────┐            ┌────────────────┐
│ Thread A       │            │ Thread X       │
│ Thread B       │            │ Thread Y       │
│ Thread C       │            │                │
│                │            │                │
│ MEMÓRIA        │            │ MEMÓRIA        │
│ COMPARTILHADA  │            │ COMPARTILHADA  │
└────────────────┘            └────────────────┘
       │                              │
       └──────────────────────────────┘
            (comunicação via IPC)
```

---

## ⚠️ Race Condition

O problema mais clássico de concorrência.

```java
// Variável compartilhada
int contador = 0;

// 2 threads rodando isso ao mesmo tempo:
contador = contador + 1;
```

Parece atômico, mas na CPU vira:

1. Ler valor atual de `contador` (registrador)
2. Somar 1
3. Escrever de volta na memória

Se duas threads executam isso simultaneamente:

```
Thread A: lê contador = 0
Thread B: lê contador = 0  (A ainda não escreveu!)
Thread A: escreve 1
Thread B: escreve 1  (deveria ser 2!)
```

**Resultado**: contador = 1 em vez de 2. Você "perdeu" um incremento.

> 🤔 **Por que esse bug é pesadelo?** Aparece raramente, depende de timing exato, é difícil reproduzir. Em código de produção que passa em todos os testes mas trava 1x por dia, geralmente é race condition.

---

## 🔒 Mutex e Locks

Solução: garantir que **só uma thread por vez** acesse a região crítica.

### Em Java

```java
// synchronized: lock automático
public synchronized void incrementar() {
    contador++;
}

// Lock explícito
private final Lock lock = new ReentrantLock();

public void incrementar() {
    lock.lock();
    try {
        contador++;
    } finally {
        lock.unlock();  // SEMPRE no finally
    }
}
```

### Operações atômicas

Pra casos simples, use classes atômicas (mais eficientes que lock):

```java
private final AtomicInteger contador = new AtomicInteger(0);

contador.incrementAndGet();  // thread-safe sem lock
```

---

## 💀 Deadlock

Duas threads esperam uma pela outra **pra sempre**:

```
Thread A: pegou lock 1, esperando lock 2
Thread B: pegou lock 2, esperando lock 1
```

Nenhuma libera. Programa congela.

> 💡 **Analogia**: dois carros num beco apertado, cada um esperando o outro recuar. Ninguém sai.

### Como evitar

- ✅ Sempre adquirir locks **na mesma ordem**
- ✅ Usar `tryLock` com timeout em vez de `lock` infinito
- ✅ Manter regiões críticas curtas
- ✅ Usar abstrações de mais alto nível (filas, executors)

---

## 🚀 Async/Await: Concorrência Sem Threads Diretas

Modelo moderno em JS, Python, C#: você escreve código que **parece síncrono**, mas que não bloqueia.

```javascript
// Antes: callback hell
fetch(url1, (data1) => {
    fetch(url2, (data2) => {
        fetch(url3, (data3) => {
            // ...
        });
    });
});

// Depois: async/await
const data1 = await fetch(url1);
const data2 = await fetch(url2);
const data3 = await fetch(url3);
```

### Como funciona em JS (Event Loop)

JavaScript é **single-threaded**, mas concorrente. O segredo:

```
┌─────────────────┐
│  Call Stack     │ ← Onde código JS roda
└────────┬────────┘
         ↓ tarefa async
┌─────────────────┐
│  Web APIs       │ ← fetch, setTimeout (rodam em outras threads)
└────────┬────────┘
         ↓ quando termina
┌─────────────────┐
│  Callback Queue │
└────────┬────────┘
         ↓ Event Loop puxa
    (Call Stack vazia)
```

> 💡 É por isso que `setTimeout(fn, 0)` não executa imediatamente — ele entra na fila e só roda quando a stack tá vazia.

### Em Java moderno

```java
// CompletableFuture (não bloqueia thread principal)
CompletableFuture.supplyAsync(() -> buscarUsuario())
    .thenApply(user -> buscarPedidos(user))
    .thenAccept(pedidos -> salvar(pedidos));

// Virtual Threads (Java 21+) - lightweight, milhares deles
Thread.startVirtualThread(() -> {
    // código que pode bloquear sem custo
});
```

---

## 🏗️ Padrões úteis (exemplos em Java)

> 💡 Os exemplos abaixo são em **Java**, mas conceitos análogos existem em qualquer linguagem com suporte a concorrência. **Node.js** usa `Promise.all`/`Promise.race`; **Python** tem `asyncio.gather` e `concurrent.futures`; **C#** tem `Task.WhenAll`; **Go** tem goroutines + channels.

### ExecutorService: Pool de Threads

Em vez de criar threads na mão, use um pool:

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

executor.submit(() -> {
    // tarefa rodando numa das 10 threads
});

executor.shutdown();
```

### CompletableFuture: Composição assíncrona

```java
CompletableFuture<Usuario> futureUser = CompletableFuture
    .supplyAsync(() -> buscarUsuario(id));

CompletableFuture<List<Pedido>> futurePedidos = CompletableFuture
    .supplyAsync(() -> buscarPedidos(id));

// Espera os dois e combina
futureUser.thenCombine(futurePedidos, (user, pedidos) ->
    new UsuarioComPedidos(user, pedidos)
);
```

### `@Async` no Spring

```java
@Service
public class EmailService {
    @Async
    public void enviarEmail(String dest) {
        // roda em thread separada do request
    }
}
```

> ⚠️ Precisa habilitar com `@EnableAsync` na classe de config.

---

## 📊 Padrões de Concorrência

| Padrão | Quando usar |
|---|---|
| **Producer-Consumer** | Uma thread produz, outra consome. Use `BlockingQueue` |
| **Worker Pool** | Várias threads pegam tarefas de uma fila |
| **Scatter-Gather** | Espalha trabalho em paralelo, junta resultados |
| **Pipeline** | Estágios sequenciais, cada um em thread própria |

---

## ✅ Resumo da página

- **Concorrência** (alternar tarefas) ≠ **Paralelismo** (fazer ao mesmo tempo)
- **Threads** compartilham memória do processo; **processos** são isolados
- **Race conditions** acontecem quando duas threads acessam variável compartilhada sem sincronização
- **Mutex/Locks** protegem regiões críticas; cuidado com **deadlocks**
- Operações **atômicas** (`AtomicInteger`) são mais eficientes que locks pra casos simples
- **Async/await** dá concorrência sem gerenciar threads diretamente
- Em Java: `ExecutorService`, `CompletableFuture`, `@Async`, e Virtual Threads (Java 21+)
- Bugs de concorrência são **difíceis de reproduzir** — desconfie de código compartilhado

---

⬅️ [Anterior: Banco de Dados](./08-banco-de-dados.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Criptografia](./10-criptografia.md)
