<div align="center">

# 📚 Fundamentos da Computação

### 🎓 Um guia completo dos fundamentos que todo dev precisa dominar

*Do fio elétrico ao código rodando em produção*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
![Status](https://img.shields.io/badge/Status-Em%20construção-brightgreen)
![Tópicos](https://img.shields.io/badge/Tópicos-18-blue)
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
```

> 💡 **Como ler**:
> - 📐 **Coluna central** = stack vertical da máquina (de baixo pra cima)
> - 🔄 **Lados** = conceitos transversais que afetam várias camadas
> - 🎯 **Topo** = engenharia (como você desenha o código)
> - 🏁 **Base** = operação (como mantém vivo em produção)

---

## 🎓 Por onde começar?

### 👶 Nunca estudou fundamentos?
Comece pela **Parte 1** (tópicos 1 a 6), na ordem. Cada um constrói sobre o anterior.

### 💼 Já é dev mas quer preencher buracos?
Vá direto pros tópicos da **Parte 4** (12 a 15) — são os que mais aparecem em entrevistas e dia a dia.

### 🚀 Quer aprender DevOps?
Pula pra **Parte 5** (tópicos 16 a 18). Mas leia **8 (Banco)** e **7 (Redes)** antes se nunca viu.

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

### 🌱 Próximos tópicos planejados
- [ ] 📨 Mensageria (RabbitMQ, Kafka, eventos)
- [ ] ⚡ Caching (Redis, estratégias, TTL)
- [ ] 🔌 APIs avançadas (REST maduro, GraphQL, gRPC)
- [ ] 🛡️ Segurança de aplicação (OWASP Top 10)
- [ ] 📈 Performance e profiling
- [ ] ☁️ Cloud (AWS/GCP fundamentos)
- [ ] 🌍 Sistemas distribuídos (CAP, consenso)

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

#### 🏛️ Domain-Driven Design
- *Domain-Driven Design* — Eric Evans (📕 livro azul)
- *Implementing Domain-Driven Design* — Vaughn Vernon (📗 livro vermelho)

#### 🚀 Operação
- *Site Reliability Engineering* — gratuito em [sre.google/books](https://sre.google/books/)

### 📖 Documentação Oficial
- 🐧 [ArchWiki](https://wiki.archlinux.org/) — referência absoluta de Linux
- ☕ [Spring Boot Docs](https://docs.spring.io/spring-boot/)
- 🐘 [PostgreSQL Docs](https://www.postgresql.org/docs/)
- 🌐 [MDN Web Docs](https://developer.mozilla.org/)
- 📋 [12-Factor App](https://12factor.net/)

### 🎥 Canais no YouTube
- 🔧 [Ben Eater](https://www.youtube.com/@BenEater) — CPU do zero, redes, eletrônica
- 🎓 [Computerphile](https://www.youtube.com/@Computerphile) — conceitos da computação
- 🔥 [Fireship](https://www.youtube.com/@Fireship) — explicações rápidas
- 🏗️ [ByteByteGo](https://www.youtube.com/@ByteByteGo) — system design

### 🛠️ Ferramentas Úteis
- 🔬 [godbolt.org](https://godbolt.org) — explorar assembly
- 🔍 [regex101.com](https://regex101.com) — testar regex
- 🎨 [excalidraw.com](https://excalidraw.com) — diagramas rápidos
- 📐 [draw.io](https://app.diagrams.net) — diagramas profissionais
- 🗃️ [DB Diagram](https://dbdiagram.io) — modelar bancos
- 🌐 [HTTPie](https://httpie.io) — alternativa ao curl

### 🎯 Aprofundamento por Tópico
- 🗄️ **Banco**: [Use The Index, Luke!](https://use-the-index-luke.com/)
- 🌿 **Git**: [Pro Git Book](https://git-scm.com/book) (gratuito)
- 📊 **Algoritmos**: [VisuAlgo](https://visualgo.net/)
- 🏗️ **System Design**: [System Design Primer](https://github.com/donnemartin/system-design-primer)
- 🌐 **Frontend**: [web.dev](https://web.dev/)
- 🛡️ **Segurança**: [OWASP Top 10](https://owasp.org/www-project-top-ten/)

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
