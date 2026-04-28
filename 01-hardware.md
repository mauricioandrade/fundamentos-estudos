# 🖨️ Hardware e Periféricos

> A casa física onde o software mora. Antes de qualquer linha de código rodar, ela precisa de eletricidade, fios e silício.

---

## 🏠 A Placa-Mãe: A Espinha Dorsal

A **placa-mãe (motherboard)** é onde tudo se conecta. Pense nela como a planta baixa de uma casa: ela define onde cada componente vai morar e como eles conversam entre si através de "barramentos" (caminhos elétricos).

> 💡 **Em palavras simples**: a placa-mãe não "faz" nada sozinha — ela é a tábua que segura e conecta CPU, RAM, GPU e armazenamento.

---

## 🧠 Componentes Essenciais

### 🔥 CPU (Processador)

O **cérebro** do computador. Executa instruções matemáticas e lógicas em velocidades absurdas.

- **Velocidade**: medida em **GHz** (gigahertz). 3 GHz = 3 bilhões de ciclos por segundo
- **Núcleos**: CPUs modernas têm vários núcleos (cores), cada um capaz de executar tarefas em paralelo
- **Threads**: cada núcleo pode lidar com 1 ou 2 threads simultâneas (Hyper-Threading da Intel, SMT da AMD)

### 💾 RAM (Memória Principal)

A **bancada de trabalho**. É onde os programas ficam **enquanto estão rodando**.

- ⚡ **Ultrarrápida** (acesso em nanosegundos)
- 🪫 **Volátil**: apaga tudo ao desligar o PC
- 📏 Capacidade em **GB**, frequência em **MHz**, geração **DDR4/DDR5**

> 🤔 **Por que importa?** Se você tem pouca RAM e abre muitos programas, o sistema empurra dados pro disco (swap) e tudo trava. Por isso 16 GB virou o mínimo recomendado pra dev.

### 💿 Armazenamento Secundário

O **cofre**. Aqui ficam o sistema operacional, seus arquivos e seus jogos — mesmo com o PC desligado.

| Tipo | Tecnologia | Velocidade | Uso típico |
|---|---|---|---|
| **HDD** | Disco magnético giratório | 🐢 Lento (~150 MB/s) | Arquivamento, backup |
| **SSD SATA** | Memória flash | 🚀 Rápido (~550 MB/s) | Sistemas modernos |
| **SSD NVMe** | Flash via PCIe | ⚡ Muito rápido (3.000-7.000 MB/s) | Gaming, dev, edição |

### 🎮 GPU (Placa de Vídeo)

Processador especializado em **paralelismo massivo**. Renderiza gráficos, calcula iluminação, e hoje em dia também processa IA.

- Tem sua própria memória chamada **VRAM** (separada da RAM)
- Enquanto a CPU tem ~16 núcleos potentes, a GPU tem milhares de núcleos pequenos
- Por isso é boa em fazer "muita coisa simples ao mesmo tempo" (renderizar pixels, treinar redes neurais)

### ⚡ Fonte de Alimentação

Converte a energia da tomada (110V/220V AC) em tensões precisas (12V, 5V, 3.3V DC) para os componentes. Uma fonte ruim pode causar travamentos misteriosos.

---

## 🖱️ Periféricos: A Ponte com o Humano

### ⌨️ Teclados

| Tipo | Como funciona | Perfil |
|---|---|---|
| **Membrana** | Borrachinha sob a tecla | Barato, escritório |
| **Mecânico** | Switch individual por tecla | Durável, preciso, gamer/dev |
| **Óptico** | Sensor de luz | Acionamento ultrarrápido |

### 🖱️ Mouses

Dois números importantes:

- **DPI** (Dots Per Inch): sensibilidade. Mais DPI = ponteiro anda mais rápido com pouco movimento
- **Polling Rate**: frequência (em Hz) com que o mouse reporta posição pra CPU. 1000 Hz = atualização a cada 1 ms

### 🖥️ Monitores

Três coisas definem qualidade:

- **Resolução**: 1080p, 1440p, 4K — quantos pixels existem
- **Taxa de atualização**: 60 Hz, 144 Hz, 240 Hz — quantas vezes a tela atualiza por segundo
- **Painel**:
  - **IPS** → cores precisas (design, fotografia)
  - **VA** → contraste alto (filmes, escuro)
  - **OLED** → preto absoluto, mas risco de queima
  - **TN** → barato, rápido, cores ruins

### 🎵 Áudio Dedicado

Placas-mãe vêm com chip de som integrado, que costuma ter ruído elétrico. Soluções:

- **DAC** (Digital-to-Analog Converter): converte o sinal digital com mais qualidade
- **Interface de áudio**: pra plugar microfones profissionais (XLR) ou fones de alta impedância
- Útil para produção musical, podcast, streaming sério

### 📹 Placa de Captura

Captura vídeo de outra fonte (console, outro PC, câmera) sem sobrecarregar a CPU principal. Padrão pra streamers que usam setup de 2 PCs.

### 🌐 Placa de Rede (NIC)

Gerencia a conexão. Cabo Ethernet é mais estável; Wi-Fi 6/7 oferece velocidade alta sem fio. Em servidores, NICs dedicadas com 10 Gbps ou mais são comuns.

---

## 🏛️ Bônus: Von Neumann vs Harvard

Quase todo computador moderno segue o **modelo de Von Neumann**: uma única memória armazena **instruções E dados juntos**.

- ✅ Simples, barato, flexível
- ❌ Tem um gargalo: ler instrução e ler dado disputam o mesmo barramento

A **arquitetura de Harvard** separa os dois — é mais rápida, mas mais cara. Comum em microcontroladores (Arduino, DSPs).

---

## ✅ Resumo da página

- O **hardware** é a base física. Sem ele, software é só ideia.
- **CPU** processa, **RAM** guarda processos ativos, **SSD** guarda permanente
- **GPU** é especialista em paralelismo (gráficos, IA)
- **Periféricos** definem como você interage com a máquina
- A maioria dos PCs segue **Von Neumann** (memória unificada)

---

🔙 [Voltar ao índice](./README.md) | ➡️ [Próximo: Processador (CPU)](./02-processador.md)
