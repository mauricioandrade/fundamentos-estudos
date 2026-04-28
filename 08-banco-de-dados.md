# 🗄️ Banco de Dados: Persistência de Dados

> Toda aplicação séria precisa guardar dados de forma confiável. Banco de dados resolve isso. Como dev backend, você passa metade do tempo lidando com queries, índices e otimização.

---

## 🤔 Por que não usar arquivos de texto?

Você poderia salvar dados num CSV ou JSON. Mas tente fazer isso com **10 milhões de usuários** e:

- 🔍 Buscar um usuário específico em <1ms
- ✏️ Atualizar 3 campos sem reescrever o arquivo todo
- 👥 Permitir 100 usuários escrevendo ao mesmo tempo sem corromper
- 🔐 Garantir que se travar no meio, nada fique inconsistente

Banco de dados resolve **todos** esses problemas. É arquivo binário otimizado + algoritmos sofisticados + protocolo de acesso concorrente.

---

## 🌳 Tipos: Relacional vs NoSQL

### 🏛️ Relacional (SQL)

Dados em **tabelas com colunas e linhas**, conectadas por **chaves**.

Exemplos: **PostgreSQL**, MySQL, SQL Server, Oracle, SQLite.

```
Tabela: usuarios          Tabela: pedidos
┌────┬──────────┐         ┌────┬──────────┬────────────┐
│ id │ nome     │         │ id │ valor    │ usuario_id │
├────┼──────────┤         ├────┼──────────┼────────────┤
│ 1  │ Mauricio │ ←──┐    │ 1  │ R$ 100   │ 1          │
│ 2  │ Ana      │    └────│ 2  │ R$ 250   │ 1          │
└────┴──────────┘         │ 3  │ R$ 50    │ 2          │
                          └────┴──────────┴────────────┘
```

✅ Forte em consistência, relações complexas, transações
✅ Padronizado (SQL é universal)
❌ Schema rígido — mudar estrutura é trabalhoso

### 🌊 NoSQL

Termo guarda-chuva pra bancos não-relacionais. Várias famílias:

| Tipo | Exemplo | Uso típico |
|---|---|---|
| **Documento** | MongoDB | JSON-like, flexível |
| **Chave-valor** | Redis | Cache, sessões |
| **Coluna** | Cassandra | Big data, escrita massiva |
| **Grafo** | Neo4j | Redes sociais, relacionamentos |

> 🤔 **SQL ou NoSQL?** A regra prática: comece com **PostgreSQL**. Ele resolve 95% dos casos. Use NoSQL quando tiver problema específico (cache → Redis, dados não estruturados → MongoDB, etc).

---

## 📝 SQL Básico

### CRUD

```sql
-- CREATE: inserir dados
INSERT INTO usuarios (nome, email)
VALUES ('Mauricio', 'mau@example.com');

-- READ: buscar dados
SELECT * FROM usuarios;
SELECT nome, email FROM usuarios WHERE id = 1;

-- UPDATE: atualizar
UPDATE usuarios SET email = 'novo@example.com' WHERE id = 1;

-- DELETE: remover
DELETE FROM usuarios WHERE id = 1;
```

### JOIN: O Coração do SQL

Combina dados de múltiplas tabelas:

```sql
-- Pegar usuário com seus pedidos
SELECT u.nome, p.valor
FROM usuarios u
INNER JOIN pedidos p ON p.usuario_id = u.id;
```

| Tipo de JOIN | O que retorna |
|---|---|
| `INNER JOIN` | Só linhas que dão match nas duas tabelas |
| `LEFT JOIN` | Tudo da esquerda + matches da direita (NULL se não tiver) |
| `RIGHT JOIN` | O contrário do LEFT |
| `FULL OUTER JOIN` | Tudo dos dois lados |

### Outras cláusulas essenciais

```sql
SELECT
    estado,
    COUNT(*) as total,
    AVG(valor) as ticket_medio
FROM pedidos
WHERE valor > 50
GROUP BY estado
HAVING COUNT(*) > 10
ORDER BY total DESC
LIMIT 10;
```

> 💡 **Ordem mental de execução** (importante!):
> `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`

---

## 🔑 Modelagem: Chaves e Relações

### Primary Key (PK)

Identifica unicamente cada linha. Geralmente um `id` numérico ou UUID.

### Foreign Key (FK)

Aponta pra PK de outra tabela. **Garante integridade referencial** — você não consegue criar um pedido pra um usuário que não existe.

```sql
CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    usuario_id INT NOT NULL REFERENCES usuarios(id),
    valor DECIMAL(10,2)
);
```

### Cardinalidade

- **1:1** — Um usuário tem um perfil
- **1:N** — Um usuário tem vários pedidos
- **N:N** — Um pedido tem vários produtos (precisa tabela intermediária)

### Normalização (resumo prático)

Princípio: **cada fato deve viver em um lugar só**.

❌ **Mal normalizado**:
```
| pedido_id | usuario_nome | usuario_email | produto |
| 1         | Mauricio     | mau@ex.com    | Caneta  |
| 2         | Mauricio     | mau@ex.com    | Caderno |
```
Se o Mauricio mudar de email, tem que atualizar em vários lugares. Inconsistência iminente.

✅ **Normalizado**:
```
usuarios: | id | nome | email |
pedidos:  | id | usuario_id | produto |
```

> 🤔 **Quando desnormalizar?** Em sistemas que precisam de muita performance de leitura, às vezes vale duplicar dados. Trade-off clássico.

---

## ⚡ Índices: A Diferença entre 10ms e 10 segundos

### O problema sem índice

Imagine uma tabela com 10 milhões de usuários. `SELECT * FROM usuarios WHERE email = 'x'` precisa varrer **linha por linha** (table scan). Lento.

### Como o índice resolve

Um índice é uma **estrutura paralela** organizada em **B-tree** (ou similar) que permite busca em `O(log n)` em vez de `O(n)`.

> 💡 **Analogia**: o índice de um livro. Você não lê o livro inteiro pra achar o capítulo "Database" — vai no índice e encontra a página direto.

### Criando

```sql
-- Índice simples
CREATE INDEX idx_usuarios_email ON usuarios(email);

-- Índice composto (ordem importa!)
CREATE INDEX idx_pedidos_user_data ON pedidos(usuario_id, criado_em);

-- Índice único (impede duplicatas)
CREATE UNIQUE INDEX idx_usuarios_email_unico ON usuarios(email);
```

### Quando NÃO usar índice

- ⚠️ Cada índice **deixa escritas mais lentas** (precisa atualizar o índice junto)
- ⚠️ Ocupa espaço em disco
- ⚠️ Em tabelas pequenas (< 1000 linhas), table scan é mais rápido

> 🤔 **Regra prática**: indexe colunas usadas em `WHERE`, `JOIN`, `ORDER BY`. Não indexe tudo.

### EXPLAIN: Seu Melhor Amigo

Mostra o plano de execução da query:

```sql
EXPLAIN ANALYZE
SELECT * FROM usuarios WHERE email = 'mau@example.com';
```

Procure por:
- ✅ **Index Scan** ou **Index Only Scan** → bom
- ❌ **Seq Scan** em tabela grande → ruim

---

## 🔐 Transações: ACID

Uma **transação** é um conjunto de operações que devem acontecer **todas juntas ou nenhuma**.

```sql
BEGIN;

UPDATE conta SET saldo = saldo - 100 WHERE id = 1;
UPDATE conta SET saldo = saldo + 100 WHERE id = 2;

COMMIT;  -- Confirma tudo
-- ou
ROLLBACK;  -- Desfaz tudo
```

Se a luz cair entre os dois UPDATEs, sem transação, o dinheiro **some**. Com transação, ou os dois rolam ou nenhum.

### Propriedades ACID

| Letra | Significa | O que garante |
|---|---|---|
| **A** | Atomicidade | Tudo ou nada |
| **C** | Consistência | Bancos sempre fica em estado válido |
| **I** | Isolamento | Transações concorrentes não interferem |
| **D** | Durabilidade | Depois do commit, persiste mesmo se cair energia |

### Níveis de isolamento

Tradeoff entre **performance** e **garantias**:

| Nível | Permite |
|---|---|
| `READ UNCOMMITTED` | Dirty reads (lê dados não commitados) |
| `READ COMMITTED` | Non-repeatable reads (padrão do PostgreSQL) |
| `REPEATABLE READ` | Phantom reads |
| `SERIALIZABLE` | Nada — máxima garantia, menor performance |

---

## 🚨 N+1: O Problema Mais Comum em ORMs

Você tem 100 usuários, cada um com pedidos. Em código:

```java
// Java + JPA
List<Usuario> usuarios = usuarioRepo.findAll();         // 1 query
for (Usuario u : usuarios) {
    System.out.println(u.getPedidos().size());          // 100 queries!
}
```

São **101 queries** quando 1 só resolveria. Isso é o **N+1 problem** — afeta **qualquer ORM**, não só JPA.

### O mesmo problema em outras stacks

```typescript
// Node.js + Prisma (sem include)
const usuarios = await prisma.usuario.findMany();
for (const u of usuarios) {
    const pedidos = await prisma.pedido.findMany({ where: { usuarioId: u.id } });
}
```

```python
# Python + SQLAlchemy (sem joinedload)
usuarios = session.query(Usuario).all()
for u in usuarios:
    print(len(u.pedidos))  # query nova a cada iteração
```

### Soluções

**Java + JPA:**
```java
@Query("SELECT u FROM Usuario u JOIN FETCH u.pedidos")
List<Usuario> findAllComPedidos();
```

**Prisma:**
```typescript
prisma.usuario.findMany({ include: { pedidos: true } })
```

**SQLAlchemy:**
```python
session.query(Usuario).options(joinedload(Usuario.pedidos)).all()
```

**Entity Framework:**
```csharp
context.Usuarios.Include(u => u.Pedidos).ToList();
```

> 🤔 **Dica universal**: ative log de queries SQL em desenvolvimento (`spring.jpa.show-sql`, `prisma log`, `echo=True` no SQLAlchemy). Veja quantas queries cada endpoint dispara — vai te assustar.

---

## 🏊 Connection Pool

Abrir uma conexão com o banco custa caro (handshake TCP, autenticação). Em vez de abrir/fechar a cada query, você mantém um **pool de conexões reutilizáveis**.

### Pools populares por stack

- **Java**: HikariCP (padrão no Spring Boot)
- **Node.js**: `pg-pool`, integrado em Prisma/TypeORM
- **Python**: pool integrado no SQLAlchemy
- **.NET**: pool nativo do ADO.NET / Entity Framework
- **Go**: `database/sql` tem pool built-in

### Configuração (exemplo Spring Boot)

```properties
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.idle-timeout=300000
```

> ⚠️ **Pool muito pequeno**: requisições ficam esperando conexão livre. **Pool muito grande**: banco fica sobrecarregado. Comece com 10 e ajuste conforme métricas.

---

## ✅ Resumo da página

- **PostgreSQL é a melhor escolha padrão**; NoSQL pra casos específicos
- SQL é declarativo: você fala **o que** quer, não **como** buscar
- **JOINs** combinam tabelas; **chaves estrangeiras** garantem integridade
- **Normalize** pra consistência, **desnormalize** pra performance quando preciso
- **Índices** são essenciais — mas não indexe tudo (afeta escrita)
- Use **EXPLAIN ANALYZE** pra diagnosticar queries lentas
- Transações ACID garantem consistência em operações complexas
- Cuidado com **N+1** em qualquer ORM (JPA, Prisma, SQLAlchemy, EF...)
- **Connection pool** (HikariCP, pg-pool, etc) é obrigatório em produção

---

⬅️ [Anterior: Redes](./07-redes.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Concorrência](./09-concorrencia.md)
