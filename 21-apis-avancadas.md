# 🔌 APIs Avançadas: REST, GraphQL e gRPC

> Você já vai ter feito várias APIs REST simples. Mas existem nuances que separam APIs **boas** de APIs **frustrantes** — e existem alternativas (GraphQL, gRPC) que brilham em certos cenários.

---

## 🎯 REST Maduro: Além do CRUD Básico

REST é o padrão mais usado, mas tem níveis. **Richardson Maturity Model**:

| Nível | O que tem | Exemplo |
|---|---|---|
| **0** | Um endpoint, um verbo | `POST /api` com tipo da operação no body |
| **1** | Recursos | `/usuarios`, `/pedidos` (mas tudo é POST) |
| **2** | Verbos HTTP | `GET /usuarios`, `POST /usuarios`, `DELETE /usuarios/1` |
| **3** | HATEOAS | Resposta inclui links pra próximas ações |

> 💡 A maioria das APIs reais fica no **nível 2**. Nível 3 é raro e cheio de cerimônia.

### Boas práticas REST

#### Use substantivos, não verbos
```
✅ GET /usuarios/123/pedidos
❌ GET /getUserOrders?id=123
```

#### Plural pra coleções
```
✅ /usuarios, /pedidos
❌ /usuario, /pedido
```

#### Versionamento
```
/v1/usuarios   → na URL (mais comum)
Accept: application/vnd.api.v1+json   → no header (mais "limpo")
```

#### Filtros, ordenação, paginação
```
GET /usuarios?ativo=true&ordenar=-criadoEm&pagina=2&tamanho=20
```

#### Status codes corretos
```
200 OK              → sucesso (GET, PUT, PATCH)
201 Created         → criado (POST)
204 No Content      → sucesso sem body (DELETE)
400 Bad Request     → erro do cliente, payload inválido
401 Unauthorized    → não autenticado
403 Forbidden       → autenticado mas sem permissão
404 Not Found       → recurso não existe
409 Conflict        → conflito (ex: email já cadastrado)
422 Unprocessable   → validação falhou
429 Too Many Reqs   → rate limit
500 Internal Error  → erro do servidor
503 Service Unavail → serviço fora
```

### Erros padronizados (RFC 7807)

```json
{
  "type": "https://api.exemplo.com/errors/email-duplicado",
  "title": "Email já cadastrado",
  "status": 409,
  "detail": "O email mauricio@example.com já está em uso",
  "instance": "/usuarios/12345"
}
```

### Idempotência (lembra do tópico de mensageria?)

```
GET     → Sempre idempotente
PUT     → Idempotente (substitui completamente)
DELETE  → Idempotente (deletar 2x = mesma coisa)
PATCH   → Geralmente idempotente
POST    → NÃO idempotente (cada chamada cria recurso novo)
```

> 🤔 **Truque pra POST idempotente**: aceitar header `Idempotency-Key`. Se vier o mesmo key 2x, devolve a resposta cacheada.

---

## 🎨 OpenAPI / Swagger

Documentação **viva** da sua API em formato JSON/YAML.

```yaml
# openapi.yaml
paths:
  /usuarios/{id}:
    get:
      summary: Buscar usuário por ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: Usuário encontrado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Usuario'
```

**Benefícios:**
- 📖 Docs interativas (Swagger UI)
- 🤖 Geração automática de **clientes** em qualquer linguagem
- ✅ Validação de contratos
- 🧪 Testes automáticos

### Em Spring Boot

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.x</version>
</dependency>
```

Acessa `http://localhost:8080/swagger-ui.html` e tem a UI pronta.

---

## 🔥 GraphQL: Cliente Pede o Que Quer

**Problema do REST**: o backend define o formato da resposta. Se o frontend precisa só do `nome`, ainda vem o `endereco`, `telefone`, `pedidos`. Over-fetching.

**Solução do GraphQL**: cliente especifica **exatamente** o que quer.

### Comparação rápida

```http
# REST: backend manda tudo
GET /usuarios/123
{
  "id": 123,
  "nome": "Mauricio",
  "email": "...",
  "endereco": {...},
  "telefones": [...],
  "pedidos": [...],
  "preferencias": {...}
}
```

```graphql
# GraphQL: cliente pede só o que precisa
query {
  usuario(id: 123) {
    nome
    pedidos(ultimos: 5) {
      total
      data
    }
  }
}
```

Resposta:
```json
{
  "data": {
    "usuario": {
      "nome": "Mauricio",
      "pedidos": [
        { "total": 100, "data": "2026-01-15" }
      ]
    }
  }
}
```

### Estrutura do GraphQL

```graphql
type Usuario {
  id: ID!
  nome: String!
  email: String!
  pedidos(ultimos: Int): [Pedido!]!
}

type Pedido {
  id: ID!
  total: Float!
  data: String!
  usuario: Usuario!
}

type Query {
  usuario(id: ID!): Usuario
  usuarios: [Usuario!]!
}

type Mutation {
  criarUsuario(nome: String!, email: String!): Usuario!
}
```

### Quando usar GraphQL

✅ Frontend precisa de flexibilidade (mobile + web pegando dados diferentes)
✅ Múltiplos consumidores com necessidades distintas
✅ Quer evitar várias requisições (1 query → tudo)

❌ APIs simples e estáveis
❌ Time pequeno (curva de aprendizado)
❌ Caching é mais difícil (toda query é POST)

### Implementações populares

| Stack | Lib |
|---|---|
| **Java** | Spring for GraphQL, DGS (Netflix) |
| **Node.js** | Apollo Server, GraphQL Yoga |
| **Python** | Strawberry, Graphene |
| **.NET** | Hot Chocolate |
| **Go** | gqlgen |

---

## ⚡ gRPC: RPC de Alta Performance

**Google Remote Procedure Call**. Em vez de pensar em "recursos" (REST), você chama **funções** num servidor remoto, como se fossem locais.

### Como funciona

1. Define interface em **Protocol Buffers** (`.proto`)
2. Compilador gera **código cliente E servidor** automaticamente
3. Comunicação binária via HTTP/2 (super rápido)

### Exemplo

```protobuf
// usuario.proto
syntax = "proto3";

service UsuarioService {
  rpc BuscarUsuario(BuscarRequest) returns (Usuario);
  rpc CriarUsuario(CriarRequest) returns (Usuario);
  rpc ListarUsuarios(ListarRequest) returns (stream Usuario);
}

message Usuario {
  int64 id = 1;
  string nome = 2;
  string email = 3;
}

message BuscarRequest {
  int64 id = 1;
}
```

A partir desse arquivo, o `protoc` gera código pra Java, Go, Python, etc. Cliente chama `usuarioService.buscarUsuario(...)` como se fosse função local.

### Vantagens

- ⚡ **Super rápido**: HTTP/2 + binário (Protocol Buffers)
- 🤖 **Tipo-safe**: gera código tipado nas duas pontas
- 🔄 **Streaming bidirecional**: cliente e servidor mandam dados ao mesmo tempo
- 🌐 **Polyglot**: cliente em Go conversa com servidor em Java tranquilamente

### Desvantagens

- 🌐 **Ruim em browser**: HTTP/2 + binário = browser não suporta nativo (precisa gRPC-Web)
- 📖 **Menos legível**: dificil debugar com curl
- 🛠️ **Setup complexo**: precisa do compilador, geração de código

### Quando usar gRPC

✅ Comunicação **interna** entre microserviços
✅ Performance crítica (alto volume, baixa latência)
✅ Múltiplas linguagens precisam se comunicar

❌ APIs públicas pra browser
❌ Times pequenos (overhead de setup)

---

## 📊 Comparação: REST vs GraphQL vs gRPC

| Aspecto | REST | GraphQL | gRPC |
|---|---|---|---|
| **Formato** | JSON/XML | JSON | Binário (Protobuf) |
| **Protocolo** | HTTP/1.1 | HTTP | HTTP/2 |
| **Schema** | Opcional (OpenAPI) | Obrigatório | Obrigatório |
| **Browser** | ✅ Nativo | ✅ Nativo | ⚠️ Precisa gRPC-Web |
| **Streaming** | Limitado | Subscriptions | ✅ Bidirecional |
| **Performance** | OK | OK | 🔥 Excelente |
| **Cacheability** | ✅ HTTP cache funciona | ❌ Difícil | ❌ Difícil |
| **Overhead** | Baixo | Médio | Baixo |
| **Curva de aprendizado** | Baixa | Média | Alta |

---

## 🎯 Como escolher?

### REST clássico

🟢 API pública pra terceiros consumirem
🟢 Time pequeno, prazo apertado
🟢 Quer caching em CDN

### GraphQL

🟢 Frontend complexo (mobile + web + admin) com necessidades diferentes
🟢 Backend-for-Frontend (BFF)
🟢 Apollo, Relay já no stack

### gRPC

🟢 Microserviços internos
🟢 Performance crítica
🟢 Diferentes linguagens precisam se comunicar

> 💡 **Combinação comum**: REST/GraphQL pra fora (cliente), gRPC entre microserviços.

---

## 🛡️ Tópicos importantes pra qualquer API

### Rate Limiting

Limita quantas requests por usuário/minuto. Evita abuso e protege a infraestrutura.

```
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1735689600
Retry-After: 60
```

### Pagination

Pra listas grandes:

```
# Offset-based (simples)
GET /usuarios?page=3&size=20

# Cursor-based (eficiente em DBs grandes)
GET /usuarios?cursor=eyJpZCI6MTAwfQ&size=20
```

> 💡 **Cursor é melhor pra paginação profunda**: offset com `OFFSET 100000` é lentíssimo no DB.

### CORS (relembrando do tópico de Redes)

API pública precisa permitir origens externas:

```http
Access-Control-Allow-Origin: https://meusite.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Authorization, Content-Type
```

### Webhooks: Direção contrária

API normal: cliente pergunta. **Webhook**: servidor avisa o cliente quando algo acontece.

```
Cliente cadastra: "me avise em https://meu-server.com/webhook"
Quando rola evento, servidor faz: POST https://meu-server.com/webhook
```

---

## ✅ Resumo da página

- **REST nível 2** (verbos HTTP corretos) é o suficiente pra maioria dos casos
- Use **status codes corretos** e **erros padronizados** (RFC 7807)
- Documente com **OpenAPI** — gera docs interativas e clientes automáticos
- **GraphQL** brilha quando frontends precisam flexibilidade
- **gRPC** brilha entre microserviços com performance crítica
- Não existe vencedor — combine conforme o problema
- **Rate limit, paginação, CORS, webhooks** — coisas que toda API séria implementa

---

⬅️ [Anterior: Caching](./20-caching.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Segurança de Aplicação](./22-seguranca.md)
