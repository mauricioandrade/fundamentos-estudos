# 📜 LGPD e GDPR pra Devs

> Leis de proteção de dados não são "papo de jurídico" — afetam diretamente como você modela banco, escreve código e desenha sistemas. Multas chegam a centenas de milhões. Como dev, você é a primeira linha de defesa.

---

## 🎯 As Duas Leis Principais

### 🇪🇺 GDPR (General Data Protection Regulation)

- Vigente desde **maio de 2018** na União Europeia
- Aplica-se a **qualquer empresa** que processe dados de cidadãos europeus, mesmo fora da UE
- Multas: até **€20 milhões** ou **4% do faturamento global** (o que for maior)

### 🇧🇷 LGPD (Lei Geral de Proteção de Dados — Lei 13.709/2018)

- Vigente desde **setembro de 2020** no Brasil
- Inspirada no GDPR, com adaptações
- Multas: até **R$ 50 milhões** por infração ou **2% do faturamento**

> 💡 As duas leis são **muito parecidas**. Quem implementa uma, geralmente está em conformidade com a outra.

---

## 📋 Vocabulário Essencial

| Termo | Significa |
|---|---|
| **Dados pessoais** | Qualquer info que identifica pessoa (nome, email, IP, foto, CPF) |
| **Dados sensíveis** | Subcategoria especial (saúde, religião, orientação sexual, biometria) |
| **Titular** | A pessoa "dona" dos dados |
| **Controlador** | Empresa que decide como tratar dados (geralmente sua empresa) |
| **Operador** | Empresa que processa dados em nome do controlador (ex: AWS) |
| **DPO** | Data Protection Officer — pessoa responsável por compliance |
| **Tratamento** | Qualquer operação com dado: coleta, armazenamento, uso, exclusão |
| **Consentimento** | Permissão clara, específica e revogável do titular |

---

## 🛡️ Os 6 Princípios Fundamentais

Tanto GDPR quanto LGPD se baseiam nesses pilares:

### 1. Finalidade

Você só pode coletar dado pra um **propósito específico**. Não vale "vou guardar caso precise depois".

```
❌ Coletar CPF "porque sim"
✅ Coletar CPF "pra emitir nota fiscal" (e só pra isso)
```

### 2. Adequação

Dado coletado deve ser **compatível** com o propósito declarado.

### 3. Necessidade

Colete **o mínimo necessário**. Não peça data de nascimento se você só precisa saber se é maior de idade.

```
❌ "Qual sua data de nascimento?" (pra saber se é +18)
✅ "Você é maior de 18 anos? [sim/não]"
```

### 4. Transparência

Usuário precisa **saber claramente** o que você faz com os dados.

### 5. Segurança

Você é **responsável** por proteger os dados.

### 6. Prestação de contas

Precisa **provar** que segue a lei. Não basta dizer — precisa documentar.

---

## ⚖️ Bases Legais (Por Que Você Pode Tratar Dados)

Pra tratar qualquer dado pessoal, você precisa de **uma base legal**. As principais:

### 1. Consentimento

Usuário aceitou **explicitamente**. Pode **revogar** a qualquer momento.

```
❌ Checkbox pré-marcado
❌ "Ao usar o site você concorda com..."
✅ Checkbox vazio + texto claro: "Aceito receber emails de marketing"
```

### 2. Execução de contrato

Pra cumprir um contrato com o usuário (ex: pegar endereço pra entregar produto comprado).

### 3. Obrigação legal

Lei obriga (ex: dados fiscais).

### 4. Legítimo interesse

Interesse legítimo do controlador, **balanceado** com direitos do titular. Cuidado — abusado.

### 5. Proteção de vida

Salvar vida (saúde, emergências).

### 6. Exercício regular de direitos

Em processos judiciais, etc.

> 💡 **Regra de ouro**: identifique a base legal **antes** de coletar. Se não tem base, não colete.

---

## 👤 Direitos dos Titulares

Usuários têm direitos que **você precisa implementar tecnicamente**:

| Direito | O que precisa |
|---|---|
| 📋 **Acesso** | Mostrar todos os dados que você tem dele |
| ✏️ **Correção** | Permitir editar dados incorretos |
| 🗑️ **Exclusão** | Deletar de verdade ("right to be forgotten") |
| 📦 **Portabilidade** | Exportar dados em formato útil (JSON, CSV) |
| 🚫 **Oposição** | Parar processamento que ele não quer |
| 🤖 **Decisões automatizadas** | Direito a revisão humana |
| 🔍 **Informação** | Saber pra quem você compartilha dados |

> ⚠️ Você precisa **responder em até 15 dias** (LGPD) ou 30 dias (GDPR).

---

## 💻 Implicações Práticas pra Devs

### 1. Modelagem de banco

Dados pessoais devem ser **identificáveis e separáveis**.

```sql
-- ❌ Dados misturados sem distinção
CREATE TABLE pedidos (
  id INT,
  produto VARCHAR,
  comprador_nome VARCHAR,
  comprador_email VARCHAR,
  comprador_cpf VARCHAR,
  -- ...
);

-- ✅ Dados pessoais centralizados
CREATE TABLE usuarios (
  id INT PRIMARY KEY,
  nome VARCHAR,
  email VARCHAR,
  cpf VARCHAR,
  data_consentimento TIMESTAMP,
  consentimento_marketing BOOLEAN
);

CREATE TABLE pedidos (
  id INT,
  usuario_id INT REFERENCES usuarios(id),
  produto VARCHAR
);
```

Com isso, "esquecer um usuário" é mais fácil:

```sql
DELETE FROM pedidos WHERE usuario_id = ?;
DELETE FROM usuarios WHERE id = ?;
```

### 2. Soft delete vs Hard delete

Pra LGPD/GDPR, **hard delete** geralmente é necessário pra "esquecimento".

```sql
-- ❌ Soft delete não é suficiente
UPDATE usuarios SET deleted_at = NOW() WHERE id = ?;

-- ✅ Anonimização ou exclusão real
UPDATE usuarios SET
  nome = 'EXCLUÍDO',
  email = NULL,
  cpf = NULL
WHERE id = ?;

-- Ou DELETE de fato
DELETE FROM usuarios WHERE id = ?;
```

> 🤔 **Dilema**: você precisa manter logs por anos pra fins fiscais/legais, mas também "esquecer" usuários. Solução: **anonimização** mantém o registro sem dados pessoais.

### 3. Pseudonimização e Anonimização

#### Pseudonimização

Substitui dado por **chave reversível**. Ainda é dado pessoal pela LGPD/GDPR.

```
"João Silva, CPF 123.456.789-00" → "Usuário #abc123"
                                    (chave fica em tabela separada protegida)
```

#### Anonimização

Remove identificadores **irreversivelmente**. Não é mais dado pessoal.

```
"João Silva, 35 anos, SP" → "Homem, 30-40 anos, sudeste"
```

### 4. Logs sem dados sensíveis

```java
// ❌ NUNCA
log.info("Usuário {} entrou com senha {}", email, senha);
log.error("Falha ao processar CPF {}", cpf);

// ✅
log.info("Usuário ID {} fez login", usuarioId);
log.error("Falha ao processar usuário ID {}", usuarioId);
```

> ⚠️ Se logs vazarem (e vazam), você não quer dados pessoais lá.

### 5. Criptografia em repouso

Dados sensíveis (CPF, dados de saúde) devem estar **criptografados no banco**.

```java
@Convert(converter = AesConverter.class)
private String cpf;
```

Em caso de vazamento, dados criptografados não são dados pessoais (se a chave estiver protegida).

### 6. Criptografia em trânsito

HTTPS **sempre**. Não tem desculpa em 2026.

### 7. Acesso mínimo (princípio do menor privilégio)

Quem **precisa ver** dados pessoais pra fazer o trabalho? Apenas essas pessoas devem ter acesso.

```
❌ Toda equipe de dev tem acesso ao banco de produção
✅ Acesso só pra DBA + alguns devs específicos com auditoria
```

### 8. Auditoria de acesso

Logs de quem acessou o quê:

```java
@PostMapping("/usuarios/{id}")
public Usuario buscar(@PathVariable Long id, Authentication auth) {
    auditService.log(auth.getName(), "READ", "Usuario", id);
    return repo.findById(id).orElseThrow();
}
```

---

## 🍪 Cookies e Tracking

### Tipos de cookies

| Tipo | Precisa consentimento? |
|---|---|
| **Essenciais** (login, carrinho) | ❌ Não |
| **Funcionais** (preferências) | ⚠️ Depende |
| **Analytics** (Google Analytics) | ✅ Sim |
| **Marketing** (Facebook Pixel) | ✅ Sim |

### Cookie banner correto

```
❌ "Ao continuar, você aceita nossos cookies"
   (consentimento implícito não vale)

✅ Banner com:
   - Aceitar todos
   - Recusar todos
   - Personalizar (escolher categorias)
   - Mesmo destaque visual em "Aceitar" e "Recusar"
```

> 🚨 **CNIL (França) multou** Google e Facebook em **€150M+ cada** por banners onde "recusar" era mais difícil que "aceitar".

---

## 🌍 Transferência Internacional de Dados

Mandar dado pra fora do país? Tem regras.

### LGPD permite se:

- País destino tem nível adequado de proteção
- Empresas têm contratos específicos (cláusulas padrão)
- Consentimento específico do titular
- Outras situações listadas na lei

### Cuidado com cloud providers

AWS US-East-1 é nos EUA. Se você processa dados de brasileiros lá, tecnicamente é transferência internacional.

> 💡 **Soluções práticas**:
> - Use região local (AWS São Paulo, GCP southamerica-east1)
> - Garanta que provider tem cláusulas contratuais
> - Documente a transferência

---

## 🚨 Em Caso de Vazamento

GDPR e LGPD exigem:

- 🚨 **Notificação à autoridade** em até **72 horas**
- 📧 **Notificação aos titulares** afetados se houver risco alto
- 📝 **Documentação** completa do incidente

### Plano de resposta a incidentes

Antes de acontecer:

1. Time identificado (DPO, DevSecOps, Comunicação, Jurídico)
2. Procedimento documentado
3. Templates de comunicação prontos
4. Logs e ferramentas pra investigação prontos

---

## 📋 Checklist Mínimo pra Devs

### Coleta de dados
- [ ] Tem base legal pra cada dado coletado
- [ ] Coleta apenas o necessário
- [ ] Consentimento documentado quando necessário
- [ ] Política de privacidade clara e acessível

### Armazenamento
- [ ] Dados pessoais em tabelas/coleções identificáveis
- [ ] Dados sensíveis criptografados em repouso
- [ ] Backups também protegidos
- [ ] Retenção definida (não guarda pra sempre)

### Processamento
- [ ] HTTPS em todas as conexões
- [ ] Acesso mínimo (RBAC)
- [ ] Logs de auditoria
- [ ] Logs sem dados pessoais sensíveis

### Direitos do titular
- [ ] Endpoint pra usuário ver seus dados
- [ ] Endpoint pra usuário corrigir
- [ ] Endpoint pra usuário exportar (JSON/CSV)
- [ ] Endpoint pra usuário deletar
- [ ] Processo de revogação de consentimento

### Operacional
- [ ] DPO designado (obrigatório em algumas situações)
- [ ] Plano de resposta a incidentes
- [ ] Treinamento da equipe
- [ ] Avaliação de impacto (DPIA) pra processamentos arriscados

---

## 🛠️ Ferramentas Úteis

### Detecção de PII em código/dados

- **Microsoft Presidio** (open source)
- **AWS Comprehend** (paga, gerenciada)
- **Google DLP** (paga)

### Cookie management

- **Cookiebot**
- **OneTrust**
- **Osano**

### Anonimização

- **ARX Data Anonymization** (open source, Java)
- **Faker** (gerar dados fake pra dev/test)

### DSR automation (Data Subject Requests)

- **OneTrust**, **TrustArc**, **DataGrail**

---

## 🎯 Casos Famosos

- **Meta**: multa de **€1.2 bilhão** em 2023 por transferir dados pros EUA
- **Amazon**: **€746 milhões** em 2021 (Luxemburgo)
- **Google**: **€90 milhões** em 2022 (cookies França)
- 🇧🇷 No Brasil, ANPD (Autoridade Nacional) já aplicou multas em 2024-2025

> 💡 **Mensagem clara**: leis são levadas a sério. Compliance não é opcional.

---

## ✅ Resumo da página

- **GDPR (UE)** e **LGPD (BR)** protegem dados pessoais — multas pesadas
- **Princípios**: finalidade, adequação, **necessidade** (colete o mínimo), transparência, segurança
- Sempre identifique a **base legal** antes de coletar
- Implemente os **direitos do titular**: acesso, correção, exclusão, portabilidade
- **Modele banco** pra que dados pessoais sejam identificáveis e excluíveis
- **Anonimize** pra manter dados úteis sem riscos
- **Não logue** dados sensíveis
- **Criptografia** em repouso e em trânsito
- **Cookie banners** precisam permitir recusa fácil
- Tenha **plano de resposta a incidentes** pronto
- Compliance é responsabilidade de **todos**, não só do jurídico

---

⬅️ [Anterior: Internacionalização](./36-i18n.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Soft Skills pra Devs](./38-soft-skills.md)
