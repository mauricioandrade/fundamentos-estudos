# 🧠 Memória e Execução de Programas

> Entender como a memória RAM é organizada explica muita coisa: stack overflow, memory leak, segmentation fault, garbage collection. Tudo conectado.

---

## 🌍 O Problema: Vários Programas, Uma RAM

Você tem 16 GB de RAM e 30 programas abertos. Como cada um sabe onde "morar" sem invadir a memória do outro?

> 💡 **Em palavras simples**: imagine um prédio com vários moradores. Sem regras, todo mundo brigaria pelos quartos. Precisa de um síndico (o Sistema Operacional) que dá pra cada morador um apartamento isolado.

---

## 🏠 Memória Virtual: Cada Processo no Seu Mundo

Cada programa **acha** que tem a RAM inteira só pra ele. Tipo uma matrix.

- O programa usa "endereços virtuais" (ex: `0x00400000`)
- Um chip dentro da CPU chamado **MMU** (Memory Management Unit) traduz esses endereços virtuais para **endereços físicos** reais na RAM
- O Sistema Operacional decide qual região física vai pra qual programa

### 📑 Paginação

A memória é dividida em **páginas** de tamanho fixo (geralmente 4 KB).

- Quando a RAM enche, o SO pega páginas que estão sem uso e **manda pro disco** (área chamada **swap**)
- Se o programa precisar dessa página depois, o SO traz de volta — chamado **page fault**

> 🤔 **Por que devs sentem isso?** Quando o sistema está com pouca RAM e começa a swappar muito, tudo fica lentíssimo. Isso se chama **thrashing** — o computador passa mais tempo movendo páginas do que executando código.

### ⚡ TLB

A **TLB** (Translation Lookaside Buffer) é uma cachezinha dentro da CPU que guarda traduções recentes de endereço virtual → físico. Sem ela, cada acesso à memória precisaria consultar tabelas inteiras.

---

## 📦 Como um Programa Vira Processo

Quando você dá duplo clique num app, o que acontece?

```
1. Disco: arquivo executável (binário)
        ↓
2. Loader (parte do SO) lê o arquivo
        ↓
3. Carrega na RAM em regiões específicas
        ↓
4. Salta pro "entry point" (primeira instrução)
        ↓
5. Programa começa a rodar
```

### 📂 Formato ELF (no Linux)

No Linux, executáveis usam o formato **ELF** (Executable and Linkable Format). Ele divide o programa em **seções** com permissões diferentes:

| Seção | O que tem | Permissões |
|---|---|---|
| `.text` | Código (instruções) | 🔒 Read + Execute (não pode escrever!) |
| `.rodata` | Constantes (`"olá mundo"`, `PI = 3.14`) | 🔒 Read-only |
| `.data` | Variáveis globais com valor inicial | Read + Write |
| `.bss` | Variáveis globais sem valor inicial | Read + Write |

> 💡 **Curiosidade**: você pode inspecionar um binário Linux com `readelf -SW <arquivo>` e ver todas as seções.

> ⚠️ Tentar **escrever** na seção `.text` causa **Segmentation Fault** — é proteção pra evitar que vírus injetem código.

> 📌 No Windows, o formato é **PE** (Portable Executable). No macOS, é **Mach-O**. Mesma ideia, sintaxe diferente.

---

## 🧱 Stack vs Heap: Os Dois Cantos da Memória

Quando seu programa está rodando, as variáveis precisam morar em algum lugar. Existem dois "bairros" distintos: **Stack** e **Heap**.

### 🥞 Stack (Pilha): Rápida, rígida, automática

- Estrutura **LIFO** (Last In, First Out — última a entrar, primeira a sair)
- Cada vez que você chama uma **função**, é criado um "frame" na stack com:
  - Os parâmetros
  - As variáveis locais
  - O endereço pra onde voltar quando a função acabar
- Quando a função retorna, o frame é **descartado automaticamente**
- Tamanhos precisam ser **conhecidos em tempo de compilação**

> 💡 **Em palavras simples**: a stack é tipo uma pilha de pratos. Você só pega o de cima. Cada chamada de função empilha um prato; cada `return` tira um prato.

> ⚠️ **Stack Overflow**: se você fizer recursão sem fim, a pilha enche e o programa morre. É literalmente o que dá nome ao site.

### 📦 Heap (Monte): Vasta, flexível, manual

- Usada para dados que:
  - **Têm tamanho dinâmico** (lista que cresce, string que muda)
  - **Precisam viver além da função** que os criou
- Você **pede** memória ao SO em runtime (`new` em Java/C#, `malloc` em C, `[]` em Python)
- O SO te dá um **endereço** — esse endereço fica guardado numa variável da stack (chamada **ponteiro** ou **referência**)

```
STACK                    HEAP
┌──────────────┐        ┌─────────────────────┐
│ x = 10       │        │                     │
│ y = "oi"     │        │   [1, 2, 3, 4, 5]   │
│ lista → ─────┼───────→│                     │
│              │        │                     │
└──────────────┘        └─────────────────────┘
```

### 🔗 Aliasing: O Pulo do Gato

Se duas variáveis na stack apontam pro **mesmo objeto** na heap, mudar uma afeta a outra:

```javascript
// JavaScript / Java / Python / C# — comportamento padrão
let a = [1, 2, 3];
let b = a;        // b aponta pro MESMO array que a
b.push(4);
console.log(a);   // [1, 2, 3, 4] !!!
```

> 🤔 **Por que isso importa?** Bugs por aliasing são clássicos. Em linguagens como C# e Java, **objetos são sempre passados por referência**. Em C/C++, você controla isso manualmente com `*` e `&`.

### 🗑️ Garbage Collection (GC)

Em linguagens como **Java, C#, Python, JavaScript**, você não precisa liberar memória manualmente — um sistema chamado **Garbage Collector** roda em background e libera objetos sem referências.

- ✅ Vantagem: sem memory leaks bobos, sem `free()` esquecido
- ❌ Desvantagem: pausas imprevisíveis ("GC pause"), uso de CPU extra

> 📌 **Detalhe específico do .NET**: objetos maiores que 85.000 bytes vão pro **Large Object Heap (LOH)**, que não é compactado por padrão. Isso pode causar fragmentação em apps que rodam por muito tempo.

> 📌 Linguagens **sem GC** (C, C++, Rust): você é responsável por liberar. Em C/C++ é manual (`free`, `delete`); em Rust o compilador resolve em tempo de compilação via **ownership**.

---

## ✅ Resumo da página

- Cada programa tem uma **visão isolada** da memória (memória virtual)
- O SO usa **paginação** e **swap** pra caber tudo na RAM
- Programas binários (formato **ELF** no Linux, **PE** no Windows, **Mach-O** no macOS) são divididos em seções com permissões
- **Stack**: rápida, automática, pra coisa pequena e local de função
- **Heap**: flexível, dinâmica, pra dados grandes e duradouros
- **GC** existe em linguagens gerenciadas; em C/C++ você controla manualmente

---

⬅️ [Anterior: Processador](./02-processador.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Sistema Operacional](./04-sistema-operacional.md)
