# 🌉 Sistema Operacional: O Mediador

> O SO é o gerente do prédio. Ele faz a ponte entre o hardware bruto e os seus aplicativos, e impede que um programa estrague o sistema todo.

---

## 🤝 O que o SO faz por você

Toda vez que você:

- Salva um arquivo → o SO escreve no SSD
- Abre um socket de rede → o SO conversa com a placa de rede
- Cria uma thread → o SO escalona ela na CPU
- Aloca memória → o SO te dá um pedaço da RAM

Sem SO, você teria que escrever **driver de cada hardware** dentro de cada app. Imagine seu app Node ter que saber falar com cada modelo de SSD existente. Caos.

> 💡 **Em palavras simples**: o SO é uma **camada de abstração**. Ele esconde a complexidade do hardware atrás de APIs padronizadas (chamadas **syscalls**).

---

## 🔐 Modos de Execução: User vs Kernel

A CPU tem **dois níveis de privilégio** (alguns processadores têm mais, mas pra prática são 2):

| Modo | Quem roda | Pode |
|---|---|---|
| 👤 **User Mode** | Apps comuns (Chrome, VS Code, jogos) | Só mexer na própria memória |
| 👑 **Kernel Mode** | Núcleo do SO, drivers | TUDO (acessar hardware, qualquer memória) |

> 🤔 **Por que essa separação existe?** Pra segurança. Se um app travasse no kernel mode, ele poderia formatar seu HD ou corromper outros programas. Em user mode, no máximo o próprio app cai.

---

## 📞 Syscalls: Como o App "pede coisas" pro SO

Quando seu programa precisa fazer algo privilegiado (ler arquivo, abrir socket, criar processo), ele faz uma **syscall** (chamada de sistema):

```
1. App em user mode quer abrir arquivo
       ↓
2. Chama a syscall (ex: open() em Linux)
       ↓
3. CPU salta pra kernel mode
       ↓
4. Kernel verifica permissões, lê do disco
       ↓
5. Devolve resultado pro app
       ↓
6. CPU volta pra user mode
```

> 💡 No Linux, `strace ./meu-programa` mostra **todas** as syscalls que um programa faz. Vale a pena experimentar — você vê quão "tagarela" um programa simples pode ser.

---

## 🪟 Windows (Microsoft)

**Filosofia**: domínio do mercado corporativo + retrocompatibilidade extrema (programas de 20 anos atrás ainda rodam).

### Pontos fortes

- 🎮 **Rei dos jogos**: a API **DirectX** é o padrão da indústria
- 💼 **Ecossistema corporativo**: Office, Active Directory, dependências de softwares antigos específicos
- 🛍️ Suporte massivo de fabricantes (drivers pra tudo)

### Para devs

- **WSL** (Windows Subsystem for Linux): roda um Kernel Linux **real** dentro do Windows, sem VM pesada. Mudou o jogo pra dev no Windows
- **PowerShell**: alternativa ao CMD, com sintaxe orientada a objetos
- ⚠️ Sintaxe de path com backslash (`C:\`) e quebras de linha CRLF causam dor com ferramentas Unix

---

## 🍏 macOS (Apple)

**Filosofia**: "Walled Garden" — Apple controla hardware E software, garantindo integração extrema.

### Pontos fortes

- 🧱 **Baseado em UNIX**: o terminal é amigável pra quem trabalha com servidores (porque servidores quase sempre são Linux/Unix)
- 🔋 **Apple Silicon (M1/M2/M3/M4)**: chips ARM com performance alta e bateria absurda
- 🎨 Padrão da indústria criativa (design, vídeo, áudio)

### Para devs

- Ferramentas Unix nativas (`grep`, `sed`, `curl`, `ssh`)
- **Homebrew**: gerenciador de pacotes (similar ao apt/pacman)
- ⚠️ Hardware fechado (não dá pra montar um Mac), preço alto

---

## 🐧 Linux (Código Aberto)

**Filosofia**: liberdade, modularidade e ausência de licença comercial.

### Importante: "Linux" só é o Kernel

Tecnicamente, **Linux é só o núcleo** que conversa com o hardware. O resto do sistema (terminal, gerenciador de pacotes, interface gráfica) é construído ao redor dele.

Combinações diferentes desses componentes geram **distribuições** (distros) diferentes.

### Onde Linux domina

- 🌐 **Servidores**: a maior parte da internet roda em Linux. Por quê?
  - Sem custo de licença
  - Leve (sem interface gráfica = menos recursos)
  - Configurável até o último byte
  - Estável (uptime de anos)
- ☁️ **Containers e nuvem**: Docker, Kubernetes, AWS, GCP — tudo rodando Linux
- 📱 **Android**: usa kernel Linux por baixo

### Distros principais

| Distro | Característica | Público |
|---|---|---|
| **Ubuntu** | Amigável, releases fixos a cada 6 meses | Iniciantes, dev, servidor |
| **Debian** | Estabilidade extrema, atualizações conservadoras | Servidores |
| **Fedora** | Tecnologia moderna, base do RHEL corporativo | Devs |
| **Arch** | DIY, minimalista, rolling release | Usuários avançados |
| **Mint** | Ubuntu mais polido, interface tipo Windows | Migrando de Windows |

> 💡 **Quer aprender Linux fundo?** Veja [Linux e Arch (deep dive)](./06-linux-arch.md) com filosofia, Pacman e AUR.

---

## 🖥️ Máquinas Virtuais: SO dentro de SO

Você pode rodar um Linux **dentro** do seu Windows (ou vice-versa) usando uma **VM**.

### Como funciona

A VM aproveita os modos de privilégio da CPU pra **simular um hardware completo**. O sistema convidado (guest) acha que está rodando em hardware real.

### Por que usar?

- 🛡️ **Isolamento**: testar software suspeito sem risco
- 🧪 **Múltiplos ambientes**: rodar Windows XP num PC moderno
- 🔄 **Snapshots**: salvar estado e voltar quando quiser
- ☁️ **Servidores virtuais**: a base da computação em nuvem

### VMs vs Containers (Docker)

- **VM**: simula hardware completo + SO completo. Pesado.
- **Container**: compartilha o kernel do host, isola só o app. Leve.

> 💡 É por isso que Docker virou tão popular — você consegue isolamento parecido com VM, mas com performance quase nativa.

---

## ✅ Resumo da página

- O **SO é mediador** entre hardware e apps
- CPU tem **modos**: user (limitado, seguro) e kernel (privilégio total)
- Apps pedem coisas ao SO via **syscalls**
- **Windows** domina jogos e corporativo; **macOS** é UNIX-based com hardware fechado; **Linux** domina servidor e nuvem
- Linux é só o **kernel** — distros são combinações de software ao redor
- **VMs** isolam SOs inteiros; **containers** são mais leves

---

⬅️ [Anterior: Memória](./03-memoria.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Linguagens](./05-linguagens.md)
