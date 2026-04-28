# 📊 Estruturas de Dados e Algoritmos

> Saber qual estrutura usar pra cada problema é o que separa código que escala de código que trava com 10mil itens. Não é decoreba — é entender o **trade-off** de cada uma.

---

## 🎯 Por que importa?

Você pode resolver 99% dos problemas com `List` e `Map`. Mas quando vem o problema de performance, é aqui que você ganha 10x, 100x, 1000x:

- Buscar um item em 1 milhão de registros: 1 segundo ou 1 milissegundo?
- Verificar se item existe: percorrer tudo ou consultar direto?
- Ordenar 10 milhões de itens: minutos ou segundos?

> 💡 **Em palavras simples**: estrutura de dados é o **formato** em que você guarda informação. Algoritmo é o **passo a passo** pra fazer algo com ela. Os dois andam juntos.

---

## 📏 Big O: A Linguagem da Performance

**Big O** descreve como o tempo (ou memória) de um algoritmo cresce conforme a entrada cresce.

```
n = quantidade de itens

O(1)        → tempo constante (não importa o tamanho)
O(log n)    → cresce muito devagar (busca binária)
O(n)        → cresce linear (varre tudo uma vez)
O(n log n)  → cresce um pouco mais que linear (sort eficiente)
O(n²)       → quadrático (loop dentro de loop)
O(2^n)      → exponencial (fugir disso!)
```

### Comparação prática

Pra uma entrada de **1 milhão** de itens:

| Big O | Operações | Tempo aprox |
|---|---|---|
| O(1) | 1 | instantâneo |
| O(log n) | 20 | instantâneo |
| O(n) | 1.000.000 | ~10 ms |
| O(n log n) | 20.000.000 | ~200 ms |
| O(n²) | 1.000.000.000.000 | **~3 horas** |
| O(2^n) | 🤯 | universo morre antes |

> 🤔 **Por que isso importa?** Loops aninhados (`for` dentro de `for`) viram O(n²). Pra 1000 itens roda fácil. Pra 1 milhão, trava. Por isso você precisa **pensar em Big O** antes de codar.

---

## 📋 Estruturas Lineares

### Array / List

```java
List<Integer> lista = new ArrayList<>();
lista.add(10);          // O(1) amortizado
lista.get(0);           // O(1) — acesso por índice
lista.contains(10);     // O(n) — varre tudo
lista.remove(0);        // O(n) — desloca todos
```

✅ Acesso por índice rápido
❌ Buscar item ou inserir no meio é lento

### LinkedList

Cada elemento aponta pro próximo (e/ou anterior).

```java
LinkedList<Integer> lista = new LinkedList<>();
lista.addFirst(10);     // O(1)
lista.addLast(20);      // O(1)
lista.get(500);         // O(n) — precisa percorrer
```

✅ Inserção/remoção nas pontas é O(1)
❌ Acesso por índice é O(n)

### Stack (Pilha) — LIFO

Last In, First Out. O último que entra é o primeiro que sai.

```java
Deque<Integer> pilha = new ArrayDeque<>();
pilha.push(1);          // empilha
pilha.push(2);
pilha.pop();            // tira o 2 (o último)
```

**Usos**: histórico de undo, chamadas de função, parsers.

### Queue (Fila) — FIFO

First In, First Out. O primeiro que entra é o primeiro que sai.

```java
Queue<Integer> fila = new LinkedList<>();
fila.offer(1);          // adiciona no final
fila.offer(2);
fila.poll();            // tira o 1 (o primeiro)
```

**Usos**: processamento de tarefas, BFS em grafos, message brokers.

---

## 🗺️ Estruturas Associativas

### HashMap (Dicionário/Hash Table)

A estrutura mais útil que existe. Chave → valor com acesso O(1).

```java
Map<String, Integer> idades = new HashMap<>();
idades.put("Mauricio", 30);  // O(1)
idades.get("Mauricio");      // O(1)
idades.containsKey("Ana");   // O(1)
```

#### Como funciona internamente

1. Aplica função hash na chave → número
2. Usa esse número pra escolher um "bucket" (array interno)
3. Coloca o valor lá

> 💡 **Pior caso é O(n)** se muitas chaves caírem no mesmo bucket (colisões). Boas implementações lidam bem com isso (em Java 8+, buckets viram árvores quando ficam grandes).

### HashSet

Mesmo princípio, mas só guarda **chaves únicas** (sem valores).

```java
Set<String> visitados = new HashSet<>();
visitados.add("page1");
visitados.contains("page1");  // O(1)
```

**Uso clássico**: remover duplicatas, verificar se algo já foi processado.

### TreeMap / TreeSet

Versões **ordenadas** baseadas em árvores (geralmente Red-Black Tree).

```java
TreeMap<String, Integer> mapa = new TreeMap<>();
mapa.firstKey();        // O(log n) — menor chave
mapa.lastKey();         // O(log n) — maior chave
```

✅ Mantém ordenação automática
❌ Operações são O(log n) em vez de O(1)

> 🤔 **Quando usar TreeMap em vez de HashMap?** Quando você precisa **iterar em ordem** ou pegar mín/máx/range.

---

## 🌳 Árvores

### Binary Search Tree (BST)

Cada nó tem até 2 filhos. Esquerda < pai < direita.

```
        50
       /  \
      30   70
     / \   / \
    20 40 60 80
```

Busca, inserção, remoção: **O(log n)** se balanceada, O(n) no pior caso.

### Árvores balanceadas

- **Red-Black Tree**: usada em `TreeMap`, kernel Linux
- **AVL Tree**: balanceamento mais rigoroso
- **B-Tree / B+Tree**: usadas em **bancos de dados** e sistemas de arquivos

> 💡 É por isso que **índices de banco** (que vimos no [tópico 8](./08-banco-de-dados.md)) usam B-tree — busca em O(log n) em discos.

### Heap (Pilha de Prioridade)

Árvore binária onde o pai é sempre menor (min-heap) ou maior (max-heap) que os filhos.

```java
PriorityQueue<Integer> heap = new PriorityQueue<>();
heap.offer(5);
heap.offer(1);
heap.offer(3);
heap.poll();  // 1 (sempre o menor)
```

**Usos**: fila de prioridade, algoritmo de Dijkstra, scheduling.

### Trie

Árvore onde cada nó é uma letra. Eficiente pra autocomplete e dicionários.

```
root
 ├── c
 │   ├── a
 │   │   ├── r → "car"
 │   │   └── t → "cat"
 │   └── ...
 └── d
     └── o
         └── g → "dog"
```

---

## 🕸️ Grafos

Conjunto de **nós** conectados por **arestas**.

### Representações

**Lista de adjacência** (mais comum):
```java
Map<Integer, List<Integer>> grafo = new HashMap<>();
grafo.put(1, List.of(2, 3));
grafo.put(2, List.of(1, 4));
```

**Matriz de adjacência**:
```
   1 2 3 4
1 [0 1 1 0]
2 [1 0 0 1]
3 [1 0 0 0]
4 [0 1 0 0]
```

### Algoritmos clássicos

| Algoritmo | Pra que serve |
|---|---|
| **BFS** (Breadth-First Search) | Caminho mais curto em grafo sem peso |
| **DFS** (Depth-First Search) | Explorar tudo, detectar ciclos |
| **Dijkstra** | Caminho mais curto com pesos positivos |
| **A\*** | Caminho ótimo com heurística (jogos, GPS) |
| **Topological Sort** | Ordenar dependências (build systems) |

> 💡 **Onde grafos aparecem na vida real**: rotas de GPS, redes sociais (amigos = nós), dependências de pacotes (npm, maven), workflows.

---

## 🔄 Algoritmos de Ordenação

Não use na vida real (use `Collections.sort()`). Mas entender o conceito ajuda.

| Algoritmo | Tempo médio | Notas |
|---|---|---|
| **Bubble Sort** | O(n²) | Simples mas terrível |
| **Insertion Sort** | O(n²) | Bom pra listas pequenas ou quase ordenadas |
| **Quick Sort** | O(n log n) | Geralmente o mais rápido na prática |
| **Merge Sort** | O(n log n) | Estável, bom pra dados externos |
| **Heap Sort** | O(n log n) | In-place, sem pior caso |

> 🤔 **`Collections.sort()` em Java usa**: Timsort (variação de merge sort). Estável, otimizado pra dados parcialmente ordenados.

---

## 🔍 Busca Binária

Algoritmo fundamental. Funciona em **arrays ordenados**.

```java
int buscaBinaria(int[] arr, int alvo) {
    int low = 0, high = arr.length - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if (arr[mid] == alvo) return mid;
        if (arr[mid] < alvo) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```

A cada iteração, descarta metade. **O(log n)**.

> 💡 1 milhão de itens? Máximo 20 comparações.

---

## 🎯 Tabela: Que estrutura usar?

| Preciso de... | Use |
|---|---|
| Coleção ordenável por índice | `ArrayList` |
| Inserção/remoção rápida nas pontas | `LinkedList` ou `ArrayDeque` |
| Lookup rápido por chave | `HashMap` |
| Lookup ordenado | `TreeMap` |
| Itens únicos | `HashSet` |
| Itens únicos + ordem | `TreeSet` |
| Sempre pegar mín/máx | `PriorityQueue` (heap) |
| Última a entrar = primeira a sair | `ArrayDeque` (stack) |
| Primeira a entrar = primeira a sair | `LinkedList` (queue) |
| Autocomplete/prefixo | Trie |
| Relacionamentos entre entidades | Grafo |

---

## ✅ Resumo da página

- **Big O** descreve como tempo cresce com a entrada — fundamental pra entender performance
- **HashMap/HashSet** resolvem 80% dos problemas — O(1) lookup
- **Listas** são padrão; `LinkedList` só pra inserção nas pontas
- **TreeMap/TreeSet** mantêm ordem (O(log n))
- **PriorityQueue** (heap) sempre devolve mín/máx
- **Árvores balanceadas** estão por trás de DBs e várias estruturas
- **Grafos** modelam relacionamentos do mundo real
- Use **busca binária** em qualquer coleção ordenada
- Em Java: prefira `Collections.sort()` em vez de implementar sort

---

⬅️ [Anterior: Git](./12-git.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Design Patterns](./14-design-patterns.md)
