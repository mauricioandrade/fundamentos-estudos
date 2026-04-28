# 🌿 Git e Versionamento

> Git é a ferramenta mais importante que todo dev usa todo dia — e a que mais gera confusão. Entender como ele funciona **por baixo** transforma "comandos mágicos" em ferramentas previsíveis.

---

## 🤔 O que Git resolve

Antes do Git, salvar versões de código era caos:

```
projeto/
├── codigo_v1.zip
├── codigo_v2_final.zip
├── codigo_v2_FINAL_DE_VERDADE.zip
├── codigo_v2_FINAL_revisado_pelo_chefe.zip  ← qual é o atual???
```

Git resolve:
- 📜 Histórico completo de cada mudança
- 🔀 Várias pessoas trabalhando no mesmo arquivo sem se atrapalhar
- ⏪ Voltar pra qualquer ponto do passado
- 🌿 Experimentar coisas em **branches** sem afetar o código principal

> 💡 **Git ≠ GitHub**. Git é a ferramenta (criada pelo Linus Torvalds em 2005). GitHub é uma plataforma que hospeda repos Git com extras (issues, PRs, CI/CD).

---

## 🧠 Como Git Pensa: Snapshots, Não Diffs

Aqui que muita gente se confunde. Git **não guarda diferenças** entre versões — ele guarda **snapshots completos** do projeto a cada commit.

```
Commit 1:  [arq.txt v1] [outro.js v1]
Commit 2:  [arq.txt v1] [outro.js v2]    ← arq.txt apenas referencia v1
Commit 3:  [arq.txt v2] [outro.js v2]
```

Arquivos que não mudaram apenas **referenciam** a versão anterior. Eficiente em espaço e velocidade.

---

## 🗄️ Os 3 Estados do Git

Todo arquivo num repo Git está em um destes estados:

```
[Working Directory] ──git add──→ [Staging Area] ──git commit──→ [Repository]
   (seus arquivos)                (preparação)              (histórico permanente)
```

| Estado | O que é |
|---|---|
| **Working Directory** | Os arquivos como você vê na sua pasta |
| **Staging Area** (Index) | Mudanças marcadas pra entrar no próximo commit |
| **Repository** | Histórico de commits, em `.git/` |

> 💡 **Por que existe staging area?** Pra você selecionar **quais mudanças** entram no próximo commit. Você pode mudar 5 arquivos e commitar só 2 deles, deixando os outros pra outro commit.

---

## 🚀 Comandos do Dia a Dia

### Setup inicial (uma vez)

```bash
git config --global user.name "Mauricio"
git config --global user.email "mau@example.com"
git config --global init.defaultBranch main
```

### Começar um repo

```bash
git init                              # repo novo local
git clone https://github.com/u/r.git  # baixar de remoto
```

### Fluxo básico

```bash
git status                  # ver o que mudou
git add arquivo.txt         # adiciona ao staging
git add .                   # adiciona TUDO
git commit -m "feat: nova feature"  # commit
git push                    # envia pro remoto
git pull                    # baixa do remoto + merge
```

### Inspecionar histórico

```bash
git log                            # histórico completo
git log --oneline                  # 1 linha por commit
git log --graph --oneline --all    # visual com branches
git show <hash>                    # detalhes de um commit
git diff                           # mudanças não commitadas
git diff --staged                  # mudanças no staging
```

---

## 🌿 Branches: A Killer Feature

Branch é um **ponteiro pra um commit**. Criar branch é barato; é literalmente criar um arquivo com o hash de um commit.

```
main:    A ──── B ──── C
                        \
feature:                 D ──── E
```

### Comandos

```bash
git branch                       # lista branches
git branch nova-feature          # cria branch
git checkout nova-feature        # muda pra ela
git checkout -b nova-feature     # cria E muda (atalho)
git switch nova-feature          # alternativa moderna ao checkout

git branch -d branch-velha       # deleta (só se já mergeada)
git branch -D branch-forçada     # deleta forçado
```

### Merge

Une duas branches:

```bash
git checkout main
git merge nova-feature
```

Tipos de merge:

- **Fast-forward**: branch alvo só "anda pra frente" (sem commit de merge)
- **3-way merge**: cria commit novo unindo os históricos
- **Conflito**: Git não consegue decidir, você resolve manualmente

---

## 🔄 Rebase: A Alternativa ao Merge

Em vez de criar commit de merge, **rebase reescreve a história** colocando seus commits em cima da outra branch.

```
ANTES:
main:    A ──── B ──── C
                        \
feature:                 D ──── E

DEPOIS DE: git rebase main (estando em feature)
main:    A ──── B ──── C
                        \
feature:                 D' ──── E'  (commits novos com mesmos conteúdos)
```

### Quando usar cada um

✅ **Merge**: quando quer **preservar a história real** da branch (geralmente em main/master)

✅ **Rebase**: quando quer **histórico linear e limpo** (em branches de feature antes de mergear)

> ⚠️ **NUNCA dê rebase em branches públicas/compartilhadas.** Rebase reescreve hashes; se outros já usavam aqueles commits, você quebra tudo pra eles. Regra de ouro: rebase só em branches **suas que ninguém mais puxou**.

---

## 🔧 Operações Avançadas Úteis

### Stash: Guardar trabalho temporário

```bash
git stash                # guarda mudanças não commitadas
git stash pop            # recupera
git stash list           # lista stashes
git stash drop           # apaga o último
```

> 💡 Útil quando você precisa trocar de branch rapidinho mas tem coisa não commitada.

### Reverter um commit

```bash
# CRIA um novo commit que desfaz o anterior (seguro pra branches públicas)
git revert <hash>

# Volta o ponteiro pra um commit anterior (PERIGOSO em branches públicas)
git reset --hard <hash>
git reset --soft <hash>     # mantém mudanças no staging
```

### Cherry-pick: Trazer um commit específico

```bash
git cherry-pick <hash>
```

Útil pra trazer um fix de outra branch sem mergear ela inteira.

### Amend: Corrigir o último commit

```bash
git commit --amend -m "nova mensagem"
git commit --amend           # adiciona staged ao último commit
```

> ⚠️ Não use `--amend` em commits que já foram pushed.

### Reflog: Salva-vidas

```bash
git reflog
```

Mostra **TODAS** as ações que você fez no Git, mesmo as que "deletaram" coisas. Se você fez merda, o reflog provavelmente salva você.

---

## 🌐 Trabalhando com Remotos

```bash
git remote -v                          # lista remotos
git remote add origin URL              # adiciona remoto
git remote remove origin               # remove

git push origin main                   # envia branch
git push -u origin main                # primeira vez (set upstream)
git push --force                       # CUIDADO: sobrescreve remoto

git pull                               # fetch + merge
git fetch                              # só baixa, sem mergear
```

### Pull Request / Merge Request

Padrão de colaboração:

1. Você cria branch
2. Faz commits
3. Push da branch pro remoto
4. Abre PR/MR no GitHub/GitLab
5. Time revisa o código
6. Aprovado → merge no main

---

## 📝 Convenções de Commit

Não é regra do Git, mas o time consistente. **Conventional Commits** é o padrão mais popular:

```
<tipo>(<escopo opcional>): <mensagem curta>

[corpo opcional explicando mais detalhes]

[footer opcional - referências a issues, breaking changes]
```

| Tipo | Quando usar |
|---|---|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `docs` | Mudança apenas em documentação |
| `style` | Formatação, sem mudar lógica |
| `refactor` | Reescrita sem mudar comportamento |
| `test` | Adição/correção de testes |
| `chore` | Tarefas internas (build, deps) |
| `perf` | Melhoria de performance |
| `ci` | Mudanças em CI/CD |

Exemplos:
```
feat(auth): adicionar login com Google
fix(api): corrigir 500 quando email é nulo
docs: atualizar README com novos endpoints
refactor(user-service): extrair validação pra classe própria
```

> 💡 Mensagens de commit são **mensagens pra você do futuro** (e pro time). Vale o investimento de escrever bem.

---

## 📁 .gitignore

Arquivos que o Git deve ignorar:

```gitignore
# Dependências
node_modules/
target/
.venv/

# Build
dist/
build/
*.class

# IDE
.vscode/
.idea/
*.iml

# Sistema
.DS_Store
Thumbs.db

# Variáveis de ambiente (NUNCA commit!)
.env
.env.local

# Logs
*.log
```

> ⚠️ Se você commitou algo errado (tipo `.env` com senhas), **só remover do próximo commit não basta** — o histórico ainda tem. Precisa reescrever o histórico ou rotacionar as credenciais.

---

## 🚨 Erros Comuns e Soluções

### "Mexi na branch errada"
```bash
git stash                # guarda mudanças
git checkout branch-certa
git stash pop            # aplica aqui
```

### "Commitei na main, queria em feature"
```bash
git branch feature       # cria branch a partir do estado atual
git reset --hard HEAD~1  # volta main 1 commit
git checkout feature     # mudanças estão preservadas aqui
```

### "Quero desfazer o último commit mas manter as mudanças"
```bash
git reset --soft HEAD~1
```

### "Push rejeitado: remoto tem commits novos"
```bash
git pull --rebase        # ou git pull (vai criar merge)
git push
```

### "Conflito de merge"
1. Git marca conflitos no arquivo com `<<<<<<<`, `=======`, `>>>>>>>`
2. Edite manualmente, escolha o que fica
3. `git add arquivo`
4. `git commit` (ou `git rebase --continue` se estiver em rebase)

---

## ✅ Resumo da página

- Git guarda **snapshots completos** (com referências), não diffs
- 3 estados: **Working Directory** → **Staging** → **Repository**
- **Branches** são ponteiros pra commits — baratas, use bastante
- **Merge** preserva histórico; **rebase** linearisa (cuidado com branches públicas)
- **Stash** salva mudanças temporárias entre trocas de contexto
- **Reflog** salva você quando você acha que "perdeu" commits
- Use **Conventional Commits** pra mensagens consistentes
- Nunca commit `.env`, `node_modules`, builds — use `.gitignore`
- Ferramentas como `git revert`, `cherry-pick` e `--amend` são poderosas mas exigem cuidado

---

⬅️ [Anterior: Compiladores](./11-compiladores.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Estruturas de Dados](./13-estruturas-dados.md)
