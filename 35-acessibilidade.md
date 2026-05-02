# ♿ Acessibilidade Web (a11y)

> **15% da população mundial** tem alguma deficiência. Acessibilidade não é "feature extra" — é **direito**, em muitos países é **lei**, e é simplesmente **boa engenharia**. Sites acessíveis são melhores pra todo mundo.

---

## 🤔 O Que é Acessibilidade?

Garantir que pessoas com **diferentes capacidades** possam usar seu site/app:

- 👁️ **Visuais**: cegueira, baixa visão, daltonismo
- 👂 **Auditivas**: surdez, perda parcial
- 🤲 **Motoras**: dificuldade com mouse, tremor, paralisia
- 🧠 **Cognitivas**: dislexia, TDAH, Alzheimer
- ⏰ **Temporárias**: braço quebrado, óculos perdidos

> 💡 **a11y** é a abreviação comum de "accessibility" (a + 11 letras + y).

---

## 🎯 Por Que Você Deveria se Importar

### Razões éticas
- 1.3 bilhão de pessoas no mundo com alguma deficiência
- Acesso digital é direito básico

### Razões legais
- 🇧🇷 **Lei Brasileira de Inclusão (LBI)** — multa quem descumpre
- 🇺🇸 **ADA (Americans with Disabilities Act)** — processos por sites inacessíveis são comuns
- 🇪🇺 **European Accessibility Act**

### Razões de negócio
- Mercado de **R$ 700 bilhões** no Brasil só com PCDs
- SEO melhora (Google adora sites acessíveis)
- UX melhor pra todos (contexto: você no celular sob sol, idoso, ambiente barulhento)

---

## 📐 WCAG: O Padrão Mundial

**Web Content Accessibility Guidelines** — guia internacional, mantido pelo W3C.

### Níveis de conformidade

| Nível | Significa |
|---|---|
| **A** | Mínimo absoluto |
| **AA** | Padrão recomendado pela maioria das leis |
| **AAA** | Excelência (difícil de atingir 100%) |

### 4 Princípios (POUR)

1. **P**erceptível: usuário precisa **ver/ouvir** o conteúdo
2. **O**perável: precisa conseguir **interagir**
3. **U**nderstandable (compreensível): precisa **entender**
4. **R**obusto: funciona com **tecnologias assistivas** (leitores de tela)

---

## 👁️ Acessibilidade Visual

### Texto alternativo em imagens

```html
<!-- ❌ Sem alt -->
<img src="grafico.png" />

<!-- ❌ Alt vazio quando deveria ter -->
<img src="grafico.png" alt="" />

<!-- ✅ Alt descritivo -->
<img src="grafico.png" alt="Gráfico de barras mostrando vendas crescendo 30% em 2026" />

<!-- ✅ Alt vazio APENAS pra imagens decorativas -->
<img src="ornamento.png" alt="" />
```

> 🤔 **Dica**: imagine que você está descrevendo a imagem por telefone. Esse é o alt.

### Contraste de cores

WCAG AA exige:
- **4.5:1** pra texto normal
- **3:1** pra texto grande (18pt+)

```
❌ Texto cinza claro #BBBBBB em fundo branco → 1.9:1 (ruim)
✅ Texto cinza escuro #595959 em fundo branco → 7:1 (excelente)
```

**Tools:**
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- DevTools do Chrome (já mostra contraste em CSS)

### Não use só cor pra transmitir informação

```
❌ "Campos em vermelho são obrigatórios"
   (daltônicos não distinguem)

✅ "Campos em vermelho com asterisco (*) são obrigatórios"
   (cor + símbolo)
```

### Tamanho de fonte

- ✅ Mínimo **16px** pra texto de corpo
- ✅ Permita **zoom até 200%** sem quebrar layout
- ❌ Não use unidades absolutas (px) sem rem/em em conjunto

```css
/* Permite usuário ajustar tamanho base */
body { font-size: 1rem; }
h1 { font-size: 2rem; }
```

---

## ⌨️ Acessibilidade por Teclado

Muitos usuários **não usam mouse**: pessoas com dificuldade motora, usuários power, leitores de tela.

### Regras fundamentais

#### 1. Tudo que clica deve funcionar com teclado

```html
<!-- ❌ div clicável (teclado não consegue) -->
<div onclick="abrirModal()">Abrir</div>

<!-- ✅ button (teclado funciona automaticamente) -->
<button onclick="abrirModal()">Abrir</button>
```

#### 2. Foco visível

```css
/* ❌ NUNCA faça isso */
*:focus { outline: none; }

/* ✅ Customize, mas mantenha visível */
*:focus-visible {
  outline: 3px solid #4A90E2;
  outline-offset: 2px;
}
```

#### 3. Ordem lógica de tabulação

A ordem em que `Tab` percorre os elementos deve fazer sentido. **Geralmente segue a ordem do HTML** — então estruture HTML logicamente.

#### 4. Skip links

Pra usuários de teclado pularem a navegação repetitiva:

```html
<a href="#conteudo-principal" class="skip-link">
  Pular para o conteúdo principal
</a>
```

```css
.skip-link {
  position: absolute;
  top: -40px;        /* escondido */
}
.skip-link:focus {
  top: 0;            /* aparece quando focado */
}
```

---

## 📢 Leitores de Tela e ARIA

Pessoas cegas usam **screen readers** (NVDA, JAWS, VoiceOver) que **leem** o conteúdo.

### HTML semântico (primeiro!)

Use o elemento correto:

```html
<!-- ❌ Tudo div -->
<div class="header">
  <div class="nav">
    <div class="nav-item">Home</div>
  </div>
</div>

<!-- ✅ Semântico -->
<header>
  <nav>
    <ul>
      <li><a href="/">Home</a></li>
    </ul>
  </nav>
</header>
```

> 💡 **80% da acessibilidade** é só usar HTML semântico corretamente. ARIA é só pra casos onde HTML nativo não dá conta.

### ARIA: Atributos de acessibilidade

ARIA adiciona **semântica extra** quando HTML não basta.

```html
<!-- Botão de fechar com ícone -->
<button aria-label="Fechar modal">
  <svg>...</svg>  <!-- ícone X sem texto -->
</button>

<!-- Estado dinâmico -->
<button aria-expanded="false" aria-controls="menu">
  Menu
</button>
<ul id="menu" hidden>...</ul>

<!-- Live regions (avisar mudanças) -->
<div aria-live="polite">
  3 mensagens novas
</div>

<!-- Esconder de leitores de tela -->
<span aria-hidden="true">→</span>
```

### Regra de ouro do ARIA

> **"No ARIA é melhor que ARIA ruim"**

ARIA mal usado é **pior** que ausência. Antes de usar ARIA, sempre tente HTML semântico.

---

## 📝 Formulários Acessíveis

Formulários são onde mais se erra.

### Labels associados

```html
<!-- ❌ Sem label -->
<input type="text" placeholder="Nome" />

<!-- ❌ Label desconectado -->
<label>Nome</label>
<input type="text" />

<!-- ✅ Label associado por for/id -->
<label for="nome">Nome</label>
<input id="nome" type="text" />

<!-- ✅ Ou label envolvendo (também válido) -->
<label>
  Nome
  <input type="text" />
</label>
```

### Mensagens de erro

```html
<label for="email">Email</label>
<input
  id="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-erro"
/>
<span id="email-erro" role="alert">
  Email inválido
</span>
```

### Agrupamento

```html
<fieldset>
  <legend>Tipo de entrega</legend>
  <input type="radio" id="padrao" name="entrega" />
  <label for="padrao">Padrão</label>

  <input type="radio" id="expressa" name="entrega" />
  <label for="expressa">Expressa</label>
</fieldset>
```

---

## 🎬 Mídia Acessível

### Vídeos

- 📝 **Legendas** (closed captions) — surdos, ambientes silenciosos
- 🎙️ **Transcrição** — busca, processamento por IA
- 🎤 **Audiodescrição** pra cegos (descreve cenas)

### Áudios

- 📜 **Transcrição obrigatória**

### Animações e movimentos

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
    transition: none !important;
  }
}
```

Pessoas com vestibular issues (vertigem) sofrem com animações intensas.

---

## 🛠️ Ferramentas de Teste

### Automatizadas (pegam ~30% dos problemas)

- 🔧 **axe DevTools** (extensão Chrome/Firefox) — gold standard
- 🔧 **Lighthouse** (Chrome DevTools) — auditoria geral incluindo a11y
- 🔧 **WAVE** ([wave.webaim.org](https://wave.webaim.org/))
- 🔧 **Pa11y** (CLI/CI)

### Manual (essencial!)

#### Teste com teclado

Desconecte o mouse. Tente usar o site só com `Tab`, `Enter`, `Space`, `Esc`, setas. Funciona?

#### Teste com leitor de tela

- 🍎 **VoiceOver** (Mac/iOS): `Cmd + F5`
- 🪟 **NVDA** (Windows, gratuito)
- 🤖 **TalkBack** (Android)

> 💡 Os primeiros 5 minutos vão ser desconfortáveis. Persista — é assim que pessoas cegas usam a web todo dia.

#### Zoom 200%

```
Ctrl + + várias vezes
```

Layout quebra? Texto sai da tela? Resolva.

---

## 🚨 Anti-Patterns Clássicos

### 1. `<div>` clicável
```html
<!-- ❌ -->
<div onclick="...">Clique aqui</div>

<!-- ✅ -->
<button onclick="...">Clique aqui</button>
```

### 2. `outline: none` no foco
```css
/* ❌ */
button:focus { outline: none; }

/* ✅ */
button:focus-visible { outline: 3px solid blue; }
```

### 3. Cor sozinha como significado
```html
<!-- ❌ -->
<span style="color: red">Erro</span>

<!-- ✅ -->
<span style="color: red">⚠️ Erro: ...</span>
```

### 4. Imagem sem alt
```html
<!-- ❌ -->
<img src="logo.png" />

<!-- ✅ -->
<img src="logo.png" alt="Logo da empresa" />
```

### 5. Modal que não captura foco
Modal abre, mas `Tab` continua navegando atrás dele. Frustrante e inacessível.

### 6. `placeholder` como label
```html
<!-- ❌ -->
<input type="text" placeholder="Email" />

<!-- ✅ -->
<label for="email">Email</label>
<input id="email" type="email" />
```

### 7. Tempo limitado sem aviso
"Sua sessão expirou" — sem chance de recuperar. Cruel.

---

## 📋 Checklist Mínimo

Em todo PR, confira:

- [ ] HTML semântico (header, nav, main, footer, button, etc)
- [ ] Imagens com alt
- [ ] Contraste de cores adequado
- [ ] Funciona só com teclado
- [ ] Foco visível em todos os elementos interativos
- [ ] Labels associados em todos os inputs
- [ ] Mensagens de erro acessíveis
- [ ] Sem `outline: none`
- [ ] Vídeos com legenda
- [ ] Testou com leitor de tela ao menos uma vez
- [ ] Lighthouse score >= 90 em a11y

---

## 🌟 Frameworks e Bibliotecas

A maioria já vem com a11y razoável:

- ⚛️ **React**: cuide do `aria-*` nos seus componentes; bibliotecas como **Radix UI**, **React Aria** já têm acessibilidade pronta
- 🟢 **Vue**: similar; use **Headless UI** ou **Vuetify** (acessível por padrão)
- 🅰️ **Angular Material**: acessível por padrão
- 🍃 **Tailwind**: você é responsável pela semântica

> 💡 **Headless UI libraries** (Radix, React Aria, shadcn/ui) são revolucionárias — dão acessibilidade pronta, você só estiliza.

---

## ✅ Resumo da página

- a11y é **direito**, **lei** em muitos países, **boa engenharia** sempre
- 15% das pessoas têm alguma deficiência — design pra todos
- Siga **WCAG 2.1 nível AA** (padrão mundial)
- 4 princípios: **Perceptível, Operável, Compreensível, Robusto**
- 80% é **HTML semântico** — só use ARIA quando necessário
- Sempre: alt em imagens, labels em inputs, contraste suficiente, foco visível
- **Teste com teclado** e **leitor de tela** — automação só pega 30%
- "No ARIA é melhor que ARIA ruim"
- Use **Headless UI libraries** pra acessibilidade pronta

---

⬅️ [Anterior: Mobile e PWA](./34-mobile-pwa.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Internacionalização](./36-i18n.md)
