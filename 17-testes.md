# 🧪 Testes

> Código sem testes é código sem rede de proteção. Com testes, você refatora sem medo, deploya com confiança e dorme melhor. Mas testar TUDO sem critério é desperdício — saber **o quê e quanto** testar é a habilidade real.

---

## 🎯 Por que testar

- 🛡️ **Confiança pra mudar código**: refatoração sem terror
- 📚 **Documentação viva**: testes mostram como o código deve ser usado
- 🐛 **Detecção precoce**: bugs pegos antes de produção custam 10x menos
- 🏗️ **Design melhor**: código difícil de testar geralmente é mal arquitetado

> 💡 **Verdade desconfortável**: quem nunca testou em alguns projetos sabe — quando o time cresce e o código fica complexo, você **precisa** de testes pra não virar refém do código.

---

## 🔺 A Pirâmide de Testes

```
        /\
       /E2E\          ← Poucos, lentos, caros (UI completa)
      /──────\
     /  Integ  \      ← Médio (camadas conversando)
    /───────────\
   /   Unit      \    ← Muitos, rápidos, baratos
  /───────────────\
```

### Por que pirâmide?

- ⚡ Testes unitários **rodam em ms** → você roda toda hora
- 🐌 Testes E2E **demoram minutos** → só rodam no CI
- 💰 E2E é **caro** de escrever e manter

> ⚠️ **Anti-pattern (cone invertido)**: muito E2E, pouco unitário. Pipeline lento, builds quebrando por motivo bobo, time desmotivado.

---

## 🧩 Tipos de Teste

### 1️⃣ Unitário

Testa **uma unidade** isolada (uma função, uma classe). Sem DB, sem rede, sem arquivo.

```java
@Test
void deveCalcularDescontoCom10PorCento() {
    Calculadora calc = new Calculadora();
    double resultado = calc.aplicarDesconto(100.0, 0.1);
    assertEquals(90.0, resultado);
}
```

✅ **Rápido** (milissegundos)
✅ **Determinístico** (sempre o mesmo resultado)
✅ Falha aponta exatamente onde está o bug

### 2️⃣ Integração

Testa **várias unidades juntas**. Pode envolver DB real, frameworks, etc.

```java
@SpringBootTest
@AutoConfigureMockMvc
class UsuarioControllerIT {
    @Autowired
    MockMvc mockMvc;

    @Test
    void deveCriarUsuario() throws Exception {
        mockMvc.perform(post("/usuarios")
                .content("{\"nome\":\"Mauricio\"}"))
            .andExpect(status().isCreated());
    }
}
```

✅ Pega problemas de **wiring** (Spring config, JPA, etc)
❌ Mais lento, mais frágil

### 3️⃣ End-to-End (E2E)

Testa o **sistema inteiro**, do clique do usuário até o banco.

```typescript
// Cypress / Playwright
test('login com sucesso', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#email', 'mau@example.com');
  await page.fill('#senha', '123456');
  await page.click('button[type=submit]');
  await expect(page).toHaveURL('/dashboard');
});
```

✅ Garante que o **fluxo do usuário** funciona
❌ Lento, frágil (qualquer mudança visual quebra)

### 4️⃣ Outros úteis

| Tipo | Pra quê |
|---|---|
| **Smoke** | "Subiu? Tá vivo?" — testes mínimos |
| **Regressão** | Garantir que bug antigo não volte |
| **Performance** | Latência, throughput sob carga |
| **Carga** (load) | Quantos usuários aguenta |
| **Stress** | Onde quebra |
| **Contrato** | API segue o schema combinado |
| **Mutation** | Testa seus próprios testes |

---

## 📐 AAA: Arrange, Act, Assert

Padrão pra estruturar qualquer teste:

```java
@Test
void deveRetornarSomaCorreta() {
    // ARRANGE: prepara o cenário
    Calculadora calc = new Calculadora();
    int a = 5;
    int b = 3;

    // ACT: executa a ação
    int resultado = calc.somar(a, b);

    // ASSERT: verifica o resultado
    assertEquals(8, resultado);
}
```

> 💡 **Em BDD** isso vira "Given/When/Then" — mesma coisa.

---

## 🎭 Mocks, Stubs, Fakes, Spies

Quando a unidade testada **depende de outra coisa** (DB, API externa), você "finge" essa dependência.

### Tipos de "test doubles"

| Tipo | O que é |
|---|---|
| **Dummy** | Objeto vazio só pra preencher assinatura |
| **Stub** | Retorna respostas pré-programadas |
| **Mock** | Stub + verifica se foi chamado corretamente |
| **Fake** | Implementação real mas simplificada (ex: in-memory DB) |
| **Spy** | Wrapper que registra chamadas |

### Mockito (Java)

```java
@Test
void deveSalvarUsuario() {
    // Arrange
    UsuarioRepository repo = mock(UsuarioRepository.class);
    Usuario usuario = new Usuario("Mauricio");
    when(repo.save(any())).thenReturn(usuario);

    UsuarioService service = new UsuarioService(repo);

    // Act
    Usuario resultado = service.cadastrar("Mauricio");

    // Assert
    assertEquals("Mauricio", resultado.getNome());
    verify(repo, times(1)).save(any());
}
```

> ⚠️ **Cuidado com over-mocking**. Se você mocka tudo, está testando que mocks funcionam, não seu código. Mocke **dependências externas**, não tudo.

---

## 🔬 TDD (Test-Driven Development)

Ciclo: **Red → Green → Refactor**

```
🔴 RED: escreve um teste que falha (feature ainda não existe)
🟢 GREEN: escreve o MÍNIMO de código pra teste passar
♻️ REFACTOR: melhora o código mantendo testes verdes
```

### Vantagens

- ✅ Você só escreve código que tem propósito (existe teste pedindo)
- ✅ Testabilidade vem de graça (você desenha pra ser testável)
- ✅ Cobertura alta naturalmente

### Desvantagens

- ❌ Curva de aprendizado
- ❌ Lento no começo
- ❌ Difícil em código exploratório (UI, prototipos)

> 💡 **Não precisa fazer TDD em tudo.** Use onde faz sentido (lógica de negócio crítica). Em código exploratório, codar primeiro e testar depois é OK.

---

## 📊 Cobertura de Código

Quanto do seu código é executado pelos testes.

```bash
# Java/Maven com JaCoCo
mvn test jacoco:report

# JS com Jest
jest --coverage
```

### Métricas

- **Line coverage**: % de linhas executadas
- **Branch coverage**: % de ifs/elses cobertos
- **Method coverage**: % de métodos chamados

### ⚠️ Cobertura ≠ Qualidade

100% de cobertura **não** garante que seus testes são bons. Você pode chamar todo método sem fazer assertion alguma.

> 🤔 **Meta razoável**: 70-80% em código de produção. 100% em código crítico (cálculos financeiros, regras de negócio).

---

## 🍃 Testes em Frameworks Web

> 💡 Os exemplos abaixo são em **Spring Boot (Java)**, mas conceitos análogos existem em qualquer stack: **NestJS** (`@nestjs/testing`), **FastAPI** (`TestClient`), **ASP.NET** (`WebApplicationFactory`), **Express** (supertest).

### Anotações principais (Spring Boot)

| Annotation | Pra quê |
|---|---|
| `@SpringBootTest` | Carrega contexto inteiro |
| `@WebMvcTest` | Só camada web |
| `@DataJpaTest` | Só JPA + H2 in-memory |
| `@MockBean` | Mock dentro do contexto Spring |
| `@Testcontainers` | Containers reais (Postgres, Redis) nos testes |

### Testcontainers: O Padrão Ouro

Em vez de mockar o banco, **rode um Postgres real em container** durante o teste:

```java
@Testcontainers
@SpringBootTest
class UsuarioRepositoryIT {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry reg) {
        reg.add("spring.datasource.url", postgres::getJdbcUrl);
        reg.add("spring.datasource.username", postgres::getUsername);
        reg.add("spring.datasource.password", postgres::getPassword);
    }

    @Test
    void deveSalvarUsuario() {
        // testa contra Postgres REAL
    }
}
```

> 💡 **Por que isso é fenomenal?** SQLite ou H2 in-memory têm comportamento diferente do banco real. Testcontainers garante que seu teste passa **com o mesmo banco da produção**. Disponível pra Java, Node.js, Python, .NET, Go.

---

## 🧪 Boas Práticas

### ✅ Faça

- **Nomes descritivos**: `deveLancarExcecaoQuandoEmailJaCadastrado` em vez de `test1`
- **Um conceito por teste**: cada `@Test` valida uma coisa
- **Independência**: testes podem rodar em qualquer ordem
- **Determinístico**: mesmo input = mesmo resultado, sempre
- **Rápido**: < 1 segundo pra unitários

### ❌ Evite

- **Testar implementação**: teste o **comportamento**, não detalhes internos
- **Testes frágeis**: que quebram a cada refactor pequeno
- **Lógica complexa em teste**: se o teste tem `if`, algo está errado
- **Comentários explicando o teste**: o nome e estrutura devem ser óbvios
- **Mocks demais**: testa mocks, não código

---

## 🎯 Quanto Testar?

| Tipo de código | Cobertura recomendada |
|---|---|
| Regras de negócio críticas | ~100% |
| Casos de uso / serviços | 80-90% |
| Controllers / handlers | 70%+ (testes de integração) |
| Getters/setters/DTOs | Não teste |
| Código de configuração | Não teste |
| Frontend de UI estática | Não teste, talvez snapshot |

---

## ✅ Resumo da página

- Pirâmide: **muito unitário, médio integração, pouco E2E**
- **Unitário** é rápido e isolado; **integração** valida wiring; **E2E** valida fluxos
- Use **AAA** (Arrange, Act, Assert) pra estruturar
- **Mocks** pra dependências externas, não pra tudo
- **TDD** ajuda em lógica de negócio, não força tudo
- **Cobertura ≠ qualidade** — 100% pode ter zero asserts úteis
- Em Spring (e equivalentes em outras stacks): **Testcontainers** é melhor que mocks de DB
- Teste **comportamento, não implementação**

---

⬅️ [Anterior: DevOps e CI/CD](./16-devops-cicd.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Observabilidade](./18-observabilidade.md)
