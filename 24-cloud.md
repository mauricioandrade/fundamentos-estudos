# ☁️ Cloud: Fundamentos

> "Cloud" não é mágica nem servidor mágico — é o data center de outra pessoa, alugado por hora, com APIs pra você criar e destruir recursos sob demanda. Entender os conceitos vale pra qualquer provider (AWS, GCP, Azure).

---

## 🤔 O Que é Cloud Computing

### Antes da cloud

Você queria rodar um app:
1. Compra servidor (R$ 20.000)
2. Aluga rack em data center (R$ 1.500/mês)
3. Configura tudo manualmente
4. Aguenta a infra mesmo se app morrer
5. Precisa escalar? Compra mais servidor + 4 semanas pra chegar

### Com a cloud

```bash
aws ec2 run-instances --image-id ami-xyz --instance-type t3.medium
# Servidor pronto em 30 segundos
```

Paga por hora/segundo de uso. Termina o uso, destrói. Custos viram **operacionais**, não capex.

> 💡 **Em palavras simples**: cloud é Uber pra servidores. Você não compra carro — usa quando precisa, paga pelo uso, e nunca pensa em manutenção.

---

## 📊 Modelos de Serviço

### 🖥️ IaaS (Infrastructure as a Service)

Você aluga **infraestrutura**: VM, disco, rede.

- **AWS**: EC2, EBS, VPC
- **GCP**: Compute Engine
- **Azure**: Virtual Machines

✅ Controle total do OS e stack
❌ Você gerencia tudo (patches, segurança, escala)

### 🏗️ PaaS (Platform as a Service)

Você sobe **código**, a plataforma cuida do resto.

- **AWS**: Elastic Beanstalk, App Runner
- **GCP**: App Engine, Cloud Run
- **Azure**: App Service
- **Outros**: Heroku, Render, Railway

✅ Sem dor de cabeça com infra
❌ Menos controle, vendor lock-in

### 📦 SaaS (Software as a Service)

Software pronto pra usar.

- Gmail, Notion, Slack, GitHub, Datadog

### ⚡ FaaS / Serverless (Function as a Service)

Você sobe **funções**, executam sob demanda.

- **AWS**: Lambda
- **GCP**: Cloud Functions
- **Azure**: Functions
- **Cloudflare**: Workers

```javascript
// Lambda em Node.js
exports.handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: 'Olá!' })
  };
};
```

✅ Paga só por execução (zero custo se não rodar)
✅ Escala automaticamente
❌ **Cold starts** (primeira chamada lenta)
❌ Limites de tempo de execução
❌ Vendor lock-in

---

## 🌍 Conceitos Centrais (todos os providers)

### Region

Localização geográfica dos data centers. Ex: `us-east-1` (Virginia), `sa-east-1` (São Paulo), `eu-west-1` (Irlanda).

### Availability Zone (AZ)

Data center isolado dentro de uma region. Cada region tem 2-6 AZs.

```
Region us-east-1
├── AZ us-east-1a (data center 1)
├── AZ us-east-1b (data center 2)
└── AZ us-east-1c (data center 3)
```

> 💡 **Multi-AZ deployment**: app rodando em 2+ AZs sobrevive ao outage de um data center inteiro.

### VPC (Virtual Private Cloud)

Sua "rede privada" dentro da cloud. Você define ranges de IP, subnets, firewall rules.

```
VPC 10.0.0.0/16
├── Public Subnet 10.0.1.0/24    (com internet gateway)
└── Private Subnet 10.0.2.0/24   (sem internet direta)
```

### IAM (Identity and Access Management)

Quem pode fazer o quê. Crítico pra segurança.

```json
// Política que permite ler de um bucket S3 específico
{
  "Effect": "Allow",
  "Action": ["s3:GetObject"],
  "Resource": "arn:aws:s3:::meu-bucket/*"
}
```

> 🛡️ **Princípio do menor privilégio**: nunca dê `*` (todas as permissões). Liste explícito.

---

## 🛠️ Serviços Essenciais (com paralelos AWS/GCP/Azure)

### Computação

| Tipo | AWS | GCP | Azure |
|---|---|---|---|
| **VM** | EC2 | Compute Engine | Virtual Machines |
| **Container gerenciado** | ECS, EKS | GKE, Cloud Run | AKS, Container Apps |
| **Serverless** | Lambda | Cloud Functions | Functions |
| **PaaS** | Elastic Beanstalk | App Engine | App Service |

### Armazenamento

| Tipo | AWS | GCP | Azure |
|---|---|---|---|
| **Object Storage** | S3 | Cloud Storage | Blob Storage |
| **Block (HD)** | EBS | Persistent Disk | Managed Disks |
| **File** | EFS | Filestore | Files |

### Banco de dados

| Tipo | AWS | GCP | Azure |
|---|---|---|---|
| **SQL gerenciado** | RDS, Aurora | Cloud SQL | Azure SQL |
| **NoSQL gerenciado** | DynamoDB | Firestore, Bigtable | Cosmos DB |
| **Cache** | ElastiCache | Memorystore | Redis Cache |
| **Data warehouse** | Redshift | BigQuery | Synapse |

### Rede e CDN

| Tipo | AWS | GCP | Azure |
|---|---|---|---|
| **Load Balancer** | ELB | Cloud LB | Load Balancer |
| **CDN** | CloudFront | Cloud CDN | Front Door |
| **DNS** | Route 53 | Cloud DNS | DNS |
| **VPN** | Site-to-Site VPN | Cloud VPN | VPN Gateway |

### Mensageria

| Tipo | AWS | GCP | Azure |
|---|---|---|---|
| **Fila** | SQS | Pub/Sub | Service Bus |
| **Pub/Sub** | SNS | Pub/Sub | Event Grid |
| **Streaming** | Kinesis | Pub/Sub | Event Hubs |

---

## 💰 Custos: Onde Cloud "Mata"

Cloud parece barato no início, vira armadilha sem cuidado.

### Anti-patterns caros

#### 1. Recursos esquecidos
```
Você cria VM pra teste → esquece → R$ 200/mês por anos
```

#### 2. Egress (saída de dados)
**Entrada na cloud é grátis**. **Saída custa**. Mover 1 TB de dados pode custar US$ 100+.

#### 3. Bancos sempre ligados
RDS pequeno = US$ 100+/mês mesmo sem uso.

#### 4. NAT Gateway
Necessário pra subnets privadas, mas custa caro (~$32/mês + tráfego).

#### 5. Logs e monitoring
Datadog, CloudWatch detalhado, X-Ray podem custar mais que a app.

### Como economizar

- 💸 **Reserved Instances / Savings Plans**: comprometimento de 1-3 anos = 30-70% off
- 🌊 **Spot Instances**: capacidade ociosa, super barata, mas pode ser interrompida
- 🛑 **Auto-shutdown** de ambientes de dev fora do horário comercial
- 📊 **Cost Explorer / Billing Alerts**: alertas quando custos sobem
- 🏷️ **Tags**: rotule todos os recursos pra rastrear gastos

> 💡 **Ferramentas de FinOps**: AWS Cost Explorer, GCP Billing, Vantage, Spot.io.

---

## 🏗️ Padrões Arquiteturais

### 1. Multi-AZ

App em 2+ availability zones pra alta disponibilidade.

```
        Load Balancer
        /          \
   AZ-1a          AZ-1b
   (App + DB)    (App + DB replica)
```

### 2. Multi-Region

Pra disaster recovery ou usuários globais.

```
Região US-EAST          Região SA-EAST (replica)
   App + DB primário  ←→  App + DB read replica
```

### 3. Microsserviços com Service Mesh

Cada microserviço em container, gerenciado por Kubernetes (EKS/GKE/AKS), com Istio/Linkerd pra tráfego.

### 4. Serverless / Event-Driven

```
S3 upload → Lambda → DynamoDB → SNS → Email
       (event-driven, escala automático)
```

---

## 🚀 Como começar (sem se afogar)

### Passo 1: conceitos antes de provider

Aprenda **VPC, IAM, computação, storage** em **um** provider. Não tente AWS + GCP + Azure de uma vez.

### Passo 2: AWS é o mais usado

Maior market share, mais documentação, maior ecossistema. Mas **GCP é mais simples** pra começar — UX melhor.

### Passo 3: certificações úteis

| Provider | Certificação inicial |
|---|---|
| **AWS** | Cloud Practitioner → Solutions Architect Associate |
| **GCP** | Associate Cloud Engineer |
| **Azure** | AZ-900 → AZ-104 |

> 💡 **Cloud Practitioner (AWS) é genérica e barata** — bom pra demonstrar conhecimento básico no LinkedIn.

### Passo 4: free tier

Todos os providers têm tier grátis. Cuidado com limites pra não tomar susto na fatura.

### Passo 5: Infrastructure as Code (IaC)

Já visto no [tópico de DevOps](./16-devops-cicd.md). Em vez de clicar no console, escreva infra como código:

```hcl
# Terraform
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t3.micro"
  tags = { Name = "web-server" }
}
```

Versionado no Git, reproduzível, auditável.

---

## ⚠️ Vendor Lock-in: Cuidado

Cloud nativa = produtividade. Mas se você usar **só serviços proprietários** (DynamoDB, Lambda, SQS), migrar é doloroso.

### Estratégias pra reduzir lock-in

- ✅ Use serviços compatíveis com padrões abertos (PostgreSQL > Aurora, Kafka > Kinesis)
- ✅ Containers são portáveis (Docker rola em qualquer cloud ou on-premise)
- ✅ Kubernetes abstrai bem (mas exige expertise)
- ✅ IaC com Terraform (multi-cloud) > CloudFormation (só AWS)

---

## 🌱 Tópicos Avançados (pra estudar depois)

- 🕸️ **Service Mesh** (Istio, Linkerd) — gerenciamento de tráfego entre microserviços
- 🐳 **Kubernetes managed** (EKS, GKE, AKS) — orquestração em escala
- 📊 **Observabilidade** cloud-native (CloudWatch, Stackdriver, Application Insights)
- 🔄 **Event-Driven Architecture** — EventBridge, Pub/Sub, Event Grid
- 🤖 **MLOps na cloud** — SageMaker, Vertex AI, Azure ML

---

## ✅ Resumo da página

- Cloud = data center alugado com APIs e cobrança por uso
- **Modelos**: IaaS (controle), PaaS (conveniência), Serverless (sem servidor), SaaS (pronto)
- Conceitos universais: **Region, AZ, VPC, IAM**
- AWS, GCP, Azure têm **paralelos pra todos os serviços comuns**
- **Custos crescem rápido** sem cuidado — monitore desde o dia 1
- Padrões arquiteturais: Multi-AZ, Multi-Region, Serverless event-driven
- Comece focado em **um provider**; AWS é mais comum no mercado
- Use **IaC (Terraform)** pra reproduzibilidade
- Cuidado com **vendor lock-in** — use padrões abertos quando possível

---

⬅️ [Anterior: Performance](./23-performance.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Sistemas Distribuídos](./25-sistemas-distribuidos.md)
