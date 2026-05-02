<div align="center">

# 📚 Fundamentos da Computação

### 🎓 Um guia completo dos fundamentos que todo dev precisa dominar

*Do fio elétrico ao código rodando em produção*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
![Status](https://img.shields.io/badge/Status-Em%20construção-brightgreen)
![Tópicos](https://img.shields.io/badge/Tópicos-41-blue)
![Idioma](https://img.shields.io/badge/Idioma-Português%20BR-green)

</div>

---

## 🎯 Sobre este guia

> 💡 **Objetivo**: ser uma referência rápida e didática sobre os fundamentos da computação para devs que querem entender o que está acontecendo **por baixo do código**.

Esse repositório é uma compilação organizada que vai do hardware físico até observabilidade em produção. Cada tópico é um arquivo `.md` separado, escrito de forma direta com:

- 🎨 **Analogias** pra concretizar conceitos abstratos
- 💡 **Callouts didáticos** com "Em palavras simples"
- 🤔 **"Por que importa?"** conectando teoria com a prática do dev
- 📊 **Tabelas comparativas** pra decisões rápidas
- 💻 **Exemplos de código** em múltiplas stacks

---

## 🛠️ Sobre os exemplos de código

Os exemplos práticos usam principalmente **Java/Spring Boot**, mas **os conceitos são universais**:

| 🎯 Conceito | ☕ Java | 🟢 Node.js | 🐍 Python | 💎 C# / .NET | 🐹 Go |
|---|---|---|---|---|---|
| **Web framework** | Spring Boot | Express, NestJS | FastAPI, Django | ASP.NET Core | Gin, Echo |
| **ORM / DB access** | JPA/Hibernate | Prisma, TypeORM | SQLAlchemy | Entity Framework | GORM |
| **DI / IoC** | Spring | NestJS, tsyringe | (manual ou DI libs) | Built-in | Wire, Fx |
| **Testes** | JUnit + Mockito | Jest, Vitest | pytest | xUnit | testing |
| **Async** | CompletableFuture | async/await | asyncio | async/await | goroutines |

> 💡 **Dica**: quando você ver `@Service`, `@Autowired` ou `JpaRepository`, traduza pro equivalente da sua stack. **A ideia por trás é sempre a mesma.**

---

## 📚 Roteiro de Estudos

### 🏗️ Parte 1 — Fundamentos da Máquina

> 🎯 *Como o computador funciona fisicamente, da eletricidade ao binário*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 1 | [🖨️ **Hardware e Periféricos**](./01-hardware.md) | CPU • RAM • SSD • GPU • Periféricos |
| 2 | [⚙️ **Processador (CPU)**](./02-processador.md) | Ciclo de instrução • ULA • Clock • Cache |
| 3 | [🧠 **Memória e Execução**](./03-memoria.md) | Memória virtual • Stack • Heap • ELF |
| 4 | [🌉 **Sistema Operacional**](./04-sistema-operacional.md) | User vs Kernel • Syscalls • VMs |
| 5 | [📝 **Linguagens de Programação**](./05-linguagens.md) | Compilação • Interpretação • JIT |
| 6 | [🐧 **Linux e Arch (deep dive)**](./06-linux-arch.md) | Filosofia KISS • Pacman • AUR |

---

### 🔌 Parte 2 — Comunicação e Persistência

> 🎯 *Como aplicações conversam entre si e guardam dados*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 7 | [🌐 **Redes**](./07-redes.md) | TCP/IP • DNS • HTTP/HTTPS • CORS |
| 8 | [🗄️ **Banco de Dados**](./08-banco-de-dados.md) | SQL • Índices • ACID • N+1 problem |

---

### 🧠 Parte 3 — Conceitos Avançados

> 🎯 *Tópicos que atravessam várias camadas e diferenciam devs juniors de seniors*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 9 | [🧵 **Concorrência e Paralelismo**](./09-concorrencia.md) | Threads • Race conditions • Mutex • Async |
| 10 | [🔐 **Criptografia**](./10-criptografia.md) | Hash • Simétrica • Assimétrica • JWT |
| 11 | [🛠️ **Compiladores**](./11-compiladores.md) | Lexer • Parser • AST • LLVM |

---

### ⚙️ Parte 4 — Engenharia de Software

> 🎯 *Como organizar código que dura anos e é mantido por times*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 12 | [🌿 **Git e Versionamento**](./12-git.md) | Branches • Merge vs Rebase • Conventional Commits |
| 13 | [📊 **Estruturas de Dados e Algoritmos**](./13-estruturas-dados.md) | Big O • Hash • Árvores • Grafos |
| 14 | [🎨 **Design Patterns**](./14-design-patterns.md) | GoF • Strategy • Observer • Repository |
| 15 | [🏛️ **Arquitetura de Software**](./15-arquitetura-software.md) | SOLID • Clean Arch • Hexagonal • DDD |

---

### 🚀 Parte 5 — Operação e Qualidade

> 🎯 *Como colocar e manter aplicações em produção*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 16 | [🚀 **DevOps e CI/CD**](./16-devops-cicd.md) | GitHub Actions • Docker • K8s • IaC |
| 17 | [🧪 **Testes**](./17-testes.md) | Pirâmide • TDD • Mocks • Testcontainers |
| 18 | [🔍 **Observabilidade**](./18-observabilidade.md) | Logs • Métricas • Tracing • Alertas |

---

### 🌐 Parte 6 — Tópicos Avançados de Produção

> 🎯 *Tópicos que aparecem quando seus sistemas crescem em escala e complexidade*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 19 | [📨 **Mensageria**](./19-mensageria.md) | RabbitMQ • Kafka • Filas • Outbox • Saga |
| 20 | [⚡ **Caching**](./20-caching.md) | Redis • TTL • Padrões • Cache stampede |
| 21 | [🔌 **APIs Avançadas**](./21-apis-avancadas.md) | REST maduro • GraphQL • gRPC • OpenAPI |
| 22 | [🛡️ **Segurança**](./22-seguranca.md) | OWASP Top 10 • XSS • CSRF • IDOR • Secrets |
| 23 | [📈 **Performance**](./23-performance.md) | Profiling • Big O • Latência • Load testing |
| 24 | [☁️ **Cloud**](./24-cloud.md) | AWS • GCP • Azure • IaaS • PaaS • Serverless |
| 25 | [🌍 **Sistemas Distribuídos**](./25-sistemas-distribuidos.md) | CAP • Consenso • Replicação • Sharding |

---

### 🤖 Parte 7 — Inteligência Artificial pra Devs

> 🎯 *IA prática pra dev: integrar LLMs em apps, não virar cientista de dados*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 26 | [🤖 **Fundamentos de IA/ML**](./26-fundamentos-ia.md) | ML • Deep Learning • Treinamento • Métricas |
| 27 | [🧠 **LLMs e como funcionam**](./27-llms.md) | Tokens • Embeddings • Contexto • Transformers |
| 28 | [🗂️ **RAG**](./28-rag.md) | Retrieval • Embeddings • Vector DB • Pipelines |
| 29 | [✍️ **Engenharia de Prompt**](./29-engenharia-prompt.md) | Few-shot • CoT • System prompts • Padrões |
| 30 | [🛡️ **IA com Responsabilidade**](./30-ia-responsavel.md) | Alucinação • Vieses • Custos • Privacidade |
| 31 | [🔌 **Integração com APIs de LLMs**](./31-integracao-llms.md) | OpenAI • Anthropic • Function calling • MCP |

---

### 🛠️ Parte 8 — Especializações Avançadas

> 🎯 *Tecnologias específicas que você pode ou não usar dependendo do projeto*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 32 | [🐳 **Kubernetes Profundo**](./32-kubernetes.md) | Pods • Deployments • Helm • Operators |
| 33 | [🧪 **MLOps**](./33-mlops.md) | Versionamento • Drift • CT • Feature Store |
| 34 | [📱 **Mobile e PWA**](./34-mobile-pwa.md) | React Native • Flutter • PWA • Offline-first |

---

### 🌎 Parte 9 — Carreira e Boas Práticas

> 🎯 *Tópicos que valem pra qualquer dev, independente da stack*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 35 | [♿ **Acessibilidade Web**](./35-acessibilidade.md) | WCAG • ARIA • Leitores de tela • Contraste |
| 36 | [🌍 **Internacionalização (i18n)**](./36-i18n.md) | i18n vs l10n • ICU • Intl API • RTL |
| 37 | [📜 **LGPD e GDPR**](./37-lgpd-gdpr.md) | Bases legais • Direitos do titular • Anonimização |
| 38 | [🧰 **Soft Skills pra Devs**](./38-soft-skills.md) | Comunicação • Code review • Mentoria • Burnout |

---

### ☕ Parte 10 — Lógica e Java

> 🎯 *Fundamentos de programação e mergulho em Java*

| 📖 | 📑 Tópico | 🔑 Conceitos-chave |
|:-:|---|---|
| 39 | [🧮 **Lógica de Programação e Paradigmas**](./39-logica-paradigmas.md) | Variáveis • Loops • Procedural • Funcional • OO |
| 40 | [🏛️ **POO: Os 4 Pilares**](./40-poo-pilares.md) | Abstração • Encapsulamento • Herança • Polimorfismo |
| 41 | [☕ **Java: Da Linguagem ao Ecossistema**](./41-java-completo.md) | JVM • Streams • Records • Virtual Threads • Maven |

---

## 🗺️ Mapa Mental: Como Tudo se Conecta

```
                       ┌─────────────────────────────┐
                       │  ⚙️  ENGENHARIA DE SOFTWARE  │
                       │                             │
                       │  📊 13 - Estruturas         │
                       │  🎨 14 - Design Patterns    │
                       │  🏛️  15 - Arquitetura       │
                       │  🌿 12 - Git                │
                       └──────────────┬──────────────┘
                                      ↓ aplicam-se em
        ┌─────────────────────────────────┐
        │  💻 Você (escreve código)       │       ┌──────────────────────┐
        └────────────────┬────────────────┘       │ 🧠 TRANSVERSAIS      │
                         ↓                        │                      │
        ┌─────────────────────────────────┐       │ 🛠️  11 - Compiladores │
        │  📝 Linguagem de Programação    │ ←──→  │ 🧵 9 - Concorrência  │
        │  ── Tópico 5 ──                 │       │ 🔐 10 - Criptografia │
        └────────────────┬────────────────┘       └──────────────────────┘
                         ↓
        ┌─────────────────────────────────┐       ┌──────────────────────┐
        │  🌉 Sistema Operacional         │ ←──→  │ 🔌 COMUNICAÇÃO       │
        │  ── Tópicos 4 e 6 ──            │       │                      │
        └────────────────┬────────────────┘       │ 🌐 7 - Redes         │
                         ↓                        │ 🗄️  8 - Banco        │
        ┌─────────────────────────────────┐       └──────────────────────┘
        │  🧠 Memória (RAM)               │
        │  ── Tópico 3 ──                 │
        └────────────────┬────────────────┘
                         ↓
        ┌─────────────────────────────────┐
        │  ⚙️  Processador (CPU)           │
        │  ── Tópico 2 ──                 │
        └────────────────┬────────────────┘
                         ↓
        ┌─────────────────────────────────┐
        │  🖨️  Hardware Físico             │
        │  ── Tópico 1 ──                 │
        └─────────────────────────────────┘

       Tudo isso roda em produção sob:
       ┌───────────────────────────────────┐
       │  🚀 OPERAÇÃO E QUALIDADE          │
       │                                   │
       │  🚀 16 - DevOps e CI/CD           │
       │  🧪 17 - Testes                   │
       │  🔍 18 - Observabilidade          │
       └───────────────────────────────────┘

       Quando o sistema cresce em escala:
       ┌───────────────────────────────────────────┐
       │  🌐 TÓPICOS AVANÇADOS DE PRODUÇÃO         │
       │                                           │
       │  📨 19 - Mensageria  ⚡ 20 - Caching      │
       │  🔌 21 - APIs        🛡️  22 - Segurança    │
       │  📈 23 - Performance ☁️  24 - Cloud        │
       │  🌍 25 - Sistemas Distribuídos            │
       └───────────────────────────────────────────┘

       E o futuro chegou: integrando IA nos apps:
       ┌───────────────────────────────────────────┐
       │  🤖 INTELIGÊNCIA ARTIFICIAL PRA DEVS      │
       │                                           │
       │  🤖 26 - Fundamentos  🧠 27 - LLMs        │
       │  🗂️  28 - RAG          ✍️  29 - Prompt     │
       │  🛡️  30 - Responsável  🔌 31 - APIs LLMs  │
       └───────────────────────────────────────────┘

       Especializações em tecnologias específicas:
       ┌───────────────────────────────────────────┐
       │  🛠️  ESPECIALIZAÇÕES AVANÇADAS             │
       │                                           │
       │  🐳 32 - Kubernetes  🧪 33 - MLOps        │
       │  📱 34 - Mobile e PWA                     │
       └───────────────────────────────────────────┘

       Boas práticas que valem pra qualquer dev:
       ┌───────────────────────────────────────────┐
       │  🌎 CARREIRA E BOAS PRÁTICAS              │
       │                                           │
       │  ♿ 35 - Acessibilidade  🌍 36 - i18n      │
       │  📜 37 - LGPD/GDPR     🧰 38 - Soft Skills │
       └───────────────────────────────────────────┘

       Fundamentos de programação e Java:
       ┌───────────────────────────────────────────┐
       │  ☕ LÓGICA E JAVA                          │
       │                                           │
       │  🧮 39 - Lógica e Paradigmas              │
       │  🏛️  40 - POO: 4 Pilares                  │
       │  ☕ 41 - Java Completo                     │
       └───────────────────────────────────────────┘
```

> 💡 **Como ler**:
> - 📐 **Coluna central** = stack vertical da máquina (de baixo pra cima)
> - 🔄 **Lados** = conceitos transversais que afetam várias camadas
> - 🎯 **Topo** = engenharia (como você desenha o código)
> - 🏁 **Meio-base** = operação (como mantém vivo em produção)
> - 🌐 **Avançados** = entram em jogo quando o sistema cresce
> - 🤖 **IA** = camada nova que está virando essencial pra devs modernos
> - 🛠️ **Especializações** = tecnologias específicas (use conforme necessidade)
> - 🌎 **Carreira** = boas práticas universais
> - ☕ **Lógica e Java** = fundamentos da programação + linguagem específica

---

## 🎓 Por onde começar?

### 👶 Nunca estudou fundamentos?
Comece pela **Parte 1** (tópicos 1 a 6), na ordem. Cada um constrói sobre o anterior.

### 💼 Já é dev mas quer preencher buracos?
Vá direto pros tópicos da **Parte 4** (12 a 15) — são os que mais aparecem em entrevistas e dia a dia.

### 🚀 Quer aprender DevOps?
Pula pra **Parte 5** (tópicos 16 a 18). Mas leia **8 (Banco)** e **7 (Redes)** antes se nunca viu.

### 🌐 Já trabalha com microserviços / sistemas grandes?
**Parte 6** (tópicos 19 a 25) é onde mora a complexidade real de produção em escala.

### 🤖 Quer integrar IA em aplicações?
**Parte 7** (tópicos 26 a 31). Você não precisa virar cientista de dados — esses tópicos são pra **dev** que quer usar IA bem.

### ☕ Vai aprender Java do zero ou revisar fundamentos?
**Parte 10** (tópicos 39 a 41). Lógica → Paradigmas → POO → Java específico. Perfeito pra quem está começando ou quer organizar o que já sabe.

### 🎯 Veio pra um tópico específico?
Use o sumário acima. Cada arquivo é autossuficiente — você pode ler em qualquer ordem.

---

## ✅ Status do Conteúdo

### 🏗️ Parte 1 — Fundamentos da Máquina
- [x] 🖨️ Hardware e Periféricos
- [x] ⚙️ Processador (CPU)
- [x] 🧠 Memória e Execução
- [x] 🌉 Sistema Operacional
- [x] 📝 Linguagens de Programação
- [x] 🐧 Linux e Arch

### 🔌 Parte 2 — Comunicação e Persistência
- [x] 🌐 Redes
- [x] 🗄️ Banco de Dados

### 🧠 Parte 3 — Conceitos Avançados
- [x] 🧵 Concorrência e Paralelismo
- [x] 🔐 Criptografia
- [x] 🛠️ Compiladores

### ⚙️ Parte 4 — Engenharia de Software
- [x] 🌿 Git e Versionamento
- [x] 📊 Estruturas de Dados e Algoritmos
- [x] 🎨 Design Patterns
- [x] 🏛️ Arquitetura de Software

### 🚀 Parte 5 — Operação e Qualidade
- [x] 🚀 DevOps e CI/CD
- [x] 🧪 Testes
- [x] 🔍 Observabilidade

### 🌐 Parte 6 — Tópicos Avançados de Produção
- [x] 📨 Mensageria
- [x] ⚡ Caching
- [x] 🔌 APIs Avançadas
- [x] 🛡️ Segurança
- [x] 📈 Performance
- [x] ☁️ Cloud
- [x] 🌍 Sistemas Distribuídos

### 🤖 Parte 7 — Inteligência Artificial pra Devs
- [x] 🤖 Fundamentos de IA/ML
- [x] 🧠 LLMs
- [x] 🗂️ RAG
- [x] ✍️ Engenharia de Prompt
- [x] 🛡️ IA com Responsabilidade
- [x] 🔌 Integração com APIs de LLMs

### 🛠️ Parte 8 — Especializações Avançadas
- [x] 🐳 Kubernetes Profundo
- [x] 🧪 MLOps
- [x] 📱 Mobile e PWA

### 🌎 Parte 9 — Carreira e Boas Práticas
- [x] ♿ Acessibilidade Web
- [x] 🌍 Internacionalização (i18n)
- [x] 📜 LGPD e GDPR
- [x] 🧰 Soft Skills pra Devs

### ☕ Parte 10 — Lógica e Java
- [x] 🧮 Lógica de Programação e Paradigmas
- [x] 🏛️ POO: Os 4 Pilares
- [x] ☕ Java: Da Linguagem ao Ecossistema

### 🌱 Possíveis tópicos futuros
- [ ] 🦫 Go (linguagem completa)
- [ ] 🐍 Python (linguagem completa)
- [ ] 🐦 Kotlin (linguagem completa)
- [ ] 🟢 Node.js fundamentos
- [ ] ⚛️ React profundo
- [ ] 🗃️ Bancos NoSQL específicos (MongoDB, Cassandra)
- [ ] 📊 Engenharia de dados (ETL, data warehouses, lakes)

---

## 📚 Referências Recomendadas

### 📘 Livros Essenciais

#### 🏗️ Fundamentos e Arquitetura
- *Operating Systems: Three Easy Pieces* — gratuito em [pages.cs.wisc.edu/~remzi/OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- *Computer Organization and Design* — Patterson & Hennessy
- *Designing Data-Intensive Applications* — Martin Kleppmann

#### 🎨 Design e Código
- *Clean Code* — Robert C. Martin
- *Clean Architecture* — Robert C. Martin
- *The Pragmatic Programmer* — Hunt & Thomas
- *Design Patterns: Elements of Reusable OO Software* — Gang of Four
- *Refactoring* — Martin Fowler

#### 🏛️ Domain-Driven Design
- *Domain-Driven Design* — Eric Evans (📕 livro azul)
- *Implementing Domain-Driven Design* — Vaughn Vernon (📗 livro vermelho)

#### 🚀 Operação e Sistemas Distribuídos
- *Site Reliability Engineering* — gratuito em [sre.google/books](https://sre.google/books/)
- *Distributed Systems* — Maarten van Steen
- *Building Microservices* — Sam Newman

#### 🛡️ Segurança
- *The Web Application Hacker's Handbook*
- *Cryptography Engineering* — Schneier, Ferguson, Kohno

#### 🤖 IA e Machine Learning
- *Hands-On Machine Learning* — Aurélien Géron
- *Deep Learning* — Goodfellow, Bengio, Courville (gratuito em [deeplearningbook.org](https://www.deeplearningbook.org/))
- *Designing Machine Learning Systems* — Chip Huyen
- *AI Engineering* — Chip Huyen (focado em LLMs e produção)

#### ☕ Java
- *Effective Java* — Joshua Bloch (📕 livro essencial)
- *Java: The Complete Reference* — Herbert Schildt
- *Modern Java in Action* — Raoul-Gabriel Urma (Java 8+ funcional)
- *Java Concurrency in Practice* — Brian Goetz

### 📖 Documentação Oficial
- 🐧 [ArchWiki](https://wiki.archlinux.org/) — referência absoluta de Linux
- ☕ [Spring Boot Docs](https://docs.spring.io/spring-boot/)
- 🐘 [PostgreSQL Docs](https://www.postgresql.org/docs/)
- 🌐 [MDN Web Docs](https://developer.mozilla.org/)
- 📋 [12-Factor App](https://12factor.net/)
- 🔴 [Redis Docs](https://redis.io/docs/)
- 🌊 [Kafka Docs](https://kafka.apache.org/documentation/)
- 🛡️ [OWASP](https://owasp.org/)

### 🎥 Canais no YouTube
- 🔧 [Ben Eater](https://www.youtube.com/@BenEater) — CPU do zero, redes, eletrônica
- 🎓 [Computerphile](https://www.youtube.com/@Computerphile) — conceitos da computação
- 🔥 [Fireship](https://www.youtube.com/@Fireship) — explicações rápidas
- 🏗️ [ByteByteGo](https://www.youtube.com/@ByteByteGo) — system design
- 🤖 [3Blue1Brown](https://www.youtube.com/@3blue1brown) — matemática e neural networks (visualizações lindas)
- 🧠 [Andrej Karpathy](https://www.youtube.com/@AndrejKarpathy) — LLMs do zero, fundamentos de IA
- 📊 [StatQuest](https://www.youtube.com/@statquest) — estatística e ML didáticos
- ☕ [Java Brains](https://www.youtube.com/@Java.Brains) — Java e Spring profundo
- ☕ [Marco Codes](https://www.youtube.com/@MarcoCodes) — Java moderno

### 🛠️ Ferramentas Úteis
- 🔬 [godbolt.org](https://godbolt.org) — explorar assembly
- 🔍 [regex101.com](https://regex101.com) — testar regex
- 🎨 [excalidraw.com](https://excalidraw.com) — diagramas rápidos
- 📐 [draw.io](https://app.diagrams.net) — diagramas profissionais
- 🗃️ [DB Diagram](https://dbdiagram.io) — modelar bancos
- 🌐 [HTTPie](https://httpie.io) — alternativa ao curl
- 📊 [k6.io](https://k6.io) — load testing
- 🤗 [Hugging Face](https://huggingface.co) — modelos de IA open source
- 🦜 [LangChain](https://www.langchain.com) — orquestração de LLMs

### 🎯 Aprofundamento por Tópico
- 🗄️ **Banco**: [Use The Index, Luke!](https://use-the-index-luke.com/)
- 🌿 **Git**: [Pro Git Book](https://git-scm.com/book) (gratuito)
- 📊 **Algoritmos**: [VisuAlgo](https://visualgo.net/)
- 🏗️ **System Design**: [System Design Primer](https://github.com/donnemartin/system-design-primer)
- 🌐 **Frontend**: [web.dev](https://web.dev/)
- 🛡️ **Segurança**: [OWASP Top 10](https://owasp.org/www-project-top-ten/) • [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- ☁️ **Cloud**: [AWS Skill Builder](https://skillbuilder.aws/) • [Google Cloud Skills Boost](https://www.cloudskillsboost.google/)
- 🌍 **Sistemas Distribuídos**: [The Morning Paper](https://blog.acolyer.org/) (papers explicados)
- 🤖 **IA pra Devs**: [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook) • [OpenAI Cookbook](https://github.com/openai/openai-cookbook)
- 📝 **Prompt Engineering**: [Anthropic Prompt Library](https://docs.claude.com/en/prompt-library) • [Learn Prompting](https://learnprompting.org/)
- ☕ **Java**: [Baeldung](https://www.baeldung.com/) • [Spring Documentation](https://spring.io/projects/spring-boot) • [DEV.JAVA](https://dev.java/)

---

## 🤝 Contribuições

Encontrou um erro? Tem sugestão? Quer adicionar um tópico?

- 🐛 **Reportar bug ou erro**: abra uma [Issue](../../issues)
- 💡 **Sugerir melhoria**: abra uma [Discussion](../../discussions) ou Pull Request
- ⭐ **Achou útil?** Deixa uma estrela no repo!

---

## 📄 Licença

[MIT](./LICENSE) — sinta-se livre pra usar, adaptar e compartilhar pros seus estudos.

---

<div align="center">

### 🚀 Bons estudos!

*Feito com ☕ e muito 💜 por <a href="https://github.com/mauricioandrade">@mauricioandrade</a>*

</div>
