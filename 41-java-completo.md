# ☕ Java: Da Linguagem ao Ecossistema

> Java é uma das linguagens mais usadas do mundo há **30 anos**. Maturidade enorme, ecossistema massivo, performance excelente. Aqui um mergulho prático: sintaxe específica, JVM, ferramentas de build, e features modernas que muita gente não usa porque "aprendeu Java em 2010".

---

## 🌍 Sobre Java

### Princípios fundamentais

- 🏛️ **Orientado a objetos** (paradigma principal)
- 📦 **Compilado pra bytecode**, executado pela JVM
- 🌐 **"Write once, run anywhere"** — bytecode roda em qualquer SO com JVM
- 🛡️ **Tipagem estática forte**
- 🗑️ **Garbage collection** automático
- 🧵 **Multithreading** nativo

### Por que Java é tão usado em backend?

- ✅ **Maturidade**: 30 anos de bibliotecas e ferramentas
- ✅ **Performance**: JIT da JVM produz código quase tão rápido quanto C
- ✅ **Tooling fenomenal**: IntelliJ, Eclipse, JProfiler
- ✅ **Ecossistema enterprise**: Spring, Jakarta EE
- ✅ **Comunidade gigante**: Stack Overflow tem resposta pra tudo
- ✅ **Concorrência madura**: Virtual Threads (Java 21+) revolucionaram

---

## 🔧 JDK, JRE, JVM: Entendendo a Sopa de Letrinhas

### JVM (Java Virtual Machine)

A "máquina virtual" que executa bytecode. **Existem várias implementações:**
- ☕ **HotSpot** (Oracle/OpenJDK — padrão)
- 🚀 **GraalVM** (compilação ahead-of-time, perf alta)
- 🌐 **Eclipse OpenJ9** (uso de memória menor)

### JRE (Java Runtime Environment)

JVM + bibliotecas padrão. **Mínimo pra rodar** apps Java.

### JDK (Java Development Kit)

JRE + ferramentas de desenvolvimento (`javac`, `jar`, `javadoc`, etc). **Necessário pra desenvolver**.

```
JDK
├── JRE
│   ├── JVM
│   └── Bibliotecas padrão
└── Ferramentas (javac, jar, javadoc)
```

### Distribuições de JDK

Várias empresas distribuem o JDK (todos baseados no OpenJDK):
- 🏢 **Eclipse Temurin** (mais usado, gratuito)
- 🏢 **Amazon Corretto** (gratuito, AWS oferece suporte)
- 🏢 **Oracle JDK** (gratuito pra dev/test, pago pra produção)
- 🏢 **GraalVM** (Oracle, performance)
- 🏢 **Microsoft Build of OpenJDK**

> 💡 **Recomendação**: comece com **Temurin**. Use [SDKMAN!](https://sdkman.io) pra gerenciar versões.

### Versões importantes

| Versão | Lançamento | Status | Highlights |
|---|---|---|---|
| **8** | 2014 | LTS | Lambdas, Streams, Optional |
| **11** | 2018 | LTS | `var`, HTTP Client, modularidade |
| **17** | 2021 | LTS | Records, sealed classes, pattern matching |
| **21** | 2023 | LTS | Virtual Threads, sequenced collections |
| **25** | 2025 | LTS | (futuro) |

> 💡 **LTS** (Long Term Support) = versões com suporte estendido. Use sempre LTS em produção. **Java 21 é o padrão moderno** em 2026.

---

## 🏗️ Estrutura de um Projeto Java

```
meu-projeto/
├── src/
│   ├── main/
│   │   ├── java/                     ← código fonte
│   │   │   └── com/exemplo/app/
│   │   │       ├── Application.java
│   │   │       └── Service.java
│   │   └── resources/                ← arquivos não-Java
│   │       └── application.properties
│   └── test/
│       └── java/                     ← testes
├── pom.xml ou build.gradle           ← build config
└── target/ ou build/                 ← saída do build
```

### Anatomia de um arquivo Java

```java
package com.exemplo.app;       // pacote (organização)

import java.util.List;          // imports
import java.util.ArrayList;

public class MinhaClasse {     // classe pública (1 por arquivo)

    // atributos
    private String nome;
    private int idade;

    // construtor
    public MinhaClasse(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }

    // métodos
    public void apresentar() {
        System.out.println("Olá, sou " + nome);
    }

    // método especial: ponto de entrada do programa
    public static void main(String[] args) {
        MinhaClasse pessoa = new MinhaClasse("Mauricio", 30);
        pessoa.apresentar();
    }
}
```

> ⚠️ **Nome do arquivo deve bater com a classe pública**: `MinhaClasse.java` contém `public class MinhaClasse`.

---

## 🧱 Tipos de Dados

### Primitivos (8 tipos)

| Tipo | Tamanho | Exemplo | Padrão |
|---|---|---|---|
| `byte` | 8 bits | `byte b = 100;` | 0 |
| `short` | 16 bits | `short s = 1000;` | 0 |
| `int` | 32 bits | `int i = 100000;` | 0 |
| `long` | 64 bits | `long l = 100000L;` | 0L |
| `float` | 32 bits | `float f = 3.14f;` | 0.0f |
| `double` | 64 bits | `double d = 3.14;` | 0.0 |
| `char` | 16 bits | `char c = 'A';` | '\u0000' |
| `boolean` | 1 bit | `boolean b = true;` | false |

### Wrapper Classes

Cada primitivo tem uma classe correspondente: `Integer`, `Double`, `Boolean`, etc.

```java
int x = 10;              // primitivo
Integer y = 10;          // objeto (wrapper)

// Autoboxing (conversão automática)
Integer z = x;           // primitivo → wrapper
int w = y;               // wrapper → primitivo
```

> 💡 **Use primitivos sempre que possível** (mais rápidos, menos memória). Wrappers só quando precisar de objeto (ex: em coleções como `List<Integer>`).

### String (não é primitivo!)

```java
String nome = "Mauricio";

// Imutável! Modificar gera nova string
String maiusculo = nome.toUpperCase();  // "MAURICIO"
// `nome` continua sendo "Mauricio"

// Concatenar muitas strings? Use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("texto");
}
String resultado = sb.toString();
```

### Inferência de tipo (`var`) — Java 10+

```java
var nome = "Mauricio";              // String
var idade = 30;                     // int
var lista = new ArrayList<String>(); // ArrayList<String>

// Mas só em variáveis locais
var x;     // ❌ erro, precisa inicializar
```

---

## 📦 Coleções (Collections Framework)

### Hierarquia simplificada

```
Collection
├── List       (ordem importa, permite duplicatas)
│   ├── ArrayList
│   └── LinkedList
├── Set        (sem duplicatas)
│   ├── HashSet
│   ├── TreeSet (ordenado)
│   └── LinkedHashSet (mantém ordem de inserção)
└── Queue/Deque (filas)
    ├── ArrayDeque
    └── PriorityQueue

Map (não estende Collection)
├── HashMap
├── TreeMap (ordenado)
└── LinkedHashMap
```

### Uso prático

```java
// List
List<String> nomes = new ArrayList<>();
nomes.add("Ana");
nomes.get(0);
nomes.size();

// Modo imutável (Java 9+)
List<String> imutavel = List.of("a", "b", "c");
// imutavel.add("d");  // ❌ UnsupportedOperationException

// Map
Map<String, Integer> idades = new HashMap<>();
idades.put("Ana", 30);
idades.get("Ana");

Map<String, Integer> imutavelMap = Map.of("Ana", 30, "Bruno", 25);

// Iterar
for (Map.Entry<String, Integer> entry : idades.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// Java moderno
idades.forEach((nome, idade) -> System.out.println(nome + ": " + idade));
```

---

## 🎯 Exceções: try/catch/finally

```java
try {
    arquivo = new FileReader("nao-existe.txt");
} catch (FileNotFoundException e) {
    System.err.println("Arquivo não encontrado: " + e.getMessage());
} catch (IOException e) {
    System.err.println("Erro de I/O: " + e.getMessage());
} finally {
    // sempre executa, mesmo com exceção
    if (arquivo != null) arquivo.close();
}
```

### Try-with-resources (Java 7+)

Fecha recursos automaticamente:

```java
try (FileReader arquivo = new FileReader("arquivo.txt");
     BufferedReader br = new BufferedReader(arquivo)) {
    String linha = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
// arquivo e br são fechados automaticamente
```

### Checked vs Unchecked

```java
// Checked: compilador OBRIGA a tratar
public void lerArquivo() throws IOException {  // declarando que pode lançar
    new FileReader("teste.txt");
}

// Unchecked (RuntimeException): você TEM A OPÇÃO de tratar
public void dividir(int a, int b) {
    int resultado = a / b;  // pode lançar ArithmeticException, mas não obriga catch
}
```

### Exceções customizadas

```java
public class SaldoInsuficienteException extends RuntimeException {
    public SaldoInsuficienteException(String msg) {
        super(msg);
    }
}

public class Conta {
    public void sacar(double valor) {
        if (valor > saldo) {
            throw new SaldoInsuficienteException("Saldo: " + saldo);
        }
        saldo -= valor;
    }
}
```

---

## ⚡ Java Funcional: Lambdas e Streams

Java 8 (2014) revolucionou a linguagem. Quem não usa, perde produtividade enorme.

### Lambdas

Funções anônimas concisas:

```java
// Antes (Java 7) — verboso
Comparator<String> comparator = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
};

// Depois (Java 8+) — lambda
Comparator<String> comparator = (a, b) -> a.length() - b.length();
```

### Method references

```java
// Lambda
list.forEach(s -> System.out.println(s));

// Method reference (mais limpo)
list.forEach(System.out::println);
```

### Streams API

Operações funcionais em coleções:

```java
List<Integer> numeros = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

int somaQuadradosPares = numeros.stream()
    .filter(n -> n % 2 == 0)        // só pares: [2, 4, 6, 8, 10]
    .mapToInt(n -> n * n)            // ao quadrado: [4, 16, 36, 64, 100]
    .sum();                          // soma: 220

// Outras operações
List<String> nomesMaiusculos = pessoas.stream()
    .filter(p -> p.getIdade() >= 18)
    .map(Pessoa::getNome)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());

// Agrupar
Map<String, List<Pessoa>> porCidade = pessoas.stream()
    .collect(Collectors.groupingBy(Pessoa::getCidade));

// Reduzir
double media = numeros.stream()
    .mapToInt(Integer::intValue)
    .average()
    .orElse(0);
```

> 💡 **Streams são lazy**: nada é executado até a operação terminal (`collect`, `sum`, `forEach`, etc).

### Optional

Pra lidar com **valores que podem não existir** sem `null`:

```java
public Optional<Usuario> buscarPorId(Long id) {
    return repo.findById(id);
}

// Uso
Optional<Usuario> opt = service.buscarPorId(123L);

opt.ifPresent(u -> System.out.println(u.getNome()));

// Com default
String nome = opt.map(Usuario::getNome).orElse("Desconhecido");

// Lançar exceção
Usuario user = opt.orElseThrow(() -> new NotFoundException("Usuário não existe"));
```

> ⚠️ **Não use Optional como atributo de classe** ou parâmetro. Só como **retorno** de método.

---

## 🆕 Java Moderno (17+)

### Records (Java 16+)

DTOs e value objects em uma linha:

```java
public record Endereco(String rua, String cidade, String cep) {}

// Equivalente a uma classe com:
// - construtor
// - getters: rua(), cidade(), cep()
// - equals, hashCode, toString
// - imutabilidade

Endereco e = new Endereco("Rua X", "São Paulo", "01234-567");
System.out.println(e.cidade());  // "São Paulo"
```

### Sealed Classes (Java 17+)

Limita quem pode estender:

```java
public sealed interface FormaPagamento
    permits Cartao, Pix, Boleto {
}

public final class Cartao implements FormaPagamento { /* ... */ }
public final class Pix implements FormaPagamento { /* ... */ }
public final class Boleto implements FormaPagamento { /* ... */ }
// Ninguém mais pode implementar FormaPagamento
```

> 💡 **Útil pra**: domínios fechados, exhaustive pattern matching.

### Pattern Matching (Java 21+)

```java
Object obj = "Olá";

// Pattern matching com instanceof
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());  // s já é String
}

// Switch pattern matching (Java 21+)
String resultado = switch (forma) {
    case Cartao c -> "Cartão " + c.numero();
    case Pix p -> "Pix " + p.chave();
    case Boleto b -> "Boleto " + b.codigoBarras();
};
```

### Virtual Threads (Java 21+)

Threads ultraleves — milhares delas sem sofrimento:

```java
// Antes: threads pesadas, pool limitado
ExecutorService executor = Executors.newFixedThreadPool(100);

// Java 21+: virtual threads, milhares
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        executor.submit(() -> {
            // tarefa I/O bound — virtual thread brilha aqui
            httpClient.get(url);
        });
    }
}
```

> 💡 **Game-changer pra apps web**. Spring Boot 3.2+ suporta nativamente.

---

## 🛠️ Build Tools: Maven vs Gradle

### Maven

XML, convenção sobre configuração. **Mais usado** historicamente.

```xml
<!-- pom.xml -->
<project>
    <groupId>com.exemplo</groupId>
    <artifactId>meu-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.0</version>
        </dependency>
    </dependencies>
</project>
```

```bash
mvn clean install      # baixa deps, compila, testa, empacota
mvn spring-boot:run    # roda app Spring
mvn test               # só testes
```

### Gradle

Groovy/Kotlin DSL, mais flexível, **builds mais rápidos**.

```kotlin
// build.gradle.kts
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
}

tasks.test {
    useJUnitPlatform()
}
```

```bash
./gradlew build
./gradlew bootRun
./gradlew test
```

> 💡 **Maven**: ainda padrão em projetos enterprise/legacy.
> **Gradle**: dominante em Android, ganhando espaço em backend novo.

---

## 🎯 Frameworks Essenciais

| Framework | Pra quê | Stack |
|---|---|---|
| **Spring Boot** | Web, microserviços, DI | Mais popular |
| **Quarkus** | Cloud native, AOT | Performance/serverless |
| **Micronaut** | Microserviços leves | Alternativa ao Spring |
| **Jakarta EE** | Padrão enterprise | Legado e enterprise |
| **JUnit 5** | Testes | Padrão de fato |
| **Mockito** | Mocks em testes | Junto com JUnit |
| **Hibernate / JPA** | ORM | Padrão pra DB relacional |
| **Lombok** | Reduz boilerplate | Polêmico mas popular |

---

## 📚 Boas Práticas Java Modernas

### 1. Use imutabilidade quando possível

```java
// ✅ Records são imutáveis por padrão
public record Pedido(Long id, BigDecimal valor, Status status) {}
```

### 2. Prefira `var` em locais

```java
// Verboso
ArrayList<Map<String, List<Pedido>>> dados = new ArrayList<>();

// Limpo
var dados = new ArrayList<Map<String, List<Pedido>>>();
```

### 3. Use Optional em retornos, não em atributos

```java
// ✅
public Optional<Usuario> buscar(Long id) { /* ... */ }

// ❌
private Optional<String> nome;  // Optional como atributo é code smell
```

### 4. Streams pra transformações

```java
// ✅
List<String> ativos = pessoas.stream()
    .filter(Pessoa::isAtivo)
    .map(Pessoa::getNome)
    .toList();
```

### 5. Try-with-resources sempre

```java
try (var conn = DriverManager.getConnection(url)) {
    // usa conn
}  // fecha automaticamente
```

### 6. Records pra DTOs

```java
public record UserResponse(Long id, String nome, String email) {}
```

### 7. `@Override` sempre que sobrescrever

```java
@Override
public String toString() { /* ... */ }
```

### 8. Não use `null` como retorno — use Optional

### 9. Não capture `Exception` genérica — seja específico

```java
// ❌
try { /* ... */ } catch (Exception e) { /* engole tudo */ }

// ✅
try { /* ... */ } catch (IOException e) { /* trata I/O */ }
  catch (SQLException e) { /* trata SQL */ }
```

### 10. Imports organizados (a IDE faz)

```java
import java.util.*;        // ❌ wildcard imports
import java.util.List;     // ✅ explícito
import java.util.Map;
```

---

## 🎯 Onde Java se Encaixa em 2026

### Pontos fortes

- 🏢 **Backend enterprise**: bancos, telecom, fintechs
- 🌊 **Sistemas de alta carga**: Netflix, Uber, LinkedIn
- 🤖 **Big data**: Spark, Kafka, Hadoop são em JVM
- 📱 **Android** (Kotlin é JVM também)
- ☕ **APIs corporativas**: SOAP/REST com Spring

### Pontos fracos

- 🚫 **Scripts pequenos** (Python ganha)
- 🚫 **Frontend** (não foi feito pra isso)
- 🚫 **Startup time** (cold start em serverless é problema — GraalVM ajuda)

---

## ✅ Resumo da página

- **Java**: linguagem OO compilada pra bytecode, executado na JVM
- **JDK** > **JRE** > **JVM** (cada um inclui o anterior)
- **Java 21 LTS** é o moderno em 2026 — use sempre versão LTS em produção
- Tipos primitivos (8) + classes wrapper + String + arrays + coleções
- **Try-with-resources** > finally manual
- **Lambdas + Streams** transformam código verboso em conciso
- **Optional** evita `null` em retornos
- **Records** simplificam DTOs e value objects
- **Sealed classes** limitam herança (combina com pattern matching)
- **Virtual Threads** (Java 21+) revolucionam concorrência I/O
- **Maven** ainda dominante, **Gradle** crescendo
- **Spring Boot** é o framework de fato pra web/backend Java
- Use Java moderno: `var`, lambdas, streams, records, switch expressions

---

⬅️ [Anterior: POO - Os 4 Pilares](./40-poo-pilares.md) | 🔙 [Voltar ao índice](./README.md)
