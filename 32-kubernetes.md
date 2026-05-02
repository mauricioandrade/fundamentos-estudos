# 🐳 Kubernetes Profundo

> Você já ouviu falar de Kubernetes (K8s) no [tópico de DevOps](./16-devops-cicd.md). Aqui vamos a fundo: como ele realmente funciona, manifests, operators, Helm. Não pra virar SRE, mas pra **entender** o que está rodando quando seu app vai pra produção em K8s.

---

## 🤔 Por que K8s existe?

Antes do K8s:
- 🐳 Você usa Docker (legal!)
- 📈 Mas tem 50 containers em 10 servidores
- 🤔 Quem decide qual container vai onde? Quem reinicia se cair? Quem balanceia?

K8s resolve isso. É um **orquestrador** de containers.

> 💡 **Em palavras simples**: K8s é o "gerente" da sua frota de containers. Você diz "quero 5 cópias desse app rodando", ele cuida de subir, manter vivos, distribuir entre servidores e substituir os que caem.

### Quando K8s **não** vale

- ❌ Projeto pequeno (Render, Railway, Fly.io são mais simples)
- ❌ Time pequeno sem expertise (curva de aprendizado é íngreme)
- ❌ App monolítico estável (containers já bastam)

### Quando K8s **vale**

- ✅ Múltiplos serviços precisando coordenar
- ✅ Escala variável (auto-scaling)
- ✅ Time com cultura DevOps madura
- ✅ Multi-cloud / on-premise (portabilidade)

---

## 🏗️ Arquitetura

K8s tem 2 partes principais:

```
┌──────────────────────────────────┐
│      CONTROL PLANE (cérebro)     │
│  ┌──────────┐ ┌──────────────┐   │
│  │ API      │ │ etcd         │   │
│  │ Server   │ │ (estado)     │   │
│  └──────────┘ └──────────────┘   │
│  ┌──────────┐ ┌──────────────┐   │
│  │Scheduler │ │ Controllers  │   │
│  └──────────┘ └──────────────┘   │
└──────────────────────────────────┘
          │ comanda
          ↓
┌──────────────────────────────────┐
│      WORKER NODES (músculos)     │
│                                  │
│  Node 1: [Pod] [Pod] [Pod]       │
│  Node 2: [Pod] [Pod]             │
│  Node 3: [Pod] [Pod] [Pod] [Pod] │
└──────────────────────────────────┘
```

| Componente | Função |
|---|---|
| **API Server** | Único ponto de entrada — você fala com ele via `kubectl` |
| **etcd** | Banco de dados que guarda todo o estado do cluster |
| **Scheduler** | Decide em qual node colocar cada Pod |
| **Controllers** | Garantem que estado real == estado desejado |
| **kubelet** | Agente em cada node que executa Pods |
| **kube-proxy** | Faz networking dos Pods |

---

## 🧱 Os Recursos Fundamentais

### 1. Pod

Menor unidade do K8s. **1 ou mais containers** que rodam juntos, compartilham rede e storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-app
spec:
  containers:
  - name: app
    image: meu-app:1.0
    ports:
    - containerPort: 8080
```

> 💡 **Geralmente 1 container por Pod**. Múltiplos containers só pra "sidecars" (proxy, log shipper, etc).

### 2. Deployment

Você raramente cria Pods diretamente. Usa **Deployment** que gerencia múltiplas réplicas + rolling updates.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meu-app
spec:
  replicas: 3                # 3 cópias
  selector:
    matchLabels:
      app: meu-app
  template:
    metadata:
      labels:
        app: meu-app
    spec:
      containers:
      - name: app
        image: meu-app:1.0
        ports:
        - containerPort: 8080
```

K8s cuida de:
- 🚀 Subir 3 Pods
- 🔄 Substituir Pods que caem
- 🆕 Rolling update quando você muda a imagem

### 3. Service

Pods são **efêmeros** (têm IPs que mudam). Service dá um **endereço estável**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: meu-app-svc
spec:
  selector:
    app: meu-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP   # interno, ou LoadBalancer pra externo
```

Tipos de Service:
- **ClusterIP** (padrão): só acessível dentro do cluster
- **NodePort**: porta exposta em cada node
- **LoadBalancer**: cria load balancer no cloud (AWS ELB, etc)
- **ExternalName**: alias DNS pra serviço externo

### 4. Ingress

Roteamento HTTP **avançado** com regras de path/host. Geralmente Nginx ou Traefik por baixo.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: meu-app-ingress
spec:
  rules:
  - host: meuapp.exemplo.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-svc
            port:
              number: 80
```

### 5. ConfigMap e Secret

Configurações e dados sensíveis separados do código.

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.host: "postgres.svc"
  log.level: "INFO"

---
# Secret (base64-encoded)
apiVersion: v1
kind: Secret
metadata:
  name: db-password
type: Opaque
data:
  password: c2VjcmV0MTIz   # "secret123" em base64
```

> ⚠️ **Secret é só base64, não criptografado**. Pra segurança real, use **Sealed Secrets** ou **External Secrets** (Vault, AWS Secrets Manager).

### 6. Namespace

Isolamento lógico. Útil pra separar ambientes (dev, staging, prod) ou times.

```bash
kubectl create namespace producao
kubectl get pods -n producao
```

---

## 🛠️ kubectl: O Canivete Suíço

```bash
# Listar recursos
kubectl get pods                       # Pods do namespace atual
kubectl get pods -n outro-ns           # De outro namespace
kubectl get pods -A                    # Todos os namespaces
kubectl get pods -o wide               # Mais detalhes

# Aplicar manifests
kubectl apply -f deployment.yaml       # Cria/atualiza
kubectl delete -f deployment.yaml      # Remove

# Inspecionar
kubectl describe pod meu-pod           # Detalhes completos
kubectl logs meu-pod                   # Logs
kubectl logs -f meu-pod                # Follow
kubectl logs --tail=100 meu-pod        # Últimas 100 linhas
kubectl exec -it meu-pod -- bash       # Entrar no container

# Debugar
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl top pods                       # Uso de CPU/RAM

# Portforward (acessar serviço local)
kubectl port-forward svc/meu-app 8080:80
```

> 💡 **Dica de produtividade**: alias `k=kubectl` no shell. Muda sua vida.

---

## 📈 Auto-scaling

### HPA (Horizontal Pod Autoscaler)

Aumenta/diminui réplicas baseado em CPU, memória ou métricas customizadas.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: meu-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: meu-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # escala se CPU > 70%
```

### Cluster Autoscaler

Adiciona/remove **nodes** quando os existentes não dão conta. Geralmente fornecido pelo cloud provider.

---

## 💾 Storage

Containers são efêmeros. Pra dados que persistem, K8s tem **Volumes**:

### PersistentVolume (PV) e PersistentVolumeClaim (PVC)

```yaml
# PVC: app pede storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 10Gi
  storageClassName: gp2  # tipo do disco (AWS)
```

```yaml
# Pod usa o PVC
spec:
  containers:
  - name: postgres
    image: postgres:16
    volumeMounts:
    - mountPath: /var/lib/postgresql/data
      name: data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: postgres-data
```

### StatefulSet

Pra apps que precisam de **identidade estável** (DBs, queues). Cada Pod tem nome fixo (`postgres-0`, `postgres-1`).

> 💡 **Banco em K8s?** Polêmico. Muitos preferem usar bancos **gerenciados** (RDS, Cloud SQL) e deixar K8s só pros apps stateless.

---

## 🎯 Padrões e Boas Práticas

### Liveness e Readiness Probes

K8s precisa saber se seu Pod está saudável e pronto.

```yaml
spec:
  containers:
  - name: app
    image: meu-app:1.0
    livenessProbe:        # se falhar, K8s reinicia
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:       # se falhar, K8s tira do load balancer
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
```

> 🤔 **Diferença**: liveness reinicia o Pod, readiness só tira do tráfego (sem reiniciar).

### Resource Requests e Limits

```yaml
resources:
  requests:    # mínimo que K8s reserva
    cpu: 250m      # 0.25 CPU
    memory: 256Mi
  limits:      # máximo que pode usar
    cpu: 1000m     # 1 CPU
    memory: 1Gi
```

> ⚠️ **Cuidado**: se Pod ultrapassa o limit de memória, é morto (OOMKilled). CPU é "throttled" (limitado, não morto).

### Labels e Selectors

Labels são metadata key-value. Selectors usam labels pra encontrar recursos.

```yaml
metadata:
  labels:
    app: meu-app
    env: producao
    team: pagamentos
```

```bash
kubectl get pods -l app=meu-app,env=producao
```

---

## 📦 Helm: Package Manager do K8s

Imagine instalar Postgres, Redis, Kafka... cada um com 5+ manifests YAML. Vira insano.

**Helm** é o "npm/apt do K8s". Você instala "charts" (pacotes).

```bash
# Adicionar repositório
helm repo add bitnami https://charts.bitnami.com/bitnami

# Instalar Postgres
helm install meu-postgres bitnami/postgresql \
  --set auth.postgresPassword=secret123 \
  --set primary.persistence.size=20Gi

# Listar instalações
helm list

# Atualizar
helm upgrade meu-postgres bitnami/postgresql --set ...

# Desinstalar
helm uninstall meu-postgres
```

### Criar seu próprio chart

```bash
helm create meu-chart
```

Estrutura:
```
meu-chart/
├── Chart.yaml          # metadados
├── values.yaml         # valores padrão (configuração)
├── templates/          # YAMLs com placeholders
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── charts/             # dependências
```

> 💡 **Helm é praticamente obrigatório** em K8s sério. Versionamento, rollback, configuração — tudo via Helm.

---

## 🤖 Operators: K8s Estendido

Operator é um **controller customizado** que gerencia uma aplicação específica.

### Por que existem?

K8s nativo entende Pods, Services, etc. Mas e gerenciar **um cluster Postgres** com replicação, backup, failover? Isso é específico de Postgres.

Um **Postgres Operator** ensina K8s a fazer isso. Você cria um recurso `PostgresCluster`, ele cuida de tudo.

```yaml
apiVersion: postgres-operator.crunchydata.com/v1beta1
kind: PostgresCluster
metadata:
  name: meu-postgres
spec:
  postgresVersion: 16
  instances:
  - replicas: 3
  backups:
    pgbackrest:
      schedules:
        full: "0 1 * * 0"   # backup completo aos domingos 1h
```

### Operators famosos

- 🐘 **Postgres**: Crunchy, Zalando
- 🔴 **Redis**: OT, Spotahome
- 📨 **Kafka**: Strimzi
- 📊 **Prometheus**: prometheus-operator
- 🔧 **Cert-Manager**: gerencia certificados TLS automaticamente

> 💡 **CRD (Custom Resource Definition)** é o que permite criar tipos próprios (`PostgresCluster`). Operators são CRDs + controllers.

---

## 🔐 Segurança em K8s

### RBAC (Role-Based Access Control)

Quem pode fazer o que.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: producao
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### NetworkPolicy

Firewall entre Pods. Por padrão tudo conversa com tudo — geralmente você quer isolar.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

Acima: só Pods com `app=frontend` podem falar com `app=api`.

### Pod Security Standards

Restrições de segurança em Pods (não rodar como root, não usar host network, etc).

---

## 🌐 Service Mesh (revisitando)

Em apps com muitos microserviços, Service Mesh adiciona:
- 🔒 mTLS automático
- 🔄 Retry, timeout, circuit breaker
- 📊 Observabilidade
- 🚦 Canary deploys

**Implementações**: Istio, Linkerd, Consul Connect.

> ⚠️ **Adiciona complexidade**. Só vale a pena com 10+ microserviços.

---

## 🎯 K8s Distribuições e Onde Rodar

| Distribuição | Quando |
|---|---|
| **Minikube / kind** | Desenvolvimento local |
| **k3s** | Edge, IoT, raspberry pi |
| **EKS / GKE / AKS** | Cloud gerenciado |
| **OpenShift** | Empresas (Red Hat) |
| **Rancher** | Multi-cluster, on-premise |

> 💡 **Pra começar a aprender**: `kind` (KinD = Kubernetes in Docker). Sobe um cluster local em segundos.

---

## ✅ Resumo da página

- K8s **orquestra containers** em escala — só vale a pena com complexidade real
- **Control plane** (API server, etcd, scheduler, controllers) + **worker nodes** (kubelet, kube-proxy)
- Recursos: **Pod, Deployment, Service, Ingress, ConfigMap, Secret, Namespace**
- **kubectl** é a CLI principal — domine `get`, `apply`, `logs`, `exec`, `describe`
- **HPA** escala Pods, **Cluster Autoscaler** escala nodes
- **PV/PVC + StatefulSets** pra storage persistente
- **Liveness/Readiness probes** + **resource requests/limits** são obrigatórios em produção
- **Helm** = package manager (use sempre)
- **Operators** estendem K8s pra apps complexas (DB, Kafka)
- **RBAC, NetworkPolicy, Pod Security** pra segurança
- **Service Mesh** (Istio, Linkerd) pra apps com muitos microserviços

---

⬅️ [Anterior: Integração com APIs de LLMs](./31-integracao-llms.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: MLOps](./33-mlops.md)
