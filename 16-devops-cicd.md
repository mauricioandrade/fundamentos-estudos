# 🚀 DevOps e CI/CD

> "DevOps" é cultura + ferramentas que conectam **desenvolvimento** e **operação**. CI/CD é a parte mais visível: automação do caminho do código até produção.

---

## 🤔 O Problema que DevOps Resolve

Antigamente:
- **Devs**: "no meu PC funciona!"
- **Ops**: "deploy quebrou de novo, culpa do código"
- Deploy era uma vez por mês, demorava horas, vinha cheio de bugs

DevOps quebra esse muro:
- Devs e ops trabalham juntos
- Deploy vira **automatizado, frequente e confiável**
- Ambiente de produção é **reproduzível**

---

## 🔄 CI vs CD

### CI - Continuous Integration

**Toda vez que você commita, o código é testado automaticamente.**

```
push → checkout → install → build → test → lint → ✅ ou ❌
```

✅ Detecta bugs cedo
✅ Garante que branches mergem sem conflito
✅ Padroniza qualidade

### CD - Continuous Delivery vs Continuous Deployment

| Termo | O que faz |
|---|---|
| **Continuous Delivery** | Pipeline pronto pra deploy, mas exige **clique manual** |
| **Continuous Deployment** | Cada commit que passa nos testes vai pra **produção automaticamente** |

> 💡 Maioria das empresas faz Continuous Delivery — o controle humano ainda é importante pra produção.

---

## 🐙 GitHub Actions: O Básico

Exemplo de pipeline pra um projeto Node.js:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

### Conceitos

- **Workflow**: arquivo YAML em `.github/workflows/`
- **Job**: conjunto de steps rodando numa VM
- **Step**: comando individual ou uso de uma "action" (plugin)
- **Trigger**: o que dispara (push, PR, schedule, manual)

### Truques úteis

#### Matrix: testar em várias versões

```yaml
jobs:
  test:
    strategy:
      matrix:
        node: [18, 20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

#### Secrets

```yaml
- run: ./deploy.sh
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

Configurados em **Settings → Secrets** do repo.

#### Cache

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
```

> 💡 Cache acelera muito builds. Maven, Gradle, npm, pip — todos têm cache que vale configurar.

---

## 🐳 Docker: Empacotando Aplicações

### O problema

Sua aplicação Spring Boot roda no seu PC com Java 17, Postgres 14, Linux. No servidor: Java 11, Postgres 12, outro Linux. **Quebra.**

### A solução

**Container** = aplicação + todas dependências + sistema operacional mínimo, empacotado como uma "imagem". Roda **igualzinho em qualquer lugar** que tenha Docker.

```
Imagem (template) → Container (instância rodando)
```

### Dockerfile básico (Spring Boot)

```dockerfile
FROM eclipse-temurin:17-jdk-alpine

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Multi-stage build (recomendado)

Reduz tamanho final da imagem:

```dockerfile
# Stage 1: build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: runtime (só o jar, sem Maven nem código fonte)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Comandos essenciais

```bash
docker build -t minha-app .              # cria imagem
docker run -p 8080:8080 minha-app        # roda
docker ps                                 # containers rodando
docker logs <container-id>                # ver logs
docker exec -it <container-id> sh         # entrar no container
docker stop <container-id>                # parar
```

---

## 🎼 Docker Compose

Pra rodar **múltiplos containers juntos** (app + DB + redis):

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: postgres
    depends_on:
      - postgres

  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  pgdata:
```

```bash
docker compose up -d        # sobe tudo em background
docker compose down         # derruba tudo
docker compose logs -f app  # logs do serviço app
```

> 💡 **Use Docker Compose pra ambiente local de desenvolvimento.** Em produção você geralmente usa Kubernetes ou plataformas como Render/Railway, mas pra rodar tudo na sua máquina (app + banco + cache + queue), Docker Compose é o padrão.

---

## ☸️ Kubernetes (visão geral)

Pra **produção em escala**, Docker Compose não basta. Aí entra Kubernetes (K8s).

### Conceitos

| Conceito | O que é |
|---|---|
| **Pod** | Menor unidade — 1 ou mais containers que rodam juntos |
| **Deployment** | Define quantas réplicas de um pod manter |
| **Service** | Abstração de rede pra expor pods |
| **Ingress** | Roteamento de tráfego HTTP externo |
| **ConfigMap / Secret** | Configuração e dados sensíveis |
| **Namespace** | Isolamento lógico (dev, staging, prod) |

> ⚠️ **K8s tem curva enorme**. Pra projetos pequenos, é overkill. Render, Railway, Fly.io abstraem isso pra você.

### Plataformas que abstraem

- **Render** — deploy simples, escala fácil, bom DX
- **Railway** — DX moderno, integração com GitHub
- **Fly.io** — deploy global em edge
- **Vercel** — frontend e serverless
- **Heroku** — clássico, ainda funciona

---

## 🌍 Estratégias de Deploy

### Big Bang Deploy

Para tudo, troca, sobe tudo de novo. **Downtime** garantido.

### Rolling Update

Atualiza instâncias gradualmente. **Sem downtime**, mas duas versões coexistem por um tempo.

```
[v1] [v1] [v1] [v1]
       ↓
[v1] [v1] [v1] [v2]
       ↓
[v1] [v1] [v2] [v2]
       ↓
[v1] [v2] [v2] [v2]
       ↓
[v2] [v2] [v2] [v2]
```

### Blue-Green Deploy

Dois ambientes idênticos. Promove o novo trocando o roteamento.

```
Antes:                    Depois:
Tráfego → [Blue v1]       Tráfego → [Green v2]
          [Green v2]                 [Blue v1] (standby)
```

### Canary Deploy

Manda **% pequeno** de tráfego pra nova versão, monitora, expande.

```
95% → v1
5%  → v2 (canário)
```

✅ Detecta bugs com baixo impacto

---

## 🏗️ Infrastructure as Code (IaC)

Definir infraestrutura em **código versionado**, não em cliques na AWS.

### Ferramentas

| Ferramenta | O que faz |
|---|---|
| **Terraform** | Padrão da indústria, multi-cloud |
| **Pulumi** | Como Terraform, mas em linguagem real (TS, Python) |
| **Ansible** | Configuração de servidores |
| **CloudFormation** | Específico AWS |

```hcl
# Exemplo Terraform: criar S3 bucket
resource "aws_s3_bucket" "backups" {
  bucket = "my-app-backups"
  tags = {
    Environment = "production"
  }
}
```

> 💡 **Por que IaC?** Reproduzibilidade, versionamento, code review, recuperação de desastres.

---

## 🔍 12-Factor App

Princípios pra apps cloud-native modernas. Vale memorizar:

1. **Codebase**: 1 repo, vários deploys
2. **Dependencies**: declaradas explicitamente
3. **Config**: em variáveis de ambiente
4. **Backing services**: tratados como recursos plugáveis (DB, queue)
5. **Build, release, run**: estágios separados
6. **Processes**: stateless
7. **Port binding**: app exporta porta, não depende de servidor injetado
8. **Concurrency**: escala horizontal
9. **Disposability**: starts/stops rápidos
10. **Dev/prod parity**: ambientes parecidos
11. **Logs**: streams de eventos (stdout)
12. **Admin processes**: tarefas administrativas como processos one-off

> 🤔 **Quando seguir?** Quando deployar em cloud (Render, Railway, Heroku, K8s). É a base do design.

---

## 📦 Versionamento Semântico (SemVer)

Padrão pra versionar releases:

```
MAJOR.MINOR.PATCH
  3.   1.   2
```

| Parte | Quando incrementar |
|---|---|
| **MAJOR** | Mudança que **quebra compatibilidade** |
| **MINOR** | Nova funcionalidade compatível |
| **PATCH** | Bugfix |

Pre-releases: `2.0.0-beta.1`, `2.0.0-rc.1`

---

## ✅ Resumo da página

- **CI** = testes automáticos a cada commit; **CD** = deploy automático
- **GitHub Actions** automatiza pipelines via YAML em `.github/workflows/`
- **Docker** empacota app + dependências = "roda igual em qualquer lugar"
- **Docker Compose** orquestra múltiplos containers locais
- **Kubernetes** é pra escala — pra projetos pequenos, use plataformas (Render, Railway)
- Estratégias de deploy: **rolling**, **blue-green**, **canary**
- **IaC** (Terraform, Pulumi) versiona infra como código
- **12-Factor** é a checklist pra apps cloud-native modernas
- **SemVer** padroniza versões: MAJOR.MINOR.PATCH

---

⬅️ [Anterior: Arquitetura de Software](./15-arquitetura-software.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Testes](./17-testes.md)
