# 🧮 Lógica de Programação e Paradigmas

> Antes de qualquer linguagem, framework ou pattern, existe a **lógica de programação** — a forma como você decompõe problemas em passos. E acima dela, os **paradigmas** — formas diferentes de organizar essa lógica. Entender isso te torna fluente em qualquer linguagem, não só Java.

---

## 🎯 O Que é Lógica de Programação

Programação é, no fundo, **dar instruções precisas a um computador** pra resolver um problema. Lógica de programação é **como você pensa** essas instruções.

> 💡 **Em palavras simples**: programar é como escrever uma receita. A lógica é como organizar os passos pra que faça sentido — você não bota o bolo no forno antes de misturar os ingredientes.

### Os blocos fundamentais

Toda linguagem (Java, Python, JS, etc) tem os mesmos elementos básicos:

1. 📦 **Variáveis** — guardar valores
2. ➕ **Operadores** — fazer contas e comparações
3. 🔀 **Condicionais** — tomar decisões (`if`/`else`)
4. 🔁 **Loops** — repetir ações (`for`, `while`)
5. 📞 **Funções/Métodos** — agrupar lógica reutilizável
6. 🗂️ **Estruturas de dados** — organizar coleções (listas, mapas)

Domine isso e você sabe **80% de qualquer linguagem**.

---

## 📦 Variáveis

Caixinhas nomeadas pra guardar dados.

```java
int idade = 30;                    // número inteiro
double salario = 4500.50;          // número decimal
String nome = "Mauricio";          // texto
boolean ativo = true;              // verdadeiro/falso
```

### Tipagem: estática vs dinâmica

```java
// Java — tipagem estática (tipo declarado, fixo)
int x = 10;
x = "texto";  // ❌ erro de compilação
```

```python
# Python — tipagem dinâmica (tipo deduzido, mutável)
x = 10
x = "texto"   # ✅ funciona, x agora é string
```

### Mutabilidade

```java
int x = 10;
x = 20;  // ✅ pode mudar

final int Y = 10;
Y = 20;  // ❌ erro — final é imutável
```

> 💡 **Boa prática moderna**: prefira variáveis **imutáveis** quando possível. Reduz bugs sutis. Em Java moderno: use `final` ou `record`.

---

## ➕ Operadores

### Aritméticos

```java
int soma = 5 + 3;        // 8
int sub  = 5 - 3;        // 2
int mult = 5 * 3;        // 15
int div  = 5 / 3;        // 1 (divisão inteira!)
double divD = 5.0 / 3;   // 1.6666...
int resto = 5 % 3;       // 2 (módulo)
```

### Comparação (retornam boolean)

```java
5 == 5    // true
5 != 3    // true
5 > 3     // true
5 <= 5    // true
```

### Lógicos

```java
true && false   // false (E)
true || false   // true  (OU)
!true           // false (negação)
```

### Curto-circuito

```java
// Se a primeira condição já decide, a segunda nem é avaliada
if (lista != null && lista.size() > 0) {
    // se lista é null, .size() nem é chamado (evita NullPointerException)
}
```

---

## 🔀 Estruturas de Decisão

### if / else if / else

```java
if (idade >= 18) {
    System.out.println("Adulto");
} else if (idade >= 12) {
    System.out.println("Adolescente");
} else {
    System.out.println("Criança");
}
```

### Operador ternário (atalho)

```java
String status = idade >= 18 ? "Adulto" : "Menor";
```

### switch (Java moderno)

```java
String dia = switch (numero) {
    case 1 -> "Segunda";
    case 2 -> "Terça";
    case 3 -> "Quarta";
    default -> "Outro";
};
```

> 💡 Java 14+ tem switch expression (com `->` e retorno). Mais conciso que o switch antigo.

---

## 🔁 Loops

### for clássico

```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}
```

3 partes: inicialização, condição, incremento.

### for-each

```java
List<String> nomes = List.of("Ana", "Bruno", "Carla");
for (String nome : nomes) {
    System.out.println(nome);
}
```

### while

```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
}
```

### do-while

Executa **pelo menos uma vez**:

```java
int senha;
do {
    senha = pedirSenha();
} while (senha != 1234);
```

### Controle de loops

```java
for (int i = 0; i < 100; i++) {
    if (i == 50) break;       // sai do loop
    if (i % 2 == 0) continue; // pula pra próxima iteração
    System.out.println(i);
}
```

---

## 📞 Funções (Métodos em Java)

Bloco de código reutilizável que recebe **inputs** e geralmente retorna um **output**.

```java
public int somar(int a, int b) {
    return a + b;
}

int resultado = somar(5, 3); // 8
```

### Anatomia

```java
public  int       somar    (int a, int b)
↓       ↓         ↓        ↓
acesso  retorno   nome     parâmetros
```

### Tipos de método

```java
// void = não retorna nada
public void imprimir(String texto) {
    System.out.println(texto);
}

// estático = pertence à classe, não à instância
public static int dobrar(int x) {
    return x * 2;
}

// múltiplos parâmetros
public String concatenar(String... partes) {  // varargs
    return String.join("-", partes);
}
```

### Recursão

Função que chama a si mesma.

```java
public int fatorial(int n) {
    if (n <= 1) return 1;       // caso base
    return n * fatorial(n - 1); // chamada recursiva
}

fatorial(5);  // 5 * 4 * 3 * 2 * 1 = 120
```

> ⚠️ **Toda recursão precisa de caso base**. Sem ele, stack overflow garantido.

---

## 🗂️ Estruturas de Dados Básicas

> 💡 Pra estruturas de dados aprofundadas, veja o [tópico 13](./13-estruturas-dados.md). Aqui o básico essencial:

### Array

Tamanho **fixo**, elementos do mesmo tipo:

```java
int[] numeros = {1, 2, 3, 4, 5};
numeros[0];          // 1
numeros.length;      // 5
```

### List (ArrayList)

Tamanho **dinâmico**:

```java
List<String> nomes = new ArrayList<>();
nomes.add("Ana");
nomes.add("Bruno");
nomes.get(0);        // "Ana"
nomes.size();        // 2
```

### Map (HashMap)

Chave-valor:

```java
Map<String, Integer> idades = new HashMap<>();
idades.put("Ana", 30);
idades.put("Bruno", 25);
idades.get("Ana");   // 30
```

### Set (HashSet)

Coleção sem duplicatas:

```java
Set<String> emails = new HashSet<>();
emails.add("a@x.com");
emails.add("a@x.com");  // não adiciona, já existe
emails.size();           // 1
```

---

## 🎯 Pensando em Algoritmos

Algoritmo = sequência finita de passos pra resolver um problema.

### Como abordar um problema

1. **Entenda** o problema (input, output, restrições)
2. **Quebre** em sub-problemas menores
3. **Resolva** cada sub-problema
4. **Combine** as soluções
5. **Teste** com casos extremos (vazio, negativo, gigante)

### Exemplo: encontrar maior número

```java
public int maior(int[] numeros) {
    if (numeros.length == 0) {
        throw new IllegalArgumentException("Array vazio");
    }

    int maior = numeros[0];
    for (int i = 1; i < numeros.length; i++) {
        if (numeros[i] > maior) {
            maior = numeros[i];
        }
    }
    return maior;
}
```

> 💡 **Pseudo-código antes**: muitos devs juniors caem na armadilha de codar logo. Escreva em português primeiro:
> ```
> Pega o primeiro como maior
> Pra cada número seguinte:
>   Se for maior que o atual, vira o novo maior
> Retorna o maior
> ```

---

## 🎨 Paradigmas de Programação

Paradigma é uma **forma de pensar e organizar** o código. Mais que sintaxe, é **filosofia**.

Linguagens podem suportar **um ou vários** paradigmas. Java é **multi-paradigma** (OO principal, mas também tem funcional desde Java 8).

---

### 1️⃣ Paradigma Procedural

Código organizado em **procedimentos/funções** que processam dados.

```java
public class CalculadoraSimples {
    public static void main(String[] args) {
        int a = 10;
        int b = 5;

        int soma = somar(a, b);
        int produto = multiplicar(a, b);

        System.out.println("Soma: " + soma);
        System.out.println("Produto: " + produto);
    }

    static int somar(int x, int y) { return x + y; }
    static int multiplicar(int x, int y) { return x * y; }
}
```

✅ Simples e direto
✅ Bom pra scripts e tarefas pequenas
❌ Difícil escalar (sem encapsulamento)
❌ Funções operam em dados "soltos"

> 💡 Linguagens **historicamente procedurais**: C, Pascal, BASIC. Mas você pode escrever procedural em qualquer linguagem.

---

### 2️⃣ Paradigma Orientado a Objetos (POO)

Código organizado em **objetos** que combinam **dados (atributos) + comportamento (métodos)**.

```java
public class Conta {
    private double saldo;
    private String titular;

    public Conta(String titular) {
        this.titular = titular;
        this.saldo = 0;
    }

    public void depositar(double valor) {
        saldo += valor;
    }

    public void sacar(double valor) {
        if (valor > saldo) throw new IllegalStateException("Saldo insuficiente");
        saldo -= valor;
    }

    public double getSaldo() {
        return saldo;
    }
}

// Uso
Conta minhaConta = new Conta("Mauricio");
minhaConta.depositar(1000);
minhaConta.sacar(300);
System.out.println(minhaConta.getSaldo()); // 700
```

✅ Modela mundo real bem (objetos têm estado e comportamento)
✅ Reutilização via herança
✅ Manutenibilidade em sistemas grandes

> 💡 **Java é principalmente OO.** Os 4 pilares (abstração, encapsulamento, herança, polimorfismo) merecem capítulo próprio — veja [POO: Os 4 Pilares](./40-poo-pilares.md).

---

### 3️⃣ Paradigma Funcional

Código baseado em **funções puras** (sem efeitos colaterais), **imutabilidade** e **composição**.

```java
// Java moderno (8+) suporta funcional via Streams e lambdas

List<Integer> numeros = List.of(1, 2, 3, 4, 5);

int somaQuadradosPares = numeros.stream()
    .filter(n -> n % 2 == 0)        // só pares
    .mapToInt(n -> n * n)            // eleva ao quadrado
    .sum();                          // soma tudo

// Resultado: 4 + 16 = 20
```

#### Conceitos centrais

**Função pura**: mesmo input = mesmo output, sem efeitos colaterais

```java
// ✅ Pura
int dobrar(int x) { return x * 2; }

// ❌ Impura (efeito colateral: imprime na tela)
int dobrarELog(int x) {
    System.out.println(x);  // efeito colateral
    return x * 2;
}
```

**Imutabilidade**: dados não mudam após criados

```java
List<Integer> original = List.of(1, 2, 3);
List<Integer> dobrados = original.stream()
    .map(n -> n * 2)
    .collect(toList());
// original continua [1, 2, 3], dobrados é [2, 4, 6]
```

**Funções de alta ordem**: funções que recebem/retornam outras funções

```java
Function<Integer, Integer> dobrar = x -> x * 2;
Function<Integer, Integer> incrementar = x -> x + 1;

// Compor funções
Function<Integer, Integer> dobrarEIncrementar = dobrar.andThen(incrementar);
dobrarEIncrementar.apply(5);  // 11 (5*2+1)
```

✅ Código mais previsível (sem efeitos colaterais escondidos)
✅ Paralelização mais segura (imutabilidade)
✅ Testabilidade alta
❌ Curva de aprendizado (mudança de mentalidade)
❌ Performance pode ser pior em alguns casos (alocação)

> 💡 **Linguagens puramente funcionais**: Haskell, Elixir, Clojure. Mas Java, Python, JS, etc. têm features funcionais que você pode (e deve) usar.

---

### 4️⃣ Outros paradigmas (menção)

- 🧬 **Declarativo**: você diz **o que** quer, não **como** (SQL é exemplo)
- ⚡ **Reativo**: programação baseada em fluxos de eventos (RxJava, Project Reactor)
- 🔁 **Lógico**: programação por regras e fatos (Prolog)

---

## 🎯 Comparação dos 3 Principais

| Aspecto | Procedural | OO | Funcional |
|---|---|---|---|
| **Foco** | Sequência de passos | Objetos com estado | Transformação de dados |
| **Estado** | Variáveis globais | Encapsulado em objetos | Imutável |
| **Reutilização** | Funções | Herança, composição | Composição de funções |
| **Bom pra** | Scripts simples | Sistemas complexos | Processamento de dados |
| **Linguagens típicas** | C, Pascal | Java, C#, Python (oo) | Haskell, Elixir, Clojure |

> 💡 **Não existe "melhor" paradigma.** Bons devs usam **o paradigma certo pra cada problema**. Em Java moderno, você naturalmente combina OO + funcional o tempo todo.

---

## 🧠 Misturando Paradigmas (Java Moderno)

Java desde a versão 8 abraçou o funcional sem abandonar OO. Resultado: você usa o que faz sentido.

```java
public class PedidoService {
    private final PedidoRepository repo; // OO: encapsulamento

    public PedidoService(PedidoRepository repo) {
        this.repo = repo;
    }

    public BigDecimal totalPedidosAprovados(Long usuarioId) {
        return repo.findByUsuario(usuarioId).stream()    // OO
            .filter(p -> p.isAprovado())                  // Funcional
            .map(Pedido::getValor)                        // Funcional
            .reduce(BigDecimal.ZERO, BigDecimal::add);    // Funcional
    }
}
```

---

## ✅ Resumo da página

- **Lógica de programação** = como você decompõe problemas em passos
- Blocos fundamentais: **variáveis, operadores, condicionais, loops, funções, estruturas de dados**
- Java tem **tipagem estática** — tipos verificados em tempo de compilação
- Toda lógica complexa começa em **pseudo-código** antes do código
- Recursão precisa de **caso base** sempre
- **Paradigmas**:
  - 🔢 **Procedural**: sequência de passos, funções soltas
  - 🏛️ **OO**: objetos com estado e comportamento (foco principal de Java)
  - 🔄 **Funcional**: funções puras, imutabilidade, composição
- **Java moderno é multi-paradigma** — combine OO + funcional naturalmente
- **Não existe "melhor" paradigma** — escolha pelo problema

---

⬅️ [Anterior: Soft Skills](./38-soft-skills.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: POO - Os 4 Pilares](./40-poo-pilares.md)
