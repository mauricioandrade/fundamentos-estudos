# 🎨 Design Patterns

> Soluções **comprovadas** pra problemas que aparecem repetidamente no design de software. Nascem do livro "Design Patterns" (1994) da Gang of Four. Você já usa vários sem saber.

---

## 🤔 O que é Design Pattern (e o que NÃO é)

### O que é

- 📋 **Receita** pra resolver um problema recorrente
- 🗣️ **Vocabulário comum** entre devs ("é um Observer aqui")
- 🧱 Combinação de classes/interfaces que se relacionam de um jeito específico

### O que NÃO é

- ❌ Código pra copiar e colar
- ❌ Solução pra todo problema (over-engineering é ruim)
- ❌ Algoritmo

> ⚠️ **Cuidado**: forçar um pattern onde não precisa cria complexidade desnecessária. Use quando o problema **realmente aparece**.

---

## 📦 Categorias

A GoF separou em 3 grupos:

| Categoria | Foco |
|---|---|
| **Criacionais** | Como **criar** objetos |
| **Estruturais** | Como **compor** objetos |
| **Comportamentais** | Como objetos **se comunicam** |

---

## 🏗️ Patterns Criacionais

### 1️⃣ Singleton

**Problema**: garantir que uma classe tem apenas **uma instância** em todo o sistema.

```java
public class Configuracao {
    private static Configuracao instancia;

    private Configuracao() {}  // construtor privado

    public static Configuracao getInstance() {
        if (instancia == null) {
            instancia = new Configuracao();
        }
        return instancia;
    }
}
```

**Quando usar**: pools de conexão, cache global, configurações.

> ⚠️ **Cuidado**: singleton vira "variável global disfarçada". Difícil de testar. **Em frameworks modernos com DI** (Spring, NestJS, ASP.NET, FastAPI), você raramente precisa implementar manualmente — o framework gerencia o ciclo de vida pra você.

### 2️⃣ Factory Method

**Problema**: criar objetos sem expor a lógica de criação.

```java
public abstract class NotificacaoFactory {
    public abstract Notificacao criar();
}

public class EmailFactory extends NotificacaoFactory {
    @Override
    public Notificacao criar() {
        return new EmailNotificacao();
    }
}

public class SmsFactory extends NotificacaoFactory {
    @Override
    public Notificacao criar() {
        return new SmsNotificacao();
    }
}
```

**Uso real**: `Calendar.getInstance()`, `LoggerFactory.getLogger()`, drivers JDBC.

### 3️⃣ Builder

**Problema**: criar objetos com **muitos parâmetros opcionais** sem ter 20 construtores.

```java
Pizza pizza = new Pizza.Builder()
    .tamanho("grande")
    .massa("fina")
    .adicionarIngrediente("mussarela")
    .adicionarIngrediente("calabresa")
    .borda("recheada")
    .build();
```

**Uso real**: `StringBuilder`, `Stream.builder()`, **Lombok `@Builder`** te dá isso de graça.

---

## 🏛️ Patterns Estruturais

### 4️⃣ Adapter

**Problema**: fazer duas interfaces incompatíveis trabalharem juntas.

```java
// Interface que seu código espera
interface PagamentoBR {
    void pagar(double valorReais);
}

// API externa em dólar
class PaypalAPI {
    public void payment(double dollars) { ... }
}

// Adapter
class PaypalAdapter implements PagamentoBR {
    private PaypalAPI paypal;
    private double cotacao = 5.0;

    @Override
    public void pagar(double valorReais) {
        paypal.payment(valorReais / cotacao);
    }
}
```

> 💡 **Analogia**: adaptador de tomada — você tem o aparelho e a tomada incompatíveis, o adaptador faz funcionar.

### 5️⃣ Decorator

**Problema**: adicionar funcionalidades a objetos **sem alterar** sua classe.

```java
Cafe cafe = new CafeSimples();
cafe = new ComLeite(cafe);
cafe = new ComChocolate(cafe);
cafe = new ComChantilly(cafe);

cafe.preco();  // soma de todos os adicionais
```

**Uso real**: streams Java (`InputStream` → `BufferedInputStream` → `DataInputStream`), middlewares HTTP.

### 6️⃣ Facade

**Problema**: simplificar a interação com um subsistema complexo.

```java
// Em vez do cliente lidar com 5 classes:
public class PedidoFacade {
    public void finalizarPedido(Carrinho c) {
        estoqueService.reservar(c);
        pagamentoService.cobrar(c);
        emailService.enviarConfirmacao(c);
        analyticsService.registrar(c);
    }
}
```

**Uso real**: APIs do Spring (`JdbcTemplate` esconde JDBC bruto), SDKs em geral.

### 7️⃣ Proxy

**Problema**: controlar acesso a um objeto (cache, lazy loading, segurança).

**Uso real**: **`@Transactional` no Spring** — o Spring cria um proxy que abre/fecha transação ao redor da sua função. JPA usa proxies pra **lazy loading** de relacionamentos.

> 🤔 **Por que `@Transactional` não funciona em métodos privados?** Porque o proxy intercepta apenas chamadas externas. Métodos internos não passam pelo proxy.

---

## 🔄 Patterns Comportamentais

### 8️⃣ Strategy

**Problema**: encapsular algoritmos intercambiáveis.

```java
interface DescontoStrategy {
    double calcular(double valor);
}

class DescontoNatal implements DescontoStrategy {
    public double calcular(double valor) { return valor * 0.85; }
}

class DescontoFidelidade implements DescontoStrategy {
    public double calcular(double valor) { return valor * 0.90; }
}

class Pedido {
    private DescontoStrategy desconto;

    public Pedido(DescontoStrategy d) { this.desconto = d; }

    public double total(double valor) {
        return desconto.calcular(valor);
    }
}
```

> 💡 **Sinal claro de que precisa de Strategy**: você tem uma cadeia de `if/else` ou `switch` decidindo qual algoritmo aplicar.

### 9️⃣ Observer

**Problema**: notificar múltiplos objetos sobre mudanças de estado.

```java
interface Observer {
    void update(String evento);
}

class Publisher {
    private List<Observer> observers = new ArrayList<>();

    public void subscribe(Observer o) { observers.add(o); }

    public void publish(String evento) {
        observers.forEach(o -> o.update(evento));
    }
}
```

**Uso real**: event listeners (DOM, Spring `ApplicationEvent`), padrão pub/sub, RxJava.

### 🔟 Template Method

**Problema**: definir o esqueleto de um algoritmo, deixando subclasses customizarem partes.

```java
abstract class ProcessadorRelatorio {
    // Template — define a ordem
    public final void processar() {
        carregar();
        validar();
        formatar();
        salvar();
    }

    protected abstract void carregar();
    protected abstract void formatar();

    // Métodos com implementação padrão
    protected void validar() { /* default */ }
    protected void salvar() { /* default */ }
}
```

**Uso real**: `HttpServlet` (você sobrescreve `doGet`/`doPost`), JUnit (`@Before`, `@After`).

### 1️⃣1️⃣ Command

**Problema**: encapsular uma requisição como objeto.

```java
interface Command {
    void execute();
}

class CriarUsuarioCommand implements Command {
    private final String nome;
    private final UsuarioService service;

    public CriarUsuarioCommand(String nome, UsuarioService s) {
        this.nome = nome;
        this.service = s;
    }

    public void execute() {
        service.criar(nome);
    }
}
```

**Usos**: undo/redo, fila de tarefas, transações.

### 1️⃣2️⃣ Iterator

**Problema**: percorrer uma coleção sem expor sua estrutura interna.

Em Java, `for-each` usa Iterator nos bastidores:
```java
for (String s : lista) { ... }
// equivalente a:
Iterator<String> it = lista.iterator();
while (it.hasNext()) { ... }
```

---

## 🚀 Patterns Modernos

Não estavam no livro original, mas são essenciais hoje.

### Repository

Abstrai acesso a dados.

```java
// Java + Spring Data JPA
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    Optional<Usuario> findByEmail(String email);
}
```

**Equivalentes em outras stacks:**

```typescript
// Node.js + Prisma
prisma.usuario.findUnique({ where: { email } })
```

```python
# Python + SQLAlchemy
session.query(Usuario).filter_by(email=email).first()
```

```csharp
// C# + Entity Framework
context.Usuarios.FirstOrDefault(u => u.Email == email);
```

> 💡 **A ideia central é a mesma**: separar lógica de negócio do código que fala com o banco. Quase todo ORM/ODM moderno implementa esse pattern.

### Dependency Injection (DI)

Em vez da classe **criar** suas dependências, ela **recebe** de fora.

```java
// Sem DI (acoplado, difícil testar)
class PedidoService {
    private EmailService email = new EmailService();  // hardcoded
}

// Com DI (flexível, testável)
class PedidoService {
    private final EmailService email;

    public PedidoService(EmailService email) {
        this.email = email;  // Spring injeta
    }
}
```

> 💡 **No Spring**, prefira **constructor injection** (como acima) em vez de `@Autowired` em campos. É a boa prática moderna.

### Strategy + DI = Polimorfismo Útil

Spring detecta múltiplas implementações de uma interface e injeta a certa:

```java
@Service("desconto-natal")
class DescontoNatal implements DescontoStrategy { ... }

@Service("desconto-fidelidade")
class DescontoFidelidade implements DescontoStrategy { ... }

@Service
class Carrinho {
    private final Map<String, DescontoStrategy> estrategias;

    public Carrinho(Map<String, DescontoStrategy> estrategias) {
        this.estrategias = estrategias;  // Spring monta o map
    }
}
```

---

## 🚨 Anti-Patterns (Evite!)

| Anti-Pattern | O que é |
|---|---|
| **God Object** | Classe com 2000 linhas que faz tudo |
| **Spaghetti Code** | Fluxo descontrolado, dependências cruzadas |
| **Magic Numbers/Strings** | `if (status == 7)` em vez de constantes |
| **Premature Optimization** | Otimizar antes de medir o gargalo real |
| **Singleton Abuse** | Singleton pra tudo, virando estado global |
| **Copy-Paste Programming** | Duplicação em vez de abstração |

---

## ✅ Resumo da página

- Patterns são **soluções comprovadas** com vocabulário comum
- **Não force** patterns onde não precisa — over-engineering é ruim
- **Singleton, Factory, Builder** criam objetos
- **Adapter, Decorator, Facade, Proxy** organizam estrutura
- **Strategy, Observer, Template Method, Command** organizam comportamento
- **DI**, **Repository** são patterns modernos essenciais (e o Spring usa muito)
- Spring Boot já implementa muitos: `@Service` (singleton), `@Transactional` (proxy), Spring Data (repository)

---

⬅️ [Anterior: Estruturas de Dados](./13-estruturas-dados.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Arquitetura de Software](./15-arquitetura-software.md)
