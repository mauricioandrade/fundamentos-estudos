# 🏛️ POO: Os 4 Pilares

> Programação Orientada a Objetos é o **paradigma dominante** em Java. E a base de tudo são 4 conceitos que aparecem em **toda entrevista**, **toda code review**, **todo design de sistema**: Abstração, Encapsulamento, Herança e Polimorfismo. Quem domina, escreve código limpo. Quem não domina, escreve código que dá manutenção difícil.

---

## 🎯 O Que é POO

POO é organizar código em **objetos** que combinam:
- 📦 **Estado** (atributos/dados)
- ⚡ **Comportamento** (métodos)

> 💡 **Em palavras simples**: em vez de ter "dados aqui" e "funções acolá", você junta tudo em objetos que **representam coisas do mundo real** ou conceitos do seu domínio.

```java
// Procedural — dados e funções soltos
double saldo = 1000;
public static void depositar(double valor) { /* ... */ }
public static void sacar(double valor) { /* ... */ }

// OO — objeto com tudo junto
public class Conta {
    private double saldo;

    public void depositar(double valor) { /* ... */ }
    public void sacar(double valor) { /* ... */ }
}
```

---

## 1️⃣ Abstração

### O Que é

**Esconder complexidade**, expor só o essencial. Você representa coisas reais como modelos simplificados.

> 💡 **Analogia**: você dirige carro sem saber como funciona o motor por dentro. O volante, pedais e câmbio são a **abstração** — você não precisa entender a mecânica completa pra usar.

### Em código

```java
// ❌ Sem abstração — usuário precisa saber detalhes
class ContaSemAbstracao {
    public double saldo;
    public List<Transacao> transacoes;
    public boolean bloqueada;

    // Cliente precisa fazer tudo manualmente:
    // verificar bloqueio, validar saldo, criar transação,
    // atualizar saldo, salvar transação...
}

// ✅ Com abstração — interface limpa
class Conta {
    private double saldo;
    private List<Transacao> transacoes;
    private boolean bloqueada;

    public void sacar(double valor) {
        // toda complexidade fica escondida aqui
        verificarBloqueio();
        validarSaldo(valor);
        registrarTransacao(valor);
        saldo -= valor;
    }
}
```

### Como aplicar

#### 1. Classes abstratas

```java
public abstract class FormaPagamento {
    // método "abstrato" — sem implementação
    public abstract void processar(double valor);

    // método concreto compartilhado
    public void registrarLog() {
        System.out.println("Pagamento processado às " + LocalDateTime.now());
    }
}

public class CartaoCredito extends FormaPagamento {
    @Override
    public void processar(double valor) {
        // implementação específica de cartão
    }
}

public class Pix extends FormaPagamento {
    @Override
    public void processar(double valor) {
        // implementação específica de Pix
    }
}
```

> ⚠️ **Você não pode instanciar classe abstrata**: `new FormaPagamento()` ❌. Só as subclasses concretas.

#### 2. Interfaces

```java
public interface Notificavel {
    void enviarNotificacao(String mensagem);
}

public class EmailNotifier implements Notificavel {
    public void enviarNotificacao(String mensagem) { /* ... */ }
}

public class SmsNotifier implements Notificavel {
    public void enviarNotificacao(String mensagem) { /* ... */ }
}
```

### Quando usar abstrata vs interface?

| Use abstrata quando | Use interface quando |
|---|---|
| Classes têm comportamento **comum** | Define **contrato** que diferentes classes podem implementar |
| Você precisa **estado compartilhado** | Quer **múltipla "herança"** (uma classe pode implementar várias interfaces) |
| Hierarquia "is-a" forte | Define capacidade ("can-do") |

---

## 2️⃣ Encapsulamento

### O Que é

**Controlar acesso** aos atributos e métodos de um objeto. Esconder o "como funciona" por trás de uma API definida.

> 💡 **Analogia**: o controle remoto da TV. Você aperta botão e a TV liga. Você não tem acesso direto aos circuitos internos — bom, porque alguém poderia mexer e estragar tudo.

### Modificadores de acesso em Java

```java
public class Pessoa {
    public String nome;        // qualquer um pode mexer
    private int idade;         // só essa classe pode mexer
    protected String email;    // essa classe + subclasses + mesmo pacote
    String telefone;           // padrão (só mesmo pacote)
}
```

| Modificador | Mesma classe | Mesmo pacote | Subclasses | Qualquer lugar |
|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| (padrão) | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

### Por que é importante

```java
// ❌ Sem encapsulamento — caos
public class Conta {
    public double saldo;
}

Conta c = new Conta();
c.saldo = -1000000;  // Saldo negativo arbitrário! Bug em potencial.
```

```java
// ✅ Com encapsulamento — proteção
public class Conta {
    private double saldo;  // ninguém mexe direto

    public void depositar(double valor) {
        if (valor <= 0) throw new IllegalArgumentException("Valor inválido");
        saldo += valor;
    }

    public double getSaldo() {
        return saldo;
    }
}
```

### Getters e Setters

Padrão clássico (mas não use cegamente):

```java
public class Pessoa {
    private String nome;

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
}
```

> ⚠️ **Cuidado com setters universais**: se você tem getter e setter pra tudo, encapsulamento é teatro. Pergunte: esse atributo **realmente** precisa ser modificável de fora?

### Imutabilidade > setters

```java
// ✅ Objeto imutável (preferível quando possível)
public final class Endereco {
    private final String rua;
    private final String cidade;

    public Endereco(String rua, String cidade) {
        this.rua = rua;
        this.cidade = cidade;
    }

    public String getRua() { return rua; }
    public String getCidade() { return cidade; }
    // sem setters!
}
```

### Java moderno: Records

Pra **objetos puramente de dados**, Java 16+ tem records:

```java
public record Endereco(String rua, String cidade) {}

// Equivalente a uma classe com:
// - construtor
// - getters (rua(), cidade())
// - equals, hashCode, toString
// - imutabilidade
```

---

## 3️⃣ Herança

### O Que é

Uma classe **estende** outra, herdando seus atributos e métodos. Modela relação **"é um"** (is-a).

> 💡 **Analogia**: cachorro é um animal. Tudo que animal faz (respirar, comer), cachorro também faz. Mas cachorro tem coisas próprias (latir).

### Em código

```java
// Classe pai (superclasse)
public class Animal {
    protected String nome;

    public void respirar() {
        System.out.println(nome + " está respirando");
    }

    public void comer() {
        System.out.println(nome + " está comendo");
    }
}

// Classe filha (subclasse)
public class Cachorro extends Animal {
    public void latir() {
        System.out.println(nome + " está latindo");
    }
}

// Uso
Cachorro rex = new Cachorro();
rex.nome = "Rex";
rex.respirar();  // herdou de Animal
rex.comer();     // herdou de Animal
rex.latir();     // próprio de Cachorro
```

### Override (sobrescrita)

Subclasse pode **redefinir** comportamento herdado:

```java
public class Animal {
    public void emitirSom() {
        System.out.println("Som genérico");
    }
}

public class Cachorro extends Animal {
    @Override
    public void emitirSom() {
        System.out.println("Au au!");
    }
}

public class Gato extends Animal {
    @Override
    public void emitirSom() {
        System.out.println("Miau!");
    }
}
```

> 💡 **Anotação `@Override`** é opcional, mas use sempre. O compilador avisa se você errou o nome do método sobrescrito.

### Super

Acessa membros da classe pai:

```java
public class Cachorro extends Animal {
    public Cachorro(String nome) {
        super.nome = nome;  // ou super() pro construtor
    }

    @Override
    public void emitirSom() {
        super.emitirSom();          // chama o do pai
        System.out.println("Au au!");
    }
}
```

### Limitação: Java tem herança simples

Uma classe **só pode estender uma** outra. Mas pode implementar **múltiplas interfaces**:

```java
public class Carro extends Veiculo implements Motorizado, Estacionavel {
    // ✅ funciona
}

public class X extends A, B {
    // ❌ erro de compilação
}
```

### ⚠️ Cuidado: prefira composição

Herança parece linda, mas tem armadilhas:
- 🔗 **Acoplamento forte** entre pai e filho
- 🦴 **Classe pai mudando** quebra todas as filhas
- 📚 **Hierarquias profundas** ficam difíceis de manter

> 💡 **Princípio**: "favor composition over inheritance" (favor composição em vez de herança).

```java
// ❌ Herança quando não faz sentido conceitual
public class CarrinhoCompras extends ArrayList<Produto> {
    public BigDecimal total() { /* ... */ }
}

// ✅ Composição — mais flexível
public class CarrinhoCompras {
    private List<Produto> produtos = new ArrayList<>();

    public void adicionar(Produto p) { produtos.add(p); }
    public BigDecimal total() { /* ... */ }
}
```

> 🤔 **Quando usar herança?** Quando a relação é genuinamente "é um". Cachorro **é um** Animal: ✅. Carrinho **é uma** lista? Não, ele **tem** uma lista: ❌.

---

## 4️⃣ Polimorfismo

### O Que é

"Muitas formas". Um mesmo método pode se comportar **diferente** dependendo do objeto.

> 💡 **Analogia**: o botão "play" funciona igual em vários aparelhos. No DVD, toca filme. No celular, toca música. No videogame, inicia jogo. Mesmo botão, comportamentos diferentes.

### Polimorfismo de subtipo (mais comum)

```java
public class Animal {
    public void emitirSom() { System.out.println("Som genérico"); }
}

public class Cachorro extends Animal {
    public void emitirSom() { System.out.println("Au au!"); }
}

public class Gato extends Animal {
    public void emitirSom() { System.out.println("Miau!"); }
}

// O pulo do gato (literalmente):
public void fazerBarulho(Animal animal) {
    animal.emitirSom();  // chama o método CERTO baseado no tipo real
}

// Uso
fazerBarulho(new Cachorro());  // "Au au!"
fazerBarulho(new Gato());      // "Miau!"
fazerBarulho(new Animal());    // "Som genérico"
```

> 💡 **Em runtime**, Java escolhe **qual método chamar** baseado no tipo **real** do objeto, não no tipo declarado. Isso é **dynamic dispatch**.

### Por que é poderoso

Você pode escrever código **genérico** que funciona com **qualquer** subtipo:

```java
List<Animal> zoologico = List.of(
    new Cachorro(),
    new Gato(),
    new Pato()
);

for (Animal animal : zoologico) {
    animal.emitirSom();  // cada um faz seu som
}
```

Adicionar novo animal? Só criar a classe. **Nenhum código existente muda.**

### Sobrecarga (overloading) — outro tipo de polimorfismo

Mesmo nome de método, **assinaturas diferentes**:

```java
public class Calculadora {
    public int somar(int a, int b) {
        return a + b;
    }

    public double somar(double a, double b) {
        return a + b;
    }

    public int somar(int a, int b, int c) {
        return a + b + c;
    }
}
```

Java decide qual chamar baseado nos tipos passados.

### Polimorfismo paramétrico (Generics)

Tipos como parâmetros — uma classe/método funciona com **vários tipos**:

```java
public class Caixa<T> {
    private T conteudo;

    public void colocar(T item) { this.conteudo = item; }
    public T pegar() { return conteudo; }
}

// Uso
Caixa<String> caixaTexto = new Caixa<>();
caixaTexto.colocar("Olá");

Caixa<Integer> caixaNum = new Caixa<>();
caixaNum.colocar(42);
```

---

## 🎯 Os 4 Pilares Trabalhando Juntos

Exemplo realista: sistema de notificações.

```java
// ABSTRAÇÃO: contrato comum
public interface Notificavel {
    void enviar(String destinatario, String mensagem);
}

// HERANÇA: classe base com lógica comum
public abstract class NotificacaoBase implements Notificavel {
    // ENCAPSULAMENTO: estado privado
    private final Logger log = LoggerFactory.getLogger(getClass());

    public void enviar(String destinatario, String mensagem) {
        log.info("Enviando notificação para {}", destinatario);
        validar(destinatario, mensagem);
        executarEnvio(destinatario, mensagem);
        log.info("Notificação enviada com sucesso");
    }

    private void validar(String dest, String msg) {  // ENCAPSULAMENTO
        if (dest == null || msg == null) {
            throw new IllegalArgumentException("Dados inválidos");
        }
    }

    // Subclasses devem implementar:
    protected abstract void executarEnvio(String dest, String msg);
}

// Implementações concretas
public class EmailNotificacao extends NotificacaoBase {
    @Override
    protected void executarEnvio(String dest, String msg) {
        // envia email
    }
}

public class SmsNotificacao extends NotificacaoBase {
    @Override
    protected void executarEnvio(String dest, String msg) {
        // envia SMS
    }
}

// POLIMORFISMO em ação
public class CampanhaService {
    private final List<Notificavel> canais;

    public CampanhaService(List<Notificavel> canais) {
        this.canais = canais;
    }

    public void notificarTodos(String dest, String msg) {
        for (Notificavel canal : canais) {
            canal.enviar(dest, msg);  // chama o certo pra cada tipo
        }
    }
}
```

**Análise:**
- 🏛️ **Abstração**: `Notificavel` esconde detalhes
- 🔒 **Encapsulamento**: `validar()` é privado, estado interno protegido
- 👨‍👩‍👧 **Herança**: classes específicas estendem `NotificacaoBase`
- 🎭 **Polimorfismo**: `CampanhaService` trata qualquer tipo de notificação igual

---

## 🚨 Anti-Patterns Comuns

### 1. Getters/setters sem propósito

```java
// ❌ Encapsulamento de fachada
public class Pessoa {
    private String nome;

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    // mesma coisa que ser public, mas com mais código
}
```

### 2. Herança em vez de composição

```java
// ❌ Engenheiro É uma Pessoa? Faz sentido.
//    Mas LeitorJSON É um Reader? Talvez faça mais sentido COMPOR.
```

### 3. Hierarquia profunda

```
Animal → Mamífero → Carnívoro → Felino → Doméstico → Persa
```

5 níveis de herança = pesadelo de manutenção.

### 4. God class (anti-encapsulamento)

Classe com 50 atributos públicos e 100 métodos. Faz tudo, encapsula nada.

### 5. Type checking em vez de polimorfismo

```java
// ❌ Anti-polimórfico
public void emitirSom(Object animal) {
    if (animal instanceof Cachorro) {
        System.out.println("Au!");
    } else if (animal instanceof Gato) {
        System.out.println("Miau!");
    }
}

// ✅ Polimorfismo de verdade
public void emitirSom(Animal animal) {
    animal.emitirSom();
}
```

---

## 💡 Dicas Práticas

### 1. Pense em **comportamento**, não só dados

Objetos não são só "containers de dados" — eles **fazem coisas**.

### 2. Imutabilidade quando possível

Reduz bugs sutis, simplifica concorrência.

### 3. Composição > Herança (geralmente)

Mais flexível, menos acoplamento.

### 4. SOLID se aplica aqui

Os princípios **SOLID** ([visto em Arquitetura](./15-arquitetura-software.md)) são basicamente "como aplicar POO bem".

### 5. Não é receita de bolo

Às vezes, código procedural simples é melhor que OO sobre-engenheirado. Bom dev sabe escolher.

---

## ✅ Resumo da página

Os **4 pilares da POO**:

- 🎭 **Abstração**: esconder complexidade, expor só essencial (interfaces, classes abstratas)
- 🔒 **Encapsulamento**: proteger estado interno, controlar acesso (private, modificadores)
- 👨‍👩‍👧 **Herança**: classe filha estende pai, modela "é um" — **mas prefira composição**
- 🎨 **Polimorfismo**: mesma operação, comportamentos diferentes baseado no tipo

**Boas práticas:**
- Imutabilidade > setters quando possível
- Composição > Herança
- `@Override` sempre que sobrescrever
- Não confunda "é um" com "tem um"
- POO bem feita = SOLID aplicado

---

⬅️ [Anterior: Lógica e Paradigmas](./39-logica-paradigmas.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Java - Da Linguagem ao Ecossistema](./41-java-completo.md)
