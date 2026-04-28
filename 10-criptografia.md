# 🔐 Criptografia: Protegendo Dados

> Toda autenticação, todo HTTPS, toda senha guardada no banco passa por criptografia. Como dev, você não precisa virar criptógrafo — mas precisa entender o suficiente pra **não cometer erros catastróficos**.

---

## ⚠️ Regra de Ouro

**NUNCA invente seu próprio algoritmo de criptografia.** Use bibliotecas estabelecidas e battle-tested. Criptografia "caseira" sempre tem furos.

> 💡 Schneier's Law: "qualquer um pode criar um sistema criptográfico que ele mesmo não consegue quebrar". O ponto é se outras pessoas conseguem.

---

## 🔍 Hash ≠ Criptografia

Confusão comum. São coisas diferentes pra propósitos diferentes.

### 🌀 Hash: Mão única

Transforma qualquer entrada em uma saída de tamanho fixo. **Não dá pra reverter.**

```
"senha123"        → SHA-256 → ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f
"senha124"        → SHA-256 → 3f2a4b6c8d... (totalmente diferente)
"senha123" + sal  → SHA-256 → resultado ainda diferente
```

Propriedades:
- ✅ Mesmo input → mesmo output (determinístico)
- ✅ Pequena mudança → output completamente diferente
- ✅ Impossível reverter (computacionalmente)
- ✅ Resistente a colisões (raro dois inputs gerarem mesmo hash)

### 🔓 Criptografia: Mão dupla

Transforma dados de forma **reversível** com uma chave.

```
"meu segredo" + chave → criptografar → "x9k2#mP@..." (cifra)
"x9k2#mP@..." + chave → descriptografar → "meu segredo"
```

---

## 🔐 Hash em Senhas: O Caso Mais Comum

### ❌ Como NÃO fazer

```java
// NUNCA! MD5 é quebrado faz 20 anos
String hash = MD5.hash("senha123");

// Também ruim — SHA-256 é rápido demais (vulnerável a brute force)
String hash = sha256("senha123");
```

### ✅ Como fazer

Use algoritmos **lentos por design** (deliberadamente caros pra tornar brute force inviável):

| Algoritmo | Status |
|---|---|
| **bcrypt** | Padrão antigo, ainda OK |
| **scrypt** | Mais resistente a hardware |
| **Argon2** | Estado da arte (vencedor de competição) |

### Salt: Por que importa

Sem salt, todo mundo com a senha "123456" tem o mesmo hash. Atacantes podem usar **rainbow tables** (tabelas pré-computadas).

**Salt** é um valor aleatório único por usuário, somado à senha antes do hash:

```
hash("senha123" + "Y7$mK9pQ") → resultado_único_por_usuario
```

### Exemplo: Java + Spring

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);  // 12 rounds (lento de propósito)
}

// Cadastrar
String hash = passwordEncoder.encode("senha123");
// Salva no banco: $2a$12$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy

// Verificar login
boolean ok = passwordEncoder.matches("senha123", hashDoBanco);
```

### Equivalentes em outras stacks

```typescript
// Node.js — bcryptjs ou argon2
import bcrypt from 'bcryptjs';
const hash = await bcrypt.hash('senha123', 12);
const ok = await bcrypt.compare('senha123', hashDoBanco);
```

```python
# Python — bcrypt ou argon2-cffi
import bcrypt
hash = bcrypt.hashpw(b'senha123', bcrypt.gensalt(12))
ok = bcrypt.checkpw(b'senha123', hashDoBanco)
```

> 🤔 **Por que `matches()`/`compare()` em vez de fazer hash e comparar?** Porque o salt está embutido no hash. Cada hash é único — só essas funções conseguem extrair o salt e refazer o cálculo corretamente.

---

## 🤝 Criptografia Simétrica

Mesma chave criptografa e descriptografa.

```
Alice ─────[chave secreta]─────→ Bob
       (cifra com a chave,
        Bob descifra com a mesma)
```

Algoritmos:
- **AES** (Advanced Encryption Standard) — padrão atual, muito rápido
- ❌ DES e 3DES — obsoletos

✅ Rápido, eficiente
❌ Como vocês compartilham a chave **com segurança** antes da comunicação?

### Quando usar

- Criptografar arquivos no disco
- Backups
- Dados em repouso no banco

---

## 🗝️ Criptografia Assimétrica (Chave Pública)

Você tem **duas chaves matematicamente ligadas**:

- 🔓 **Chave pública**: você compartilha com todo mundo
- 🔐 **Chave privada**: você guarda em segredo

```
Bob quer mandar mensagem secreta pra Alice:

1. Bob pega a CHAVE PÚBLICA da Alice (qualquer um pode ter)
2. Bob criptografa a mensagem com essa chave
3. Resultado: só a CHAVE PRIVADA da Alice consegue descriptografar
```

> 💡 **Analogia**: caixa de correios. Qualquer um pode botar carta dentro (chave pública), mas só quem tem a chave do prédio pode abrir e ler (chave privada).

Algoritmos:
- **RSA** — clássico, ainda muito usado
- **ECC** (Elliptic Curve) — chaves menores, mesma segurança

✅ Resolve o problema de troca de chave
❌ Lento (não usado pra grandes volumes de dados)

### Como o HTTPS realmente funciona

A genialidade do TLS/HTTPS é **combinar os dois**:

1. Cliente conecta no servidor
2. Servidor manda **certificado** com sua chave pública
3. Cliente gera uma **chave simétrica aleatória**
4. Cliente criptografa essa chave com a chave pública do servidor
5. Servidor descriptografa com sua chave privada
6. **Daqui pra frente, comunicação usa a chave simétrica** (rápida)

> 🤔 Resultado: segurança da assimétrica + velocidade da simétrica.

---

## ✍️ Assinatura Digital

Mesma matemática, uso inverso:

- Você **assina** com sua chave **privada**
- Outros **verificam** com sua chave **pública**

Isso prova:
- ✅ Autenticidade (foi você mesmo)
- ✅ Integridade (ninguém alterou)
- ✅ Não-repúdio (você não pode negar depois)

Usado em: certificados HTTPS, commits Git assinados, JWT, transações de criptomoedas.

---

## 🎫 JWT: JSON Web Tokens

**JWT** é um padrão pra transportar informações assinadas. Você usa muito em autenticação de APIs.

### Estrutura

Um JWT tem 3 partes separadas por `.`:

```
xxxxx.yyyyy.zzzzz
header.payload.signature
```

Exemplo decodificado:

**Header** (algoritmo):
```json
{ "alg": "HS256", "typ": "JWT" }
```

**Payload** (dados do usuário):
```json
{
  "sub": "1234567890",
  "name": "Mauricio",
  "role": "admin",
  "exp": 1735689600
}
```

**Signature**: HMAC-SHA256 do header + payload + chave secreta.

### Como funciona

```
1. Login: usuário manda email/senha
2. Servidor valida e gera JWT (assinando com chave secreta)
3. Cliente guarda o JWT (localStorage, cookie httpOnly)
4. Cada request: cliente manda JWT no header Authorization
5. Servidor valida assinatura com a mesma chave secreta
   - Se válido, lê os dados do payload (sem precisar consultar DB!)
```

### ⚠️ Cuidados com JWT

- 🔓 **Payload é APENAS Base64**, não criptografado. Não bote dados sensíveis lá
- ⏰ **Sempre defina expiração curta** (`exp`)
- 🔄 **Use refresh tokens** pra renovar sem pedir login de novo
- 🚫 **Revogar JWT é difícil** (não tem estado no servidor) — em sistemas críticos, considere alternativas
- 🍪 Em apps web, **httpOnly cookies > localStorage** (proteção contra XSS)

> 💡 **Bibliotecas por stack**:
> - **Java**: `spring-security` + `jjwt` ou `nimbus-jose-jwt`
> - **Node.js**: `jsonwebtoken`, `jose`
> - **Python**: `pyjwt`, `python-jose`
> - **.NET**: `System.IdentityModel.Tokens.Jwt`
> - **Go**: `golang-jwt/jwt`

---

## 🛡️ Outros Conceitos Importantes

### HMAC

Assinatura usando hash + chave secreta. Garante integridade + autenticidade. Usado em webhooks (GitHub, Stripe), APIs.

### TOTP (Time-based One-Time Password)

A base do **2FA com Google Authenticator**. Algoritmo gera código de 6 dígitos baseado em tempo + chave compartilhada.

### Hashing pra deduplicação/integridade

Sites de download mostram um hash SHA-256 do arquivo. Você baixa, calcula o hash localmente, compara. Se bater, o arquivo não foi corrompido nem alterado.

```bash
sha256sum arquivo.iso
# Compara com o publicado pelo site
```

---

## 🚨 Erros Comuns que Você Deve Evitar

❌ Usar MD5 ou SHA-1 pra senhas
❌ Não usar salt
❌ Inventar algoritmo próprio
❌ Botar dados sensíveis em payload de JWT (achando que está criptografado)
❌ Hardcoded de chaves secretas no código (usar variáveis de ambiente, secret managers)
❌ Comparar strings de hash com `==` (use comparação **constant-time** pra evitar timing attacks)
❌ Misturar hash de propósito geral (SHA) com hash de senha (deve ser bcrypt/argon2)
❌ Reusar a mesma chave em contextos diferentes

---

## ✅ Resumo da página

- **Hash** é mão única; **criptografia** é reversível
- Senhas: **NUNCA MD5/SHA**, sempre **bcrypt/Argon2** com salt
- **Simétrica** (AES): rápida, mesma chave nos dois lados
- **Assimétrica** (RSA): chave pública/privada, resolve troca de chave
- **HTTPS** combina os dois: assimétrica pra trocar chave + simétrica pra comunicação
- **JWT** é assinado, não criptografado — não bote segredos no payload
- **Nunca invente** algoritmo próprio; use bibliotecas estabelecidas
- Use **variáveis de ambiente** ou **secret managers** pra chaves, nunca hardcoded

---

⬅️ [Anterior: Concorrência](./09-concorrencia.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Compiladores](./11-compiladores.md)
