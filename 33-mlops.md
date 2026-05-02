# 🧪 MLOps: ML em Produção

> Treinar modelo num notebook é fácil. **Manter modelo em produção** atendendo milhões de requests, com qualidade, monitoramento e atualização contínua — isso é MLOps. É a interseção entre Machine Learning, DevOps e Engenharia de Dados.

---

## 🤔 Por que MLOps existe?

ML clássico tem problemas que DevOps tradicional não resolve:

### Problema 1: Modelos "se deterioram" sozinhos

Você treinou um modelo de detecção de fraude em 2024. Em 2026:
- Padrões de fraude mudaram (criminosos evoluem)
- Comportamento dos usuários mudou
- O modelo continua o mesmo, mas a realidade não

**Resultado**: precisão cai com o tempo. Isso é **drift**.

### Problema 2: Reprodutibilidade

Cientista de dados treina modelo com:
- Dados de uma data específica
- Versão X de uma biblioteca
- Notebook com 500 células executadas em ordem aleatória

Como reproduzir isso 6 meses depois? **Quase impossível** sem MLOps.

### Problema 3: Deploy de modelos é diferente

App tradicional: código + dependências.
ML: código + dependências + **modelo treinado** + **dados de treino** + **transformações** + **features**.

Cada um precisa ser versionado, testado, monitorado.

---

## 🎯 O Ciclo de MLOps

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Coleta de    │ ──→ │ Preparação   │ ──→ │ Treinamento  │
│ dados        │     │ + features   │     │ + tuning     │
└──────────────┘     └──────────────┘     └──────────────┘
                                                  │
                                                  ↓
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Monitoramento│ ←── │ Deploy       │ ←── │ Avaliação    │
│ + drift      │     │ (inferência) │     │ + validação  │
└──────────────┘     └──────────────┘     └──────────────┘
        │
        └─────────────→ Re-treino quando necessário
```

> 💡 **Em palavras simples**: MLOps faz pra modelos o que CI/CD faz pra código — automatiza, versiona, monitora e mantém.

---

## 📊 Versionamento: O Que Precisa ser Versionado

Em ML, você versiona **muito mais coisas** que em código tradicional:

| O que | Ferramenta típica |
|---|---|
| 💻 **Código** | Git (você já conhece) |
| 📊 **Dados** | DVC, LakeFS, Delta Lake |
| 🤖 **Modelos** | MLflow, Weights & Biases |
| 🧪 **Experimentos** | MLflow, Comet, W&B |
| 🔧 **Features** | Feast, Tecton (feature stores) |
| 📐 **Configurações** | Hydra, OmegaConf |

### DVC (Data Version Control)

Git pra dados. Você commita um "ponteiro" leve, dados ficam em S3/GCS.

```bash
# Inicializa
dvc init
dvc remote add -d storage s3://meu-bucket/dvc

# Versiona dataset
dvc add data/train.csv
git add data/train.csv.dvc .gitignore
git commit -m "data: dataset inicial"

# Sobe pro storage
dvc push
```

### MLflow Tracking

Registra cada experimento (parâmetros, métricas, modelo gerado).

```python
import mlflow

with mlflow.start_run():
    mlflow.log_param("learning_rate", 0.01)
    mlflow.log_param("epochs", 100)

    # ... treina ...

    mlflow.log_metric("accuracy", 0.92)
    mlflow.log_metric("f1", 0.89)
    mlflow.sklearn.log_model(modelo, "modelo")
```

Resultado: dashboard com **todos os experimentos**, comparação lado a lado, capacidade de "reviver" qualquer um.

---

## 🚀 Deploy de Modelos

Diferente de deployar API tradicional. Há padrões:

### 1. Modelo embutido na app

```python
# Carrega modelo no startup
model = joblib.load("model.pkl")

@app.post("/predict")
def predict(data):
    return model.predict(data)
```

✅ Simples
❌ Modelo grande infla a imagem
❌ Atualizar modelo = redeploy da app

### 2. Model Server dedicado

App chama um servidor de modelos via API.

**Tools**: TensorFlow Serving, TorchServe, BentoML, Triton, KServe.

```
App (Java) ──HTTP──→ Model Server (Python) ──→ Modelo
```

✅ Times separados (backend vs ML)
✅ Modelos atualizam independente
✅ Otimizações de inferência (batching, GPU)
❌ Latência extra de rede

### 3. Serverless / Cloud

- AWS SageMaker Endpoints
- Google Vertex AI Predictions
- Azure ML Endpoints

✅ Escala automática
✅ Sem gerenciar infra
❌ Vendor lock-in
❌ Custos podem disparar

### 4. Edge / On-device

Modelo roda no celular ou IoT (TensorFlow Lite, Core ML, ONNX).

✅ Latência zero, sem internet
❌ Modelos precisam ser pequenos
❌ Atualizar é mais difícil

---

## 🔄 Estratégias de Deploy

Mesmo do [tópico de DevOps](./16-devops-cicd.md), aplicadas a modelos:

### Shadow Mode

Modelo novo recebe tráfego **em paralelo** ao antigo, mas só pra observação. Não afeta usuário.

```
Request → [Modelo v1 (resposta real)] → Cliente
       └→ [Modelo v2 (sombra)] → Logs
```

✅ Testa em produção sem risco
✅ Compara performance real com a do antigo

### A/B Testing

X% do tráfego no v1, Y% no v2. Mede métricas de negócio.

### Canary

Como em DevOps. Começa com 5% de tráfego no novo, sobe gradualmente.

### Multi-armed Bandit

Algoritmo que **automaticamente** distribui mais tráfego pro modelo que está performando melhor.

---

## 📉 Monitoramento de Modelos

Diferente de monitorar app tradicional. Você quer detectar **degradação silenciosa**.

### O que monitorar

#### 1. Métricas de modelo

- Acurácia, precisão, recall em produção
- Latência de inferência
- Distribuição das predições

#### 2. Data Drift

A distribuição dos **inputs** mudou comparado com o treino?

**Exemplo**: modelo treinado com idade média 35. Em produção, idade média virou 50. Isso é drift — o modelo provavelmente está pior do que foi avaliado.

```python
# Detecção simples (Kolmogorov-Smirnov test)
from scipy.stats import ks_2samp

stat, p_value = ks_2samp(idades_treino, idades_producao)
if p_value < 0.05:
    print("⚠️ Distribuição mudou!")
```

#### 3. Concept Drift

A relação entre **input e output** mudou?

**Exemplo**: durante a pandemia, padrões de compra mudaram drasticamente. Modelos de recomendação treinados antes ficaram inúteis.

#### 4. Performance Drift

Acurácia caindo. Pode ser causa de qualquer um dos drifts acima.

### Ferramentas de monitoramento

- **Evidently AI** (open source, ótimo)
- **Arize AI** (paga, completa)
- **WhyLabs** (paga)
- **Fiddler**
- **Datadog ML Monitoring**

---

## 🤖 CI/CD pra ML (CT — Continuous Training)

Pipeline automatizado que **re-treina** quando necessário:

```
1. Schedule diário ou trigger por drift
   ↓
2. Pega dados novos
   ↓
3. Treina modelo automaticamente
   ↓
4. Avalia (vs baseline atual)
   ↓
5. Se melhor: deploy automático
   Se pior: alerta e investiga
```

### Ferramentas de pipeline

- **Kubeflow Pipelines** (K8s-native)
- **Apache Airflow** (genérico, popular)
- **Prefect**
- **Dagster**
- **AWS Step Functions**

```python
# Exemplo Airflow simplificado
from airflow import DAG
from airflow.operators.python import PythonOperator

with DAG('treinar_modelo', schedule='@daily'):
    extrair = PythonOperator(task_id='extrair', python_callable=extrair_dados)
    treinar = PythonOperator(task_id='treinar', python_callable=treinar_modelo)
    avaliar = PythonOperator(task_id='avaliar', python_callable=avaliar_modelo)
    deploy = PythonOperator(task_id='deploy', python_callable=deploy_se_melhor)

    extrair >> treinar >> avaliar >> deploy
```

---

## 🏪 Feature Store

Problema clássico: features (variáveis derivadas) são calculadas **diferente** no treino e em produção. Resultado: modelo prevê mal.

**Feature Store** centraliza features:
- ✅ Mesma lógica em treino e inferência
- ✅ Reutilização entre modelos
- ✅ Cacheado pra inferência rápida

### Ferramentas

- **Feast** (open source)
- **Tecton** (paga, completa)
- **Hopsworks**
- **AWS SageMaker Feature Store**

```python
# Feast — registrando feature
@feature_view
def usuario_features(usuario):
    return {
        "idade": usuario.idade,
        "ultimas_compras_30d": count_compras(usuario, days=30),
        "ticket_medio": media_compras(usuario)
    }

# Mesma feature usada em treino e produção
```

---

## 🎛️ Experiment Tracking

Cientistas treinam **muitos** modelos. Como gerenciar?

### MLflow (mais popular open source)

```python
mlflow.start_run(run_name="experiment_42")
mlflow.log_params({"lr": 0.01, "batch_size": 32})
mlflow.log_metric("accuracy", 0.92)
mlflow.log_artifact("plot.png")
```

### Weights & Biases (wandb)

Mais visual, ótimas integrações.

```python
import wandb
wandb.init(project="meu-projeto")
wandb.config = {"lr": 0.01}
wandb.log({"accuracy": 0.92})
```

### Outras

- **Comet ML**
- **Neptune**
- **TensorBoard** (built-in TensorFlow/PyTorch, mais limitado)

---

## 🛡️ Considerações de Produção

### Latência

Modelos grandes podem demorar segundos. Estratégias:
- 🔥 **Quantization**: comprimir modelo (float32 → int8)
- 🪶 **Distillation**: treinar modelo pequeno imitando o grande
- ⚡ **GPU/TPU** quando justifica
- 📦 **Batching**: processar múltiplas requests juntas

### Fallback

E se o modelo cair? Tenha plano B:
- Modelo mais simples como fallback
- Resposta default razoável
- Cache de respostas anteriores

### Reprodutibilidade

```yaml
# requirements.txt fixo
pandas==2.1.0
scikit-learn==1.3.0
numpy==1.24.3
```

Sempre fixe versões. "Funcionou na minha máquina" em ML é pesadelo.

---

## 🎯 Stack Típica de MLOps Moderno

```
┌────────────────────────────────────────┐
│  ORQUESTRAÇÃO: Airflow / Kubeflow      │
└────────────────────────────────────────┘
┌────────────────────────────────────────┐
│  EXPERIMENTAÇÃO: MLflow / W&B          │
└────────────────────────────────────────┘
┌────────────────────────────────────────┐
│  FEATURE STORE: Feast / Tecton         │
└────────────────────────────────────────┘
┌────────────────────────────────────────┐
│  VERSIONAMENTO DE DADOS: DVC / LakeFS  │
└────────────────────────────────────────┘
┌────────────────────────────────────────┐
│  MODEL SERVING: BentoML / KServe       │
└────────────────────────────────────────┘
┌────────────────────────────────────────┐
│  MONITORAMENTO: Evidently / Arize      │
└────────────────────────────────────────┘
┌────────────────────────────────────────┐
│  INFRA: Kubernetes + GPU nodes         │
└────────────────────────────────────────┘
```

---

## 💡 LLMOps: O Subcampo Emergente

Com LLMs, surge **LLMOps** — desafios específicos:

- 🪙 **Custos por token** (não por hora de servidor)
- 🎯 **Avaliação subjetiva** (texto pode ser "bom" de muitas formas)
- 🚨 **Prompt engineering** virou parte do ciclo
- 🛡️ **Segurança**: prompt injection, dados sensíveis
- 📊 **Tracing** de chamadas de tools/agents

### Ferramentas LLMOps emergentes

- **LangSmith** (LangChain)
- **Helicone**
- **Langfuse**
- **PromptLayer**
- **Weights & Biases** (também faz LLM)

---

## ✅ Resumo da página

- MLOps = DevOps + ML — necessário porque modelos **se deterioram com o tempo**
- Versione **tudo**: código, dados, modelos, features, experimentos
- Padrões de deploy: embutido, model server, serverless, edge
- Estratégias: **shadow, A/B, canary, bandits**
- Monitore **drift**: data drift, concept drift, performance drift
- **CT (Continuous Training)** automatiza re-treino
- **Feature Store** garante consistência treino/produção
- **Experiment tracking** (MLflow, W&B) é essencial
- **LLMOps** é o subcampo emergente pra LLMs em produção

---

⬅️ [Anterior: Kubernetes](./32-kubernetes.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Mobile e PWA](./34-mobile-pwa.md)
