# 🌐 Redes: Como os Computadores se Falam

> Toda vez que você abre um site, manda uma requisição ou faz deploy, dezenas de protocolos estão trabalhando. Entender o básico explica MUITA coisa do dia a dia de dev: por que CORS dá erro, por que sua API tá lenta, por que o DNS "demora pra propagar".

---

## 📮 Analogia inicial: Rede = Sistema Postal

Imagine que você quer mandar uma carta pra alguém em outro estado:

1. Você escreve a carta (**dado**)
2. Coloca o endereço do destinatário no envelope (**IP**)
3. Anota o número do apartamento (**porta**)
4. Coloca o remetente (**seu IP + porta**)
5. Os correios fazem o roteamento, passando por várias agências (**roteadores**)
6. A carta chega ao destinatário

A internet funciona basicamente igual — só que em vez de carteiros, são fios e sinais elétricos/ópticos. E em vez de dias, leva milissegundos.

---

## 🧱 O Modelo em Camadas

Redes são complicadas. Pra organizar, foi criado um modelo em camadas onde **cada camada cuida de uma coisa específica** e "entrega" pra próxima.

Existem dois modelos famosos: **OSI** (7 camadas) e **TCP/IP** (4 camadas). Na prática, basta entender a essência:

```
┌─────────────────────────────────┐
│ APLICAÇÃO                       │ ← HTTP, DNS, SSH, FTP
│ (o que você usa)                │
├─────────────────────────────────┤
│ TRANSPORTE                      │ ← TCP, UDP
│ (entrega confiável ou não)      │
├─────────────────────────────────┤
│ REDE                            │ ← IP
│ (roteamento entre máquinas)     │
├─────────────────────────────────┤
│ ENLACE / FÍSICA                 │ ← Ethernet, Wi-Fi, fibra
│ (sinais elétricos, ondas)       │
└─────────────────────────────────┘
```

> 💡 **Em palavras simples**: cada camada conversa só com a vizinha. Seu HTTP vira TCP, que vira IP, que vira sinal elétrico no fio. Do outro lado, o processo é inverso. Cada camada **encapsula** a anterior dentro de um envelope.

---

## 🏷️ Endereços IP

### IPv4 vs IPv6

**IPv4**: 4 números de 0-255 separados por ponto (ex: `192.168.1.1`)
- 4,3 bilhões de endereços possíveis
- Esgotou faz tempo

**IPv6**: 8 grupos de 4 hexadecimais (ex: `2001:0db8:85a3::8a2e:0370:7334`)
- Quantidade absurda (3,4 × 10³⁸)
- Adoção lenta, ainda em transição

### IP Público vs IP Privado

Sua rede doméstica usa **IPs privados** — não roteáveis na internet pública:

| Faixa | Uso típico |
|---|---|
| `10.0.0.0/8` | Empresas grandes |
| `172.16.0.0/12` | Empresas médias |
| `192.168.0.0/16` | Casa, escritórios pequenos |
| `127.0.0.0/8` | Localhost (sua própria máquina) |

### 🔄 NAT: Como sua casa inteira tem 1 IP só

Você tem **1 IP público** (o que aparece em [ifconfig.me](https://ifconfig.me)), mas vários dispositivos conectados. Como funciona?

O **roteador** faz **NAT** (Network Address Translation):

1. Seu PC manda pacote pra `google.com` usando IP privado `192.168.1.5`
2. Roteador troca o IP de origem pelo IP público antes de mandar pra internet
3. Resposta volta pro IP público; roteador lembra de quem foi e devolve pro PC certo

> 🤔 **Por que isso importa?** É por isso que você não consegue receber conexões de fora sem **port forwarding**. Servidores hospedados em casa atrás de NAT não são "encontráveis" diretamente da internet — daí a necessidade de hospedagem (HostGator, Render, AWS).

---

## 🔢 Portas

Um IP identifica uma **máquina**. Uma porta identifica um **serviço** dentro dela.

> 💡 **Analogia**: IP é o endereço do prédio, porta é o número do apartamento.

### Portas que todo dev precisa saber

| Porta | Serviço |
|---|---|
| 22 | SSH |
| 25 / 587 | SMTP (envio de email) |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3000 | Node.js (Next.js padrão) |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | HTTP alternativo (Spring Boot dev) |
| 27017 | MongoDB |

Faixas:
- **0-1023**: well-known (precisa root pra usar no Linux)
- **1024-49151**: registradas
- **49152-65535**: efêmeras (cliente usa em conexões temporárias)

---

## 🚚 TCP vs UDP

Os dois protocolos de transporte mais importantes.

### TCP: Entrega Garantida

- Estabelece **conexão** primeiro (handshake de 3 passos)
- Garante que pacotes chegam **na ordem**
- Reenvia se algum se perde
- Mais lento, mais confiável

> 💡 **Analogia**: carta registrada com aviso de recebimento.

**Usado em**: HTTP, HTTPS, SSH, FTP, email, bancos de dados

### UDP: Entrega Sem Garantia

- Sem conexão — manda e esquece
- Pacotes podem chegar fora de ordem ou se perder
- Muito mais rápido

> 💡 **Analogia**: cartão postal.

**Usado em**: DNS, streaming (YouTube, Netflix), jogos online, VoIP

> 🤔 **Por que streaming usa UDP?** Em vídeo ao vivo, é melhor pular um frame perdido do que esperar pra receber ele. Latência importa mais que integridade.

---

## 📞 DNS: A Lista Telefônica da Internet

Quando você digita `google.com`, seu navegador não sabe qual o IP. Ele pergunta pro **DNS**:

```
1. Você: "DNS, qual o IP de google.com?"
2. DNS: "É 142.250.79.110"
3. Navegador conecta nesse IP
```

### Tipos de registros DNS

| Tipo | O que faz |
|---|---|
| `A` | Aponta domínio → IPv4 |
| `AAAA` | Aponta domínio → IPv6 |
| `CNAME` | Apelido (alias) pra outro domínio |
| `MX` | Servidor de email do domínio |
| `TXT` | Texto livre (verificações, SPF) |
| `NS` | Quem é o servidor DNS autoritativo |

> 🤔 **"Por que minha mudança de DNS demora pra aparecer?"** Servidores DNS guardam respostas em cache pelo tempo definido no **TTL** (de 5 min a 24h). Você muda no painel do domínio, mas a internet só atualiza depois que o cache expira. Isso é a famosa "propagação" de DNS.

### Tools úteis

```bash
# Ver o IP de um domínio
nslookup google.com
dig google.com

# Ver todos os tipos de registro
dig google.com ANY

# Ver registros MX (servidores de email)
dig google.com MX
```

---

## 🌐 HTTP e HTTPS

O protocolo da web. Cliente manda **request**, servidor manda **response**.

### Anatomia de uma requisição

**Request** (cliente → servidor):
```http
GET /api/users HTTP/1.1
Host: api.exemplo.com
Authorization: Bearer abc123
Accept: application/json
```

**Response** (servidor → cliente):
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 56

{"users": [{"id": 1, "name": "Mauricio"}]}
```

### Métodos (verbs)

| Método | Quando usar |
|---|---|
| `GET` | Buscar dados (sem efeito colateral) |
| `POST` | Criar recurso |
| `PUT` | Atualizar (substituindo todo o recurso) |
| `PATCH` | Atualizar parcial |
| `DELETE` | Remover |
| `OPTIONS` | Preflight CORS, descobrir métodos disponíveis |

### Status Codes

| Faixa | Significado | Exemplos |
|---|---|---|
| **1xx** | Informativo | 100 Continue |
| **2xx** | Sucesso | 200 OK, 201 Created, 204 No Content |
| **3xx** | Redirecionamento | 301 Moved Permanently, 304 Not Modified |
| **4xx** | Erro do cliente | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| **5xx** | Erro do servidor | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

### 🔒 HTTPS: HTTP com criptografia

HTTPS = HTTP + **TLS** (antigo SSL).

O que o TLS resolve:

- 🔐 **Confidencialidade**: ninguém entre você e o servidor consegue ler
- ✅ **Integridade**: ninguém altera a mensagem no caminho
- 🪪 **Autenticidade**: você confirma que está falando com o servidor certo (certificado)

> 🤔 **Como confiamos nos certificados?** O navegador vem com lista de **Autoridades Certificadoras (CAs)** confiáveis (Let's Encrypt, DigiCert, etc). Quando o site apresenta um certificado, o navegador verifica se foi assinado por alguma dessas CAs.

### Versões do HTTP

- **HTTP/1.1**: 1 requisição por vez por conexão (lento, mas universal)
- **HTTP/2**: multiplexação — várias requisições em paralelo na mesma conexão
- **HTTP/3**: usa **QUIC** (sobre UDP), ainda mais rápido — adoção crescendo

---

## 🚧 CORS: O Pesadelo do Frontend

**CORS** (Cross-Origin Resource Sharing) é um mecanismo de segurança **do navegador**, não do servidor.

### O problema

Por padrão, JavaScript rodando em `meusite.com` **não pode** fazer fetch em `outraapi.com`. Isso evita que sites maliciosos acessem dados em outros domínios usando seus cookies.

### A solução

O servidor (`outraapi.com`) precisa **autorizar explicitamente** o domínio com headers:

```http
Access-Control-Allow-Origin: https://meusite.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Content-Type, Authorization
```

> 🤔 **Erro clássico**: você bate na sua API e o navegador devolve "blocked by CORS policy". A API funcionou! O problema é que ela não devolveu o header dizendo que aceita seu domínio. Pra confirmar isso, faça a mesma request com `curl` — vai funcionar. CORS só existe no navegador.

> 💡 No Spring Boot: anote `@CrossOrigin` no controller ou configure um `WebMvcConfigurer` global.

---

## 🛠️ Ferramentas Essenciais

### Diagnóstico

```bash
# Conectividade básica
ping google.com              # Verifica se chega no host
ping -c 4 google.com         # 4 pacotes

# Caminho até o destino (cada salto = 1 roteador)
traceroute google.com        # Linux/Mac
tracert google.com           # Windows

# Resolução DNS
nslookup google.com
dig google.com
host google.com

# Conexões e portas abertas na sua máquina
netstat -tulpn               # Linux antigo
ss -tulpn                    # Linux moderno
lsof -i :8080                # Quem tá usando a porta 8080?
```

### Testes HTTP com `curl`

```bash
# GET simples
curl https://api.github.com/users/mauricio

# Ver headers da resposta
curl -I https://google.com

# Modo verboso (ver tudo, incluindo TLS)
curl -v https://google.com

# POST com JSON
curl -X POST https://api.exemplo.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Mauricio"}'

# Salvar e usar cookies
curl -c cookies.txt https://exemplo.com/login
curl -b cookies.txt https://exemplo.com/perfil

# Seguir redirects automaticamente
curl -L https://bit.ly/algumacoisa
```

---

## 🏗️ Conceitos de Deploy

Tudo isso aparece quando você coloca uma aplicação em produção.

### 🌍 CDN (Content Delivery Network)

Servidores espalhados pelo mundo que servem assets estáticos (imagens, CSS, JS) **do servidor mais próximo do usuário**.

Exemplos: **Cloudflare**, AWS CloudFront, Akamai, Fastly.

> 💡 **Por que importa?** Latência muito menor pra usuário internacional. Um usuário no Brasil pega seus assets de um servidor em São Paulo em vez de buscar na origem nos EUA.

### ⚖️ Load Balancer

Distribui requisições entre **vários servidores idênticos**.

```
                    ┌─────────────┐    ┌─→ Server 1
                    │ Load        │    ├─→ Server 2
   Usuários  →     │ Balancer    │ ──→├─→ Server 3
                    └─────────────┘    └─→ Server 4
```

Estratégias comuns: Round Robin, Least Connections, IP Hash, Weighted.

### 🛡️ Reverse Proxy

Servidor na frente da sua aplicação que adiciona funcionalidades:

- 🔒 Termina HTTPS (decifra TLS antes de passar pro app)
- 🗜️ Comprime respostas (gzip, brotli)
- 💾 Faz cache de respostas
- 🚦 Rate limiting (limita requisições por IP)
- 📊 Logging centralizado

Mais usados: **Nginx**, **Caddy** (auto SSL), **Traefik**, **HAProxy**.

> 🤔 **Padrão comum em produção**:
> Usuário → Cloudflare (CDN/WAF) → Nginx (reverse proxy) → seu app Spring Boot
>
> Cada camada agrega valor: a CDN serve assets, o Nginx termina TLS e cacheia, seu app só processa o que é dinâmico.

---

## ✅ Resumo da página

- Redes funcionam em **camadas** (Aplicação → Transporte → Rede → Física)
- **IP** identifica máquina; **porta** identifica serviço
- **NAT** permite que vários dispositivos compartilhem 1 IP público
- **TCP** é confiável; **UDP** é rápido sem garantias
- **DNS** traduz nomes pra IPs; mudanças levam tempo pelo cache (TTL)
- **HTTP** é o protocolo da web; **HTTPS** adiciona criptografia (TLS) e autenticidade (certificados)
- **CORS** é proteção do navegador, não erro do servidor
- Ferramentas do dia a dia: `ping`, `curl`, `dig`, `netstat`/`ss`
- Em produção: **CDN** (proximidade) + **Load Balancer** (distribuição) + **Reverse Proxy** (TLS, cache)

---

⬅️ [Anterior: Linux e Arch](./06-linux-arch.md) | 🔙 [Índice](./README.md) | ➡️ [Próximo: Banco de Dados](./08-banco-de-dados.md)
