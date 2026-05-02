# 📱 Mobile e PWA

> Smartphones são a forma como a maioria do mundo acessa a internet. Pra dev backend, isso significa: APIs precisam atender mobile bem. Pra dev frontend, escolher entre **nativo, híbrido ou PWA** é decisão estratégica importante.

---

## 🎯 As 3 Grandes Estratégias

| Abordagem | Como | Exemplos |
|---|---|---|
| 📱 **Nativo** | Código específico por plataforma | Swift (iOS), Kotlin (Android) |
| 🔀 **Híbrido / Cross-platform** | Um código pra ambos | React Native, Flutter |
| 🌐 **PWA (Progressive Web App)** | Site web que vira "app" | Twitter, Spotify Web |

---

## 📱 Mobile Nativo

Você usa as ferramentas oficiais de cada plataforma.

### iOS

- **Linguagem**: Swift (moderna) ou Objective-C (legado)
- **IDE**: Xcode (só roda em Mac)
- **UI**: SwiftUI (moderno) ou UIKit (clássico)

### Android

- **Linguagem**: Kotlin (recomendado) ou Java (legado)
- **IDE**: Android Studio (qualquer SO)
- **UI**: Jetpack Compose (moderno) ou XML Views (clássico)

### Vantagens
- 🚀 **Performance máxima** (sem camadas extras)
- 🎨 **UX nativo** (componentes da plataforma)
- 🔐 **Acesso completo** a APIs do dispositivo
- 📦 **Suporte oficial** da Apple/Google

### Desvantagens
- 💰 **2 codebases** (= 2x esforço)
- 👨‍💻 **2 perfis de dev** (Swift + Kotlin)
- ⏰ **Cada feature em dobro**

> 💡 **Quando usar nativo**: jogos, apps que dependem muito de hardware (câmera AR, sensores), apps grandes onde performance é crítica.

---

## 🔀 Cross-Platform: React Native vs Flutter

A solução pra evitar 2 codebases.

### React Native

Criado pelo Facebook. Você escreve **JavaScript/TypeScript** com componentes React; ele renderiza com componentes **nativos** por baixo.

```jsx
import { View, Text, Button } from 'react-native';

export default function App() {
  return (
    <View>
      <Text>Olá Mundo!</Text>
      <Button title="Clique" onPress={() => alert('Oi')} />
    </View>
  );
}
```

✅ Time React/web aproveitado
✅ Componentes nativos (UX correto)
✅ Hot reload rapidíssimo
✅ Maturidade alta (8+ anos)
❌ Performance pode degradar em apps complexos
❌ Bridge JS-nativo limita certas tarefas

**Apps famosos**: Facebook, Instagram, Discord, Tesla, Coinbase.

### Flutter

Criado pelo Google. Você escreve **Dart**; ele desenha tudo do zero (não usa componentes nativos).

```dart
import 'package:flutter/material.dart';

class MeuApp extends StatelessWidget {
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Olá Mundo!'),
        ElevatedButton(
          onPressed: () => print('Oi'),
          child: Text('Clique'),
        ),
      ],
    );
  }
}
```

✅ Performance excelente (compila pra nativo)
✅ Mesma UI em qualquer plataforma (iOS, Android, web, desktop)
✅ Animações suavíssimas
❌ Aprender Dart (linguagem nova pra maioria)
❌ Componentes não-nativos (parecem com nativos, mas são desenhados)
❌ Apps maiores (engine Flutter embutida)

**Apps famosos**: Google Pay, BMW, Alibaba, eBay Motors.

### React Native vs Flutter: Como escolher?

| Critério | React Native | Flutter |
|---|---|---|
| **Time vem de** | React/web | Backend, mobile, novo |
| **Performance** | Boa | Excelente |
| **UI** | Componentes nativos | Desenhado pelo Flutter |
| **Curva de aprendizado** | Menor (se já sabe React) | Maior (Dart é nova) |
| **Tooling** | Expo facilita muito | Flutter SDK direto |
| **Escolha por padrão** | Time web migrando | Projeto novo, sem amarras |

### Outras opções

- 🔄 **.NET MAUI** (Microsoft): C# multiplataforma
- 🟢 **NativeScript**: JS com componentes nativos
- 🔷 **Ionic**: HTML/CSS/JS dentro de WebView (mais simples, menos performance)
- ⚡ **Kotlin Multiplatform**: compartilha lógica entre iOS/Android, UI separada

---

## 🌐 PWA (Progressive Web App)

**Site web que se comporta como app**. Sem app store, sem download — abre no browser, mas pode ser "instalado" e funciona offline.

### O que faz um site virar PWA

3 ingredientes mínimos:

#### 1. HTTPS
Obrigatório.

#### 2. Manifest

Arquivo JSON que diz como o "app" se comporta:

```json
{
  "name": "Meu App",
  "short_name": "App",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

#### 3. Service Worker

Script JS que roda em background. Habilita:
- 🔌 **Funcionar offline** (cache de recursos)
- 📨 **Push notifications**
- 🔄 **Background sync**

```javascript
// service-worker.js
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then(cache => {
      return cache.addAll(['/', '/styles.css', '/app.js']);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(response => {
      return response || fetch(event.request);
    })
  );
});
```

### Vantagens

- 🚀 **Sem app stores** (sem 30% de comissão Apple/Google)
- 🔄 **Atualização instantânea** (igual site)
- 💾 **Tamanho menor** que apps nativos
- 🔗 **Linkável** (URLs diretas pra qualquer tela)
- 🌐 **Multi-plataforma** automaticamente

### Desvantagens

- 🍎 **iOS limita PWA** (push notifications limitadas, sem instalação fluida)
- 🔌 **APIs restritas** (Bluetooth, NFC, etc — algumas só nativo)
- 📱 **UX não 100% app** (dá pra perceber)

### Apps que são PWAs famosos

- Twitter Lite
- Spotify Web
- Pinterest
- Starbucks
- Uber

> 💡 **Quando PWA é a escolha certa**: produtos com grande parcela de uso web, mercados emergentes (instalação de app é caro em planos limitados), ferramentas internas corporativas.

---

## 🎯 Como Escolher

```
Precisa de hardware específico (AR, sensores, BLE complexo)?
├── SIM → Nativo
└── NÃO ↓

Time já é forte em React?
├── SIM → React Native ou PWA
└── NÃO ↓

Precisa app store / monetização via store?
├── SIM → Cross-platform (RN ou Flutter)
└── NÃO → PWA

Performance crítica + UX nativa importa muito?
└── Flutter ou Nativo
```

---

## 📡 APIs Pensando em Mobile

Apps mobile têm restrições que apps web não têm. Sua API precisa respeitar:

### 1. Bandwidth limitada

```
❌ GET /usuarios → retorna 5MB de JSON
✅ GET /usuarios?fields=id,nome → retorna só o necessário
```

### 2. Latência alta (4G, 3G)

- Use **gzip/brotli** no response
- **HTTP/2** ou **HTTP/3** ajudam
- **Pagination cursor-based** (mais eficiente que offset)

### 3. Conexões instáveis

- **Retry com backoff** no cliente
- **Idempotência** em writes
- **Optimistic updates**: UI atualiza primeiro, sincroniza depois

### 4. Cache offline

```
App busca → tem no cache? → mostra instantâneo
                           → faz request em background
                           → atualiza UI se mudou
```

### 5. Deep linking

```
meuapp://produto/123 → abre o app na tela do produto
https://meuapp.com/produto/123 → abre o app SE instalado, ou web
```

### 6. Push notifications

- **iOS**: APNs (Apple Push Notification service)
- **Android**: FCM (Firebase Cloud Messaging)
- **PWA web**: Web Push API

---

## 🔄 Sincronização Offline-First

Um padrão crítico pra mobile real.

### Estratégia

```
1. App faz mudança → salva LOCAL primeiro (SQLite, IndexedDB)
2. Mostra UI atualizada imediatamente
3. Em background, sincroniza com servidor
4. Em caso de conflito, resolve (last-write-wins, merge, prompt)
```

### Bibliotecas

- **WatermelonDB** (React Native)
- **Drift / Floor** (Flutter)
- **Realm** (multi-platform)
- **PowerSync** (sync engine)
- **Replicache**

> 🤔 **Por que offline-first é difícil?** Resolução de conflitos. Dois dispositivos editaram o mesmo registro offline — qual vence? Aí entra a complexidade real.

---

## 🛠️ Stack Mobile Moderna

### React Native

```
Expo (framework)
├── React Navigation (rotas)
├── Tanstack Query (dados/cache)
├── Zustand ou Redux (estado)
├── NativeWind (Tailwind pra RN)
└── Expo Router (file-based routing)
```

### Flutter

```
Flutter SDK
├── go_router (navegação)
├── Riverpod ou Bloc (estado)
├── dio (HTTP client)
└── flutter_hooks (composição)
```

---

## 📊 Métricas Importantes em Mobile

### Performance

- ⚡ **App startup time**: < 2s ideal
- 🔋 **Battery usage**: monitorado pelos OSes
- 💾 **App size**: cada MB extra reduz instalações
- 🌡️ **Frame rate**: 60fps mínimo, 120fps ideal em telas modernas

### Engagement

- **DAU/MAU** (daily/monthly active users)
- **Session length**
- **Retention** (D1, D7, D30)
- **Crash rate** (< 0.5% considerado bom)

### Tools de monitoramento

- **Firebase Crashlytics** (crashes)
- **Sentry** (erros + performance)
- **Datadog RUM** (Real User Monitoring)
- **Amplitude / Mixpanel** (analytics de produto)

---

## 🎨 Princípios de UX Mobile

- 👆 **Touch targets >= 44x44pt** (Apple) ou 48x48dp (Android)
- 📱 **Safe areas** (não sobrepor notch, barra de gesto)
- ⏳ **Feedback imediato** (skeleton screens, animações)
- 🔄 **Pull to refresh**
- ↩️ **Gestos de voltar** (swipe right no iOS)
- 🌗 **Dark mode** (respeitar configuração do sistema)
- ♿ **Acessibilidade** (VoiceOver, TalkBack)

---

## ✅ Resumo da página

- 3 abordagens: **nativo** (Swift/Kotlin), **cross-platform** (React Native/Flutter), **PWA** (web)
- **Nativo**: máxima performance, 2x esforço
- **React Native**: aproveita time React, componentes nativos
- **Flutter**: performance excelente, UI consistente, Dart
- **PWA**: sem app store, atualização instantânea, mas iOS limita
- **APIs pra mobile** precisam considerar bandwidth, latência, offline
- **Offline-first** é essencial mas difícil (resolução de conflitos)
- **Push notifications**: APNs (iOS), FCM (Android), Web Push (PWA)
- Métricas: startup time, battery, app size, crash rate, retention

---

⬅️ [Anterior: MLOps](./33-mlops.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Acessibilidade](./35-acessibilidade.md)
