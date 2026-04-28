# 🏛️ Arquitetura de Software

> Como **organizar** o código de uma aplicação inteira. Não é sobre frameworks ou linguagens — é sobre **fronteiras**, **dependências** e **responsabilidades**.

---

## 🎯 Por que arquitetura importa

Software pequeno aguenta qualquer estrutura. Software que **dura anos** e tem **vários devs** mexendo precisa de regras claras, senão vira:

- Um módulo que ninguém entende
- Mudança em um lugar quebra outros 5
- Bug que reproduz só em produção
- Dev novo demora 3 meses pra produzir

> 💡 **Arquitetura é sobre o que vai mudar com o tempo**. Você não consegue prever todos os requisitos, mas pode estruturar pra que mudanças sejam **localizadas**.

---

## 🧱 Princípios Fundamentais

### Coesão e Acoplamento

- **Coesão**: o quanto as coisas relacionadas estão juntas. **Alta coesão é bom.**
- **Acoplamento**: o quanto módulos dependem uns dos outros. **Baixo acoplamento é bom.**

```
✅ Bom: módulo de Usuario tem tudo sobre usuário, conhece pouco de outros
❌ Ruim: classe de Usuario importa Pedido, Pagamento, Email, Estoque...
```

### SOLID

5 princípios pra OO. **Memorize a sigla:**

| Letra | Princípio | Resumo |
|---|---|---|
| **S** | Single Responsibility | Uma classe, **um motivo** pra mudar |
| **O** | Open/Closed | Aberto pra extensão, fechado pra modificação |
| **L** | Liskov Substitution | Subclasses devem substituir superclasses sem quebrar |
| **I** | Interface Segregation | Interfaces pequenas e focadas, não "gigante que faz tudo" |
| **D** | Dependency Inversion | Depender de **abstrações**, não de implementações |

> 💡 **Dependency Inversion** é a base de toda arquitetura limpa moderna. É por isso que você cria interfaces e injeta com DI.

### DRY, KISS, YAGNI

- **DRY** (Don't Repeat Yourself): evite duplicação de **conhecimento** (não de código literal)
- **KISS** (Keep It Simple, Stupid): a solução mais simples que resolve
- **YAGNI** (You Aren't Gonna Need It): não construa pra "talvez precisar"

---

## 🏗️ Estilos Arquiteturais

### 1️⃣ Monolito

Uma única aplicação, tudo junto.

```
┌─────────────────────┐
│   Aplicação Única   │
│  ┌──────────────┐   │
│  │   Frontend   │   │
│  ├──────────────┤   │
│  │   Backend    │   │
│  ├──────────────┤   │
│  │   Database   │   │
│  └──────────────┘   │
└─────────────────────┘
```

✅ Simples de desenvolver, deployar, debugar
✅ Performance (tudo em memória)
❌ Difícil escalar partes específicas
❌ Cresce e vira "monolito espaguete"

> 💡 **Monolito BEM ESTRUTURADO** é melhor que microserviços mal feitos. Comece monolito.

### 2️⃣ Microserviços

Aplicação dividida em **serviços pequenos e independentes**.

```
┌──────────┐   ┌──────────┐   ┌──────────┐
│  Users   │   │  Orders  │   │ Payments │
│ Service  │   │ Service  │   │ Service  │
└────┬─────┘   └────┬─────┘   └────┬─────┘
     │              │               │
     └──────────────┴───────────────┘
              (mensageria)
```

✅ Times trabalham independentes
✅ Escala partes específicas
✅ Pode usar tech diferente por serviço
❌ Complexidade enorme (rede, observabilidade, deploys)
❌ Transações distribuídas são difíceis

> ⚠️ **Microserviços só compensam em escala**. Pra time de 5 pessoas, custo > benefício.

### 3️⃣ Serverless

Você só escreve funções; o cloud provider gerencia tudo.

```
Cliente → API Gateway → Lambda Function → Database
```

✅ Escala automática, paga só pelo uso
✅ Sem servidor pra manter
❌ Cold starts (latência inicial)
❌ Vendor lock-in
❌ Limites de tempo de execução

---

## 🧅 Clean Architecture

Proposta por **Robert C. Martin (Uncle Bob)**. A ideia: **regras de negócio independentes de framework, banco e UI**.

### As 4 camadas

```
        ┌────────────────────────────────┐
        │  Frameworks & Drivers          │  ← Spring, JPA, REST
        │  ┌──────────────────────────┐  │
        │  │  Interface Adapters      │  │  ← Controllers, Repositories
        │  │  ┌────────────────────┐  │  │
        │  │  │  Use Cases         │  │  │  ← Application logic
        │  │  │  ┌──────────────┐  │  │  │
        │  │  │  │  Entities    │  │  │  │  ← Regras de negócio puras
        │  │  │  └──────────────┘  │  │  │
        │  │  └────────────────────┘  │  │
        │  └──────────────────────────┘  │
        └────────────────────────────────┘

       Direção das dependências: → DENTRO
```

### Regra de Ouro: Dependências apontam pra dentro

- Camada de **fora** conhece a de dentro
- Camada de **dentro NUNCA** conhece a de fora
- Núcleo (Entities) não depende de NADA

### Aplicação prática (exemplo de estrutura)

```
src/
├── domain/              ← Entities + Value Objects + regras puras
│   ├── Pedido.java
│   └── ValueObjects/
├── application/         ← Use Cases (orquestram domínio)
│   └── CriarPedidoUseCase.java
├── adapters/            ← Adapters concretos
│   ├── in/web/          ← Controllers REST
│   └── out/persistence/ ← Repositories
└── infrastructure/      ← Configuração de framework
    └── FrameworkConfig.java
```

> 💡 **A estrutura acima é em Java/Spring**, mas o **conceito vale pra qualquer stack**:
> - **Node.js**: pastas equivalentes em TypeScript com NestJS
> - **Python**: módulos com FastAPI/Django seguindo a mesma divisão
> - **Go**: packages organizados em `domain/`, `application/`, `adapters/`
> - **C#/.NET**: namespaces e pastas com mesma divisão

> 💡 **Por que isso é poderoso?** Você pode trocar o framework web, o ORM, o tipo de banco — **sem mexer nas regras de negócio**. Se hoje é REST e amanhã vira gRPC, a camada de domínio nem percebe.

---

## 🏰 Hexagonal Architecture (Ports and Adapters)

Proposta por **Alistair Cockburn**. Mesma ideia da Clean, com nomenclatura diferente.

```
                  ┌──────────────────────┐
                  │      Driving          │
                  │   (HTTP, CLI, Tests) │
                  └─────────┬────────────┘
                            ↓ Port
                ┌───────────────────────┐
                │                       │
                │      DOMAIN           │
                │   (regras puras)      │
                │                       │
                └───────────┬───────────┘
                            ↓ Port
                  ┌─────────────────────┐
                  │      Driven          │
                  │  (DB, Email, APIs)  │
                  └─────────────────────┘
```

- **Port** = interface (contrato definido pelo domínio)
- **Adapter** = implementação concreta da port

### Tipos de Port

- **Driving (input)**: alguém chama o domínio (HTTP, CLI, message queue)
- **Driven (output)**: o domínio precisa de algo externo (DB, email)

> 🤔 **Diferença pra Clean Architecture?** Quase nenhuma. São **a mesma ideia** com vocabulário diferente. Hexagonal foca mais em "ports", Clean em "camadas circulares". Ambas: **dependências invertidas, núcleo isolado**.

---

## 🎯 Domain-Driven Design (DDD)

Mais que arquitetura — é uma **filosofia de modelagem**. Foco no **domínio do negócio**.

### Conceitos centrais

#### Ubiquitous Language

Time inteiro (devs + negócio) usa o **mesmo vocabulário**. Se o pessoal de combustível diz "movimentação", o código tem `Movimentacao`, não `FuelEntry`.

#### Bounded Context

Modelos diferentes pra contextos diferentes. "Cliente" no contexto de Vendas é diferente de "Cliente" no contexto de Suporte.

```
┌─────────────────┐    ┌─────────────────┐
│   Vendas BC     │    │   Logística BC  │
│                 │    │                 │
│  Cliente        │    │  Cliente        │
│  - desconto     │    │  - endereço     │
│  - histórico    │    │  - rota         │
└─────────────────┘    └─────────────────┘
```

#### Building Blocks

| Elemento | O que é |
|---|---|
| **Entity** | Tem identidade única que persiste (`Pedido` com ID) |
| **Value Object** | Definido apenas pelos atributos (`Endereco`, `Dinheiro`) |
| **Aggregate** | Cluster de entities tratado como unidade (`Pedido` + `ItensPedido`) |
| **Repository** | Abstração de persistência |
| **Domain Service** | Lógica que não pertence a uma entity específica |
| **Domain Event** | Algo que aconteceu no domínio (`PedidoFinalizado`) |

#### Strategic vs Tactical DDD

- **Strategic**: bounded contexts, ubiquitous language, context mapping
- **Tactical**: entities, value objects, aggregates, repositories

> 💡 **Exemplo prático**: imagine um sistema de e-commerce. `Pedido` é uma **entity** (tem ID, persiste). `Endereco` é um **value object** (definido pelos atributos: rua, número, CEP). `Pedido` + seus `ItensPedido` formam um **aggregate** tratado como unidade. A regra "pedido não pode ter mais de 10 itens" vive no domínio, não no controller.

---

## 🧩 CQRS e Event Sourcing

### CQRS (Command Query Responsibility Segregation)

Separa modelos de **leitura** e **escrita**.

```
Cliente
  ↓ Command (escrita)         ↓ Query (leitura)
Write Model (validação)    Read Model (otimizado pra UI)
  ↓                             ↑
  └── Events ────────────────→ Sincroniza
```

**Quando vale**: domínios complexos com leituras muito diferentes das escritas.

### Event Sourcing

Em vez de salvar **estado**, salva **todos os eventos** que levaram a ele.

```
Eventos:
- ContaCriada(saldo: 0)
- DepositoFeito(100)
- SaqueFeito(30)

Estado atual = sum dos eventos = 70
```

✅ Auditoria perfeita, replay do histórico
❌ Complexidade alta, queries difíceis sem CQRS

---

## 🌐 Frontend Architecture

Não é só backend que tem arquitetura.

### Patterns clássicos

- **MVC** (Model-View-Controller): clássico
- **MVVM** (Model-View-ViewModel): popular em Vue, Angular
- **Flux/Redux**: estado global imutável (React)
- **Component-based**: tudo é componente reutilizável

### Atomic Design

Pra design systems:

1. **Atoms**: button, input, label
2. **Molecules**: form field (input + label)
3. **Organisms**: header, formulário completo
4. **Templates**: layout
5. **Pages**: instâncias preenchidas

---

## 📡 Comunicação Entre Serviços

| Estilo | Quando usar |
|---|---|
| **REST** | APIs públicas, simples, padrão da web |
| **GraphQL** | Cliente precisa flexibilidade no que pega |
| **gRPC** | Comunicação interna, alta performance |
| **Mensageria** (RabbitMQ, Kafka) | Assíncrono, eventos, resiliência |
| **WebSocket** | Tempo real (chat, notifications) |

### Sync vs Async

- **Síncrono**: cliente espera resposta. Simples mas **acopla** serviços
- **Assíncrono**: dispara evento e segue. **Desacopla** mas precisa lidar com falhas eventualmente consistentes

---

## ✅ Resumo da página

- Arquitetura é sobre **fronteiras**, **dependências** e **responsabilidades**
- **SOLID, DRY, KISS, YAGNI** são princípios — não regras absolutas
- **Comece monolito**; só migre pra microserviços se realmente justificar
- **Clean Architecture** e **Hexagonal** são a mesma ideia: dependências apontam pro núcleo
- **DDD** modela código com base no negócio, não em estruturas técnicas
- **Bounded Contexts** evitam que um modelo gigante tente servir tudo
- **CQRS + Event Sourcing** são poderosos mas caros — só pra casos certos
- **REST**, **gRPC**, **mensageria** — escolha pelo problema, não pela moda

---

⬅️ [Anterior: Design Patterns](./14-design-patterns.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: DevOps e CI/CD](./16-devops-cicd.md)
