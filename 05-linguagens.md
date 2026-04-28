# 📝 Linguagens de Programação: Como o Código Vira Execução

> Você escreve `console.log("oi")`. A CPU só entende zeros e uns. Quem faz a tradução? Depende da **linguagem** — e existem 3 estratégias diferentes.

---

## 🎯 O Problema Central

Humanos pensam em palavras (`if`, `for`, `function`). CPUs pensam em **opcodes binários**:

```
Seu código:           Java/JS/Python
                          ↓
                   ?? (alguém traduz) ??
                          ↓
CPU executa:        10110000 01100001
```

Quem é esse "alguém"? É aí que as linguagens se diferem.

---

## 🔨 1. Linguagens Compiladas (Estáticas)

### Como funciona

Antes de você executar o programa, um **Compilador** lê **todo** o código e gera um **arquivo binário** (`.exe`, `.out`, ELF) já em linguagem de máquina.

```
código.c → [Compilador] → programa.exe → CPU roda direto
                                ↑
                    (já tá em binário, sem intermediário)
```

### Exemplos: **C, C++, Rust, Go**

### ✅ Vantagens

- **Performance máxima**: a CPU lê direto, sem intérpretes no caminho
- **Otimização agressiva**: o compilador analisa o código todo e otimiza
- **Código fonte protegido**: o binário não é legível (sem reverse engineering pesado)

### ❌ Desvantagens

- **Compilação lenta**: cada mudança exige recompilar
- **Não é portável**: um `.exe` compilado pra Windows x86 não roda em Mac M1 (ARM). Precisa **recompilar** pra cada plataforma
- **Mais difícil de debugar** (o código rodando não é o que você escreveu, é a versão otimizada)

> 💡 **Por que servidores de alta performance usam Go ou Rust?** Justamente pela velocidade nativa. Se você precisa processar 1 milhão de requisições por segundo, cada nanossegundo conta.

---

## 🎵 2. Linguagens Interpretadas (Dinâmicas)

### Como funciona

Existe um programa intermediário (o **interpretador**) que **lê e executa o código texto linha por linha** em tempo real.

```
código.py → [Interpretador Python lê e executa cada linha] → CPU
                              ↑
                    (faz isso TODA vez que você roda)
```

### Exemplos: **Python, Ruby, PHP clássico, JavaScript antigo**

### ✅ Vantagens

- **Portabilidade total**: o mesmo `.py` roda em Windows, Mac, Linux, ARM, x86 — desde que tenha o interpretador instalado
- **Iteração rápida**: mudou o código? Só rodar de novo. Sem compilação
- **Flexibilidade dinâmica**: tipos podem mudar em runtime, código pode gerar código

### ❌ Desvantagens

- **Performance inferior**: a tradução acontece toda hora, gasta CPU
- **Loops pesados são lentos** (relativamente)
- **Erros aparecem só em runtime**: erro de sintaxe numa linha que nunca executa? Você só descobre quando ela for executada

> 🤔 **Curiosidade**: por isso Python é tão usado em ciência de dados — não pelo Python ser rápido (não é), mas porque as **bibliotecas críticas (NumPy, Pandas)** são escritas em **C**. Python só "orquestra".

---

## 🔀 3. Linguagens Híbridas: Bytecode + JIT

### Como funciona

Tenta pegar o melhor dos dois mundos. O processo tem **3 etapas**:

```
1. código.java → [Compilador javac] → código.class (BYTECODE)
                                            ↓
2. Você roda → [Máquina Virtual (JVM)] interpreta o bytecode
                                            ↓
3. JIT detecta trechos quentes → compila pra nativo em RUNTIME → CPU
```

### Exemplos: **Java, C#, Kotlin, PHP 8+, JavaScript moderno (V8)**

### Os elementos-chave

#### 📦 Bytecode

Um formato **intermediário e portável**. Não é binário de máquina, mas também não é texto. É um "meio do caminho" otimizado.

#### 🖥️ Máquina Virtual (VM)

- **JVM** para Java/Kotlin
- **CLR** para C#/.NET
- **V8** para JavaScript moderno (Node.js, Chrome)

A VM lê o bytecode e executa. Se você tem a VM correta, **o mesmo bytecode roda em qualquer plataforma** ("Write Once, Run Anywhere").

#### ⚡ JIT (Just-In-Time Compiler)

A mágica. A VM **observa** quais partes do código rodam mais (hot paths) e, em runtime, **compila esses trechos pra código nativo de máquina**. Da segunda vez que você chama essa função, ela voa.

> 💡 **É por isso que apps Java demoram pra "esquentar"**: nos primeiros segundos, está tudo sendo interpretado. Depois que o JIT otimiza os hot paths, a performance dispara.

### ✅ Vantagens

- **Portabilidade**: bytecode roda onde tiver VM
- **Performance próxima de nativo**: depois do warm-up, JIT é quase tão rápido quanto C
- **Instrumentação dinâmica**: você pode plugar profilers, agentes, hot reload sem recompilar tudo

### ❌ Desvantagens

- **Dependência da VM**: o usuário precisa ter Java instalado, .NET runtime, etc.
- **Tempo de warm-up**: as primeiras execuções são lentas
- **Maior consumo de RAM**: a VM em si ocupa memória

---

## 📊 Tabela Comparativa

| Aspecto | Compilada | Interpretada | Híbrida (JIT) |
|---|---|---|---|
| **Quando traduz** | Antes de rodar | Durante a execução | Compila pra bytecode antes; JIT em runtime |
| **Velocidade** | 🔥🔥🔥 Máxima | 🔥 Lenta | 🔥🔥 Quase nativa (após warm-up) |
| **Portabilidade** | ❌ Baixa (recompila por SO/arch) | ✅ Alta (qualquer plataforma com interpretador) | ✅ Alta (qualquer plataforma com VM) |
| **Iteração no dev** | 🐢 Lenta | 🚀 Rápida | 🚀 Rápida |
| **Uso de memória** | Baixo | Médio | Alto (VM + bytecode + JIT) |
| **Exemplos** | C, C++, Rust, Go | Python, Ruby, PHP clássico | Java, C#, JS moderno, PHP 8+ |

---

## 🤔 Como escolher uma linguagem?

Não tem resposta única, mas:

| Vou fazer... | Considere... |
|---|---|
| Sistema operacional, driver | C, Rust |
| Game AAA, motor de jogo | C++ |
| Servidor web de alta performance | Go, Rust, Java |
| Backend corporativo | **Java, C#** (ecossistema maduro) |
| API rápida e simples | Node.js, Python (FastAPI) |
| Ciência de dados, ML | Python (com NumPy/PyTorch em C) |
| Frontend web | JavaScript/TypeScript |
| Mobile nativo | Swift (iOS), Kotlin (Android) |
| Mobile multiplataforma | Flutter (Dart), React Native |
| Script de automação | Python, Bash |

> 💡 **Verdade prática**: a maioria dos sistemas modernos é **poliglota** — usa várias linguagens em camadas diferentes. Um app web típico tem JS no frontend, Java/Python/Go no backend, SQL no banco.

---

## ✅ Resumo da página

- **Compiladas** (C, Rust, Go): traduz tudo antes, gera binário rápido mas amarrado a uma plataforma
- **Interpretadas** (Python, Ruby): traduz linha por linha em runtime, lentas mas portáveis
- **Híbridas** (Java, C#, JS moderno): bytecode + JIT — combinam portabilidade com performance
- A escolha da linguagem depende do **trade-off** entre velocidade, portabilidade e produtividade
- Sistemas reais costumam **misturar várias linguagens**

---

⬅️ [Anterior: Sistema Operacional](./04-sistema-operacional.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Linux e Arch](./06-linux-arch.md)
