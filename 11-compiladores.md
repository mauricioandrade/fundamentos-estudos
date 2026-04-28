# 🛠️ Compiladores: Como o Código Vira Código

> Você escreve `if (x > 0) {...}` e magicamente vira instruções binárias da CPU. Quem faz essa mágica é o **compilador**. Entender o processo te ajuda a escrever código melhor, debugar problemas estranhos e até criar suas próprias DSLs.

---

## 🎯 O Problema

Como transformar texto humano em código de máquina?

```
"if (idade > 18) { liberado = true; }"
                  ↓
        ?? alguma mágica ??
                  ↓
   10110000 01100001 11001010 ...
```

A resposta: **não é uma mágica única**, é uma **pipeline de etapas** bem definidas.

> 💡 **Compilador é só um programa que lê outro programa e cospe outro programa.** O input dele é texto; o output é (geralmente) binário ou bytecode.

---

## 🏭 O Pipeline de Compilação

```
Código fonte
    ↓
[1. Lexer]        → tokens
    ↓
[2. Parser]       → AST (Abstract Syntax Tree)
    ↓
[3. Análise Semântica]  → AST + checagem de tipos
    ↓
[4. Otimização]   → AST otimizada / IR
    ↓
[5. Geração de código]  → assembly / bytecode / binário
```

Vamos por partes.

---

## 🔤 1. Lexer (Análise Léxica)

Transforma a string bruta em **tokens** — pedacinhos com significado.

### Entrada
```c
if (idade > 18) {
    liberado = true;
}
```

### Saída do Lexer

| Token | Tipo |
|---|---|
| `if` | KEYWORD |
| `(` | PAREN_OPEN |
| `idade` | IDENTIFIER |
| `>` | OPERATOR |
| `18` | NUMBER |
| `)` | PAREN_CLOSE |
| `{` | BRACE_OPEN |
| `liberado` | IDENTIFIER |
| `=` | OPERATOR |
| `true` | KEYWORD |
| `;` | SEMICOLON |
| `}` | BRACE_CLOSE |

> 💡 **Em palavras simples**: o lexer ignora espaços e comentários, e categoriza cada "palavra" do código.

### Erros típicos detectados aqui

```c
int x = 5@;  // erro: '@' não é um token válido
"texto sem fim  // erro: string não fechada
```

---

## 🌳 2. Parser (Análise Sintática)

Pega os tokens e monta uma **árvore sintática abstrata** (AST). Verifica se a estrutura faz sentido **gramaticalmente**.

### Entrada (tokens)
```
if ( idade > 18 ) { liberado = true ; }
```

### Saída (AST)

```
        IfStatement
       /            \
   Condition       Body
      |              |
   BinaryOp     Assignment
   />  18      /          \
   |       liberado     true
  idade
```

### Erros típicos

```c
if (idade > 18 {       // erro: faltou ')'
    liberado = true;
}

x = ;                  // erro: '=' precisa de valor
```

> 🤔 **Por que AST e não só uma lista plana?** Porque programas têm **estrutura aninhada**. `if` dentro de `for` dentro de função. Árvore representa essa hierarquia naturalmente.

---

## 🔍 3. Análise Semântica

Sintaxe correta não significa código válido. Aqui o compilador verifica **significado**:

```java
String nome = 42;          // erro: tipo incompatível
listaDeClientes.adicionar(produto);  // produto não é Cliente
chamarFuncaoQueNaoExiste();  // erro: símbolo não definido
int x = y + 1;             // erro: y não foi declarado
```

### O que essa fase faz

- 🔤 **Resolução de nomes**: cada variável aponta pra qual declaração?
- 🏷️ **Type checking**: tipos batem nas operações?
- 🔁 **Escopo**: a variável existe nesse contexto?
- 📋 **Tabela de símbolos**: registro de tudo que foi declarado

> 💡 **É aqui que linguagens estáticas (Java, Rust) brilham.** Elas pegam erros que só apareceriam em runtime em linguagens dinâmicas (Python, JS clássico).

---

## ⚡ 4. Otimização

Compiladores modernos são **muito espertos**. Eles reescrevem seu código pra ser mais rápido **sem mudar o comportamento**.

### Exemplos de otimizações

#### Constant folding
```c
int x = 5 * 10;           // vira: int x = 50;
```

#### Dead code elimination
```c
if (false) {
    fazerCoisa();         // removido completamente
}
```

#### Loop unrolling
```c
for (int i = 0; i < 4; i++) sum += arr[i];

// vira algo como:
sum += arr[0];
sum += arr[1];
sum += arr[2];
sum += arr[3];
```

#### Inlining
```c
int dobrar(int x) { return x * 2; }
int y = dobrar(5);

// vira:
int y = 5 * 2;  // ou direto: int y = 10;
```

> 🤔 **Por que isso importa pra dev?** Porque às vezes você "otimiza manualmente" código e o compilador já fazia melhor. Escreva código **claro**; deixe o compilador otimizar.

### IR (Intermediate Representation)

Antes de gerar o assembly final, compiladores usam uma **representação intermediária** — código mais simples que a linguagem fonte mas mais alto que o assembly. LLVM IR é o exemplo mais famoso.

```
Linguagem fonte (C, Rust, Swift) → LLVM IR → Assembly (x86, ARM, RISC-V)
```

> 💡 **É por isso que Rust, Swift e C++ moderno geram código quase tão rápido quanto C** — todos usam LLVM como backend.

---

## 🎯 5. Geração de Código

Última etapa: traduzir a IR otimizada para o formato alvo.

### Possíveis alvos

| Alvo | Quando |
|---|---|
| **Assembly nativo** (x86, ARM) | C, C++, Rust, Go |
| **Bytecode** (JVM, CLR) | Java, Kotlin, C#, F# |
| **Outra linguagem** | TypeScript → JS, CoffeeScript → JS |
| **WebAssembly** | Rust, C++, Go pra rodar no browser |

### Exemplo simples (pseudo-assembly x86)

```c
int x = a + b;
```

Vira algo tipo:
```asm
mov eax, [a]      ; carrega a no registrador eax
mov ebx, [b]      ; carrega b no ebx
add eax, ebx      ; soma
mov [x], eax      ; salva em x
```

---

## 🔥 Compilador vs Interpretador vs JIT

Já vimos no [tópico de Linguagens](./05-linguagens.md), mas vale revisar com a ótica de compiladores:

### Compilador clássico (AOT - Ahead Of Time)
- Faz **todas** as etapas antes de rodar
- Output: arquivo binário pronto
- Ex: GCC, Clang, Rustc

### Interpretador
- Faz lexer + parser, mas **executa direto a AST** (ou bytecode)
- Sem geração de código nativo
- Ex: Python clássico, Ruby

### JIT (Just-In-Time)
- Híbrido: começa interpretando, mas detecta hot paths e **compila para nativo em runtime**
- Otimiza com base em **dados reais de execução**
- Ex: JVM HotSpot, V8 (Chrome/Node), .NET CLR

> 🤔 **JIT pode ser mais rápido que AOT?** Em alguns casos sim! O JIT vê quais branches são realmente tomadas em produção e otimiza pra isso. Compilador AOT só pode adivinhar.

---

## 🛠️ Ferramentas Pra Brincar

### Ver o assembly gerado pelo seu C

```bash
# Compila e mostra o assembly
gcc -S -O2 programa.c -o programa.s
cat programa.s
```

### [godbolt.org](https://godbolt.org)

Site fenomenal pra explorar. Cole código C/C++/Rust/Java e veja o assembly em tempo real, com diferentes compiladores e flags de otimização.

### Ver bytecode Java

```bash
javac MeuPrograma.java
javap -c MeuPrograma.class
```

Você vê instruções tipo:
```
iload_1       // carrega int do slot 1
iconst_5      // empilha constante 5
imul          // multiplica
istore_2      // salva em slot 2
```

---

## 🧪 Quando Você Pode Querer Criar Um Compilador

Não precisa virar dev de compilador. Mas alguns casos onde o conhecimento ajuda:

- 📝 **DSL** (Domain-Specific Language): linguagem própria pra problema específico (ex: queries customizadas, regras de negócio)
- 🔧 **Transpilador**: TypeScript → JS, JSX → JS
- 📐 **Templates engines**: Handlebars, Thymeleaf — são mini-compiladores
- 🎯 **Linters/formatters**: ESLint, Prettier — usam parser + análise
- 🧬 **Geração de código**: a partir de schema (OpenAPI → cliente HTTP)

### Bibliotecas úteis

| Linguagem | Bibliotecas |
|---|---|
| **JavaScript** | Babel, ANTLR, PEG.js |
| **Java** | ANTLR, JavaCC |
| **Python** | PLY, Lark, ANTLR |
| **Rust** | `nom`, `pest`, `lalrpop` |

---

## ✅ Resumo da página

- Compilação é uma **pipeline**: Lexer → Parser → Semântica → Otimização → Geração
- **Lexer** quebra texto em tokens; **parser** monta AST
- **Análise semântica** pega erros de tipo e escopo (linguagens estáticas brilham aqui)
- **Otimizações** modernas são extremamente avançadas — escreva código claro
- **LLVM IR** é o backend de várias linguagens modernas (Rust, Swift, C++)
- **JIT** pode superar AOT em alguns casos por usar dados de runtime
- Ferramentas como **godbolt.org** e `javap -c` te ajudam a entender o que seu código vira
- Criar mini-compiladores é útil pra DSLs, transpiladores, linters e geração de código

---

⬅️ [Anterior: Criptografia](./10-criptografia.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Git](./12-git.md)
