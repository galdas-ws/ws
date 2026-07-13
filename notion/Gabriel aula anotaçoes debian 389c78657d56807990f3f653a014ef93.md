# Gabriel aula anotaçoes debian

---

# Guia Completo — Firewall iptables no Debian

---

## Sumário

1. [O que é o iptables?](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#1-o-que-%C3%A9-o-iptables)
2. [Topologia da Rede](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#2-topologia-da-rede)
3. [Conceitos Essenciais](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#3-conceitos-essenciais)
4. [Pré-requisitos e Instalação](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#4-pr%C3%A9-requisitos-e-instala%C3%A7%C3%A3o)
5. [Passo 1 — Habilitar IP Forwarding](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#5-passo-1--habilitar-ip-forwarding)
6. [Passo 2 — Limpar Regras Existentes](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#6-passo-2--limpar-regras-existentes)
7. [Passo 3 — Definir Política Padrão](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#7-passo-3--definir-pol%C3%ADtica-padr%C3%A3o)
8. [Passo 4 — Regras Base](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#8-passo-4--regras-base)
9. [Regra 1 — Firewall acessa DNS do Google](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#9-regra-1--firewall-acessa-dns-do-google)
10. [Regra 2 — Servidor Web acessa Servidor FTP](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#10-regra-2--servidor-web-acessa-servidor-ftp)
11. [Regra 3 — Servidor Web acessa a Internet](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#11-regra-3--servidor-web-acessa-a-internet)
12. [Regra 4 — Servidor FTP acessa Internet na porta 8080](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#12-regra-4--servidor-ftp-acessa-internet-na-porta-8080)
13. [Regra 5 — Servidor FTP acessa gerenciamento do Firewall](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#13-regra-5--servidor-ftp-acessa-gerenciamento-do-firewall)
14. [Regra 6 — ICMP entre as redes](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#14-regra-6--icmp-entre-as-redes)
15. [Passo 5 — Salvar as Regras](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#15-passo-5--salvar-as-regras)
16. [Como Verificar as Regras](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#16-como-verificar-as-regras)
17. [Script Completo](https://www.notion.so/Gabriel-aula-anota-oes-debian-389c78657d56807990f3f653a014ef93#17-script-completo)

---

## 1. O que é o iptables?

O **iptables** é a ferramenta padrão de firewall do Linux. Ele funciona como um conjunto de regras que o kernel do Linux usa para decidir o que fazer com cada pacote de rede que chega, sai ou passa pelo servidor.

O iptables organiza as regras em três estruturas principais:

### Tabelas

São os "módulos" de funcionalidade do iptables:

| Tabela | Função |
| --- | --- |
| `filter` | Tabela padrão. Filtra pacotes (ACCEPT, DROP, REJECT). |
| `nat` | Tradução de endereços de rede (NAT/MASQUERADE/DNAT/SNAT). |
| `mangle` | Modifica cabeçalhos dos pacotes (menos usado no dia a dia). |

Quando você não especifica a tabela no comando, o iptables usa a tabela `filter` automaticamente.

### Cadeias (Chains)

Dentro de cada tabela existem cadeias. As cadeias padrão da tabela `filter` são:

| Cadeia | Quando é usada |
| --- | --- |
| `INPUT` | Pacotes **destinados ao próprio servidor/firewall**. |
| `OUTPUT` | Pacotes **gerados pelo próprio servidor/firewall**. |
| `FORWARD` | Pacotes que **passam pelo servidor** (roteados de uma rede a outra). |

### Fluxo de um pacote no iptables

```
Pacote chega pela interface de rede
          |
          v
    [PREROUTING] (tabela nat — ex: DNAT)
          |
          v
   O pacote é para este servidor?
      /            \
    SIM            NÃO
     |               |
  [INPUT]        [FORWARD]
     |               |
  Processo       [POSTROUTING] (tabela nat — ex: MASQUERADE)
  local              |
     |          Sai pela interface de rede
  [OUTPUT]
     |
  [POSTROUTING]
```

### Ações (Targets)

O que fazer com o pacote quando a regra bate:

| Ação | O que faz |
| --- | --- |
| `ACCEPT` | Permite o pacote passar. |
| `DROP` | Descarta o pacote silenciosamente (sem aviso ao remetente). |
| `REJECT` | Descarta o pacote e avisa o remetente com uma mensagem de erro. |
| `MASQUERADE` | Substitui o IP de origem pelo IP da interface de saída (NAT). |
| `LOG` | Registra o pacote no log do sistema (não para o processamento). |

---

## 2. Topologia da Rede

```
                        ┌──────────────────────────┐
                        │         INTERNET          │
                        └────────────┬─────────────┘
                                     │
                               Interface eth0
                               (IP público / WAN)
                        ┌────────────┴─────────────┐
                        │         FIREWALL          │
                        │  (Servidor Debian)        │
                        │  eth1: 192.168.10.1       │
                        │  eth2: 192.168.20.1       │
                        └──────┬───────────┬────────┘
                               │           │
                         Interface      Interface
                           eth1            eth2
                               │           │
               ┌───────────────┘           └──────────────┐
               │                                          │
       192.168.10.0/24                          192.168.20.0/24
       ┌───────────────┐                       ┌──────────────────┐
       │  Servidor Web │                       │  Servidor FTP    │
       │ 192.168.10.10 │                       │ 192.168.20.20    │
       └───────────────┘                       └──────────────────┘
```

### Mapeamento de IPs e interfaces

| Dispositivo | Interface | Endereço IP | Rede |
| --- | --- | --- | --- |
| Firewall (WAN) | eth0 | IP do provedor | Internet |
| Firewall (LAN1) | eth1 | 192.168.10.1 | 192.168.10.0/24 |
| Firewall (LAN2) | eth2 | 192.168.20.1 | 192.168.20.0/24 |
| Servidor Web | — | 192.168.10.10 | 192.168.10.0/24 |
| Servidor FTP | — | 192.168.20.20 | 192.168.20.0/24 |

---

## 3. Conceitos Essenciais

### O que é IP Forwarding?

Por padrão, o Linux não encaminha pacotes entre interfaces de rede. O **IP Forwarding** é a funcionalidade que permite ao servidor funcionar como um roteador/firewall, repassando pacotes de uma interface para outra (por exemplo, de `eth1` para `eth0`).

### O que é NAT / MASQUERADE?

**NAT (Network Address Translation)** é a tradução de endereços de rede. Quando um servidor da rede interna (ex: 192.168.10.10) acessa a internet, o firewall substitui o IP de origem privado pelo seu próprio IP público. O servidor de destino na internet responde ao IP público do firewall, que então encaminha a resposta para o servidor interno correto.

O `MASQUERADE` é uma forma dinâmica de SNAT (Source NAT) — ele detecta automaticamente o IP da interface de saída, sendo ideal quando o IP público pode mudar.

### O que é Stateful Firewall (--state)?

O iptables pode rastrear o **estado das conexões** usando o módulo `state` ou `conntrack`. Isso evita que você precise criar regras de retorno para cada regra criada.

| Estado | Significado |
| --- | --- |
| `NEW` | Novo pacote iniciando uma conexão. |
| `ESTABLISHED` | Pacote pertencente a uma conexão já estabelecida. |
| `RELATED` | Pacote relacionado a uma conexão existente (ex: FTP passivo). |
| `INVALID` | Pacote que não pertence a nenhuma conexão conhecida. |

---

## 4. Pré-requisitos e Instalação

### Verificar se o iptables está instalado

```bash
iptables --version
```

**O que faz:** Mostra a versão instalada do iptables. Se não estiver instalado, o comando retornará um erro.

### Instalar o iptables e o iptables-persistent

```bash
apt update && apt upgrade -y
```

**Linha por linha:**

- `apt update` — Atualiza a lista de pacotes disponíveis nos repositórios.
- `apt upgrade -y` — Instala as atualizações disponíveis. O `y` confirma automaticamente sem perguntar.
- `&&` — Executa o segundo comando **somente se** o primeiro tiver sucesso.

```bash
apt install iptables iptables-persistent -y
```

**O que faz:**

- `iptables` — O próprio firewall (já costuma vir instalado no Debian).
- `iptables-persistent` — Serviço que salva e restaura automaticamente as regras do iptables após reboot.
- `y` — Confirma a instalação automaticamente.

> Durante a instalação do `iptables-persistent`, ele perguntará se deseja salvar as regras atuais. Responda **"Yes"** nas duas perguntas (IPv4 e IPv6).
> 

---

## 5. Passo 1 — Habilitar IP Forwarding

Para que o firewall encaminhe pacotes entre as interfaces de rede (funcionar como roteador), o IP Forwarding precisa estar ativo.

### Habilitar temporariamente (até o próximo reboot)

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

**Linha por linha:**

- `echo 1` — Gera o valor `1` (habilitado).
- `>` — Redireciona a saída para o arquivo indicado.
- `/proc/sys/net/ipv4/ip_forward` — Arquivo virtual do kernel que controla o IP Forwarding. Escrever `1` ativa, `0` desativa.

**Atenção:** Esta alteração é perdida ao reiniciar o servidor.

### Habilitar permanentemente

```bash
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
```

**Linha por linha:**

- `sed` — Editor de texto em linha de comando (Stream Editor).
- `i` — Edita o arquivo diretamente (in-place), sem criar cópia.
- `'s/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/'` — Substitui (`s/`) o texto com `#` (comentado) pelo mesmo texto sem `#` (descomentado).
- `/etc/sysctl.conf` — Arquivo de configuração de parâmetros do kernel no Debian.

### Aplicar as configurações do sysctl.conf imediatamente

```bash
sysctl -p
```

**O que faz:** Recarrega as configurações do `/etc/sysctl.conf` sem precisar reiniciar.

### Verificar se está ativo

```bash
cat /proc/sys/net/ipv4/ip_forward
```

**Resultado esperado:** `1` (habilitado). Se retornar `0`, não está ativo.

---

## 6. Passo 2 — Limpar Regras Existentes

Antes de criar as novas regras, é importante remover qualquer configuração anterior para evitar conflitos.

```bash
iptables -F
```

**O que faz:** `-F` (Flush) — Remove **todas as regras** de todas as cadeias da tabela `filter` (padrão). As políticas padrão (INPUT, OUTPUT, FORWARD) não são alteradas.

```bash
iptables -X
```

**O que faz:** `-X` (Delete) — Remove todas as cadeias **customizadas** (criadas pelo usuário). As cadeias padrão (INPUT, OUTPUT, FORWARD) não são removidas.

```bash
iptables -t nat -F
```

**O que faz:** `-t nat` especifica a tabela `nat`. `-F` remove todas as regras dessa tabela. Isso limpa as regras de NAT/MASQUERADE existentes.

```bash
iptables -t nat -X
```

**O que faz:** Remove as cadeias customizadas da tabela `nat`.

```bash
iptables -t mangle -F
iptables -t mangle -X
```

**O que faz:** Limpa regras e cadeias customizadas da tabela `mangle` (modificação de pacotes). Boa prática mesmo que não haja regras lá.

---

## 7. Passo 3 — Definir Política Padrão

A política padrão define o que acontece com um pacote que **não casou com nenhuma regra**.

```bash
iptables -P INPUT   DROP
iptables -P FORWARD DROP
iptables -P OUTPUT  DROP
```

**Linha por linha:**

- `iptables -P` — Define a **P**olítica padrão (Policy).
- `INPUT DROP` — Todo pacote destinado ao firewall que não tiver regra explícita será descartado.
- `FORWARD DROP` — Todo pacote roteado que não tiver regra explícita será descartado.
- `OUTPUT DROP` — Todo pacote gerado pelo firewall que não tiver regra explícita será descartado.

**Por que usar DROP e não ACCEPT?**
A filosofia de segurança é: **"tudo que não é explicitamente permitido, é proibido"**. Começar com DROP e abrir apenas o necessário é muito mais seguro do que começar com ACCEPT e tentar bloquear o que é perigoso.

---

## 8. Passo 4 — Regras Base

Estas regras são necessárias para o funcionamento básico do sistema e devem ser criadas antes das regras específicas.

### Interface Loopback

```bash
iptables -A INPUT  -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
```

**Linha por linha:**

- `iptables -A INPUT` — **A**diciona uma regra no final da cadeia INPUT.
- `i lo` — `i` especifica a interface de **entrada** (input). `lo` é a interface de loopback (127.0.0.1).
- `j ACCEPT` — **J**umps to (pula para) a ação ACCEPT — permite o pacote.
- `o lo` — `o` especifica a interface de **saída** (output). `lo` é a interface de loopback.

**Por que é necessário?** O loopback é usado pelo próprio sistema operacional para comunicação interna entre processos. Sem esta regra, muitas aplicações param de funcionar (bancos de dados, servidores locais, etc.).

### Conexões Estabelecidas e Relacionadas (Stateful)

```bash
iptables -A INPUT   -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT  -m state --state ESTABLISHED,RELATED -j ACCEPT
```

**Linha por linha:**

- `m state` — Carrega o **m**ódulo `state` do iptables (rastreamento de estado de conexões).
- `-state ESTABLISHED,RELATED` — Aplica a regra apenas a pacotes cujo estado seja:
    - `ESTABLISHED`: pacotes de retorno de uma conexão já permitida.
    - `RELATED`: pacotes de uma conexão secundária relacionada (ex: canal de dados do FTP).
- `j ACCEPT` — Permite esses pacotes.

**Por que é necessário?** Com a política padrão DROP, sem esta regra, as respostas de conexões legítimas também seriam bloqueadas. Por exemplo: o firewall envia um DNS query para 8.8.8.8, a resposta vem, mas seria bloqueada sem esta regra. O estado ESTABLISHED/RELATED permite que respostas de conexões previamente autorizadas passem automaticamente.

---

## 9. Regra 1 — Firewall acessa DNS do Google

**Objetivo:** Permitir que o próprio firewall consulte o DNS do Google no endereço 8.8.8.8.

**DNS (Domain Name System)** é o serviço que traduz nomes de domínio (ex: [google.com](http://google.com/)) em endereços IP. Usa as portas **53/UDP** (consultas rápidas) e **53/TCP** (consultas maiores ou transferências de zona).

**Cadeia utilizada:** `OUTPUT` — porque o tráfego é **originado no próprio firewall**.

```bash
iptables -A OUTPUT -d 8.8.8.8 -p udp --dport 53 -j ACCEPT
```

**Linha por linha:**

- `iptables -A OUTPUT` — Adiciona regra no final da cadeia OUTPUT.
- `d 8.8.8.8` — `d` especifica o **d**estino do pacote. Apenas pacotes destinados ao IP 8.8.8.8 (DNS do Google) serão afetados.
- `p udp` — `p` especifica o **p**rotocolo. `udp` para consultas DNS padrão.
- `-dport 53` — Porta de destino (destination port) 53 — porta padrão do DNS.
- `j ACCEPT` — Permite o pacote.

```bash
iptables -A OUTPUT -d 8.8.8.8 -p tcp --dport 53 -j ACCEPT
```

**Diferença da linha anterior:** `-p tcp` em vez de `-p udp`. O DNS pode usar TCP quando a resposta é muito grande para caber em um pacote UDP (mais de 512 bytes) ou em transferências de zona DNS.

> **Nota:** A resposta do DNS (pacote de retorno do 8.8.8.8) é automaticamente permitida pela regra de estado `ESTABLISHED` criada no Passo 4.
> 

---

## 10. Regra 2 — Servidor Web acessa Servidor FTP

**Objetivo:** Permitir que o Servidor Web (192.168.10.10) acesse o Servidor FTP (192.168.20.20).

**FTP (File Transfer Protocol)** usa duas portas:

- **Porta 21/TCP** — Canal de controle (comandos FTP: login, listar arquivos, etc.)
- **Porta 20/TCP** — Canal de dados (transferência dos arquivos em si)

**Cadeia utilizada:** `FORWARD` — porque o pacote **entra pela `eth1`** (rede do Web) e **sai pela `eth2`** (rede do FTP), ou seja, **passa pelo firewall** sem ser destinado a ele.

```bash
iptables -A FORWARD -i eth1 -o eth2 \
    -s 192.168.10.10 -d 192.168.20.20 \
    -p tcp -m multiport --dports 20,21 \
    -j ACCEPT
```

**Linha por linha:**

- `iptables -A FORWARD` — Adiciona regra na cadeia FORWARD.
- `i eth1` — Pacote deve **entrar** (input) pela interface `eth1` (rede do Servidor Web).
- `o eth2` — Pacote deve **sair** (output) pela interface `eth2` (rede do Servidor FTP).
- `\` — Caractere de continuação de linha no shell (permite dividir o comando em várias linhas).
- `s 192.168.10.10` — `s` especifica o IP de **s**origem. Somente pacotes do Servidor Web.
- `d 192.168.20.20` — `d` especifica o IP de **d**estino. Somente pacotes para o Servidor FTP.
- `p tcp` — Protocolo TCP (FTP usa TCP).
- `m multiport` — Carrega o módulo `multiport`, que permite especificar múltiplas portas em uma só regra.
- `-dports 20,21` — Portas de destino: 20 (dados FTP) e 21 (controle FTP).
- `j ACCEPT` — Permite o pacote.

> **Nota:** A resposta do Servidor FTP de volta ao Servidor Web é automaticamente permitida pela regra `ESTABLISHED,RELATED` do Passo 4. O módulo RELATED é especialmente importante aqui, pois o FTP abre conexões secundárias para transferência de dados.
> 

---

## 11. Regra 3 — Servidor Web acessa a Internet

**Objetivo:** Permitir que o Servidor Web (192.168.10.10) acesse qualquer destino na internet.

Esta regra requer dois componentes:

1. **Regra FORWARD** — Para permitir o roteamento do pacote pelo firewall.
2. **Regra NAT (MASQUERADE)** — Para traduzir o IP privado em IP público.

**Cadeia FORWARD:**

```bash
iptables -A FORWARD -i eth1 -o eth0 \
    -s 192.168.10.10 \
    -j ACCEPT
```

**Linha por linha:**

- `A FORWARD` — Adiciona na cadeia FORWARD.
- `i eth1` — Pacote entra pela `eth1` (rede do Servidor Web).
- `o eth0` — Pacote sai pela `eth0` (interface WAN/Internet).
- `s 192.168.10.10` — Somente pacotes originados pelo Servidor Web.
- **Sem `d`** — Não há restrição de destino: pode acessar qualquer IP na internet.
- `j ACCEPT` — Permite o roteamento.

**Regra NAT — MASQUERADE:**

```bash
iptables -t nat -A POSTROUTING -s 192.168.10.10 -o eth0 -j MASQUERADE
```

**Linha por linha:**

- `t nat` — Especifica a tabela `nat`.
- `A POSTROUTING` — Adiciona na cadeia POSTROUTING. Esta cadeia é processada **após** a decisão de roteamento, antes do pacote sair pelo interface.
- `s 192.168.10.10` — Somente pacotes do Servidor Web.
- `o eth0` — Somente quando o pacote sair pela interface WAN (internet).
- `j MASQUERADE` — Substitui o IP de origem (192.168.10.10) pelo IP público da interface `eth0`.

**Por que MASQUERADE?** O Servidor Web tem um IP privado (192.168.10.10). Servidores na internet não conseguem responder para IPs privados. O MASQUERADE resolve isso fazendo o firewall "fingir" que é ele quem fez a requisição, e depois repassa a resposta ao Servidor Web.

---

## 12. Regra 4 — Servidor FTP acessa Internet na porta 8080

**Objetivo:** Permitir que o Servidor FTP (192.168.20.20) acesse a internet, mas **apenas na porta 8080/TCP**.

A diferença em relação à Regra 3 é a restrição de porta (`--dport 8080`), limitando o acesso apenas ao protocolo/porta específico.

**Cadeia FORWARD:**

```bash
iptables -A FORWARD -i eth2 -o eth0 \
    -s 192.168.20.20 \
    -p tcp --dport 8080 \
    -j ACCEPT
```

**Linha por linha:**

- `A FORWARD` — Adiciona na cadeia FORWARD.
- `i eth2` — Pacote entra pela `eth2` (rede do Servidor FTP).
- `o eth0` — Pacote sai pela `eth0` (WAN/Internet).
- `s 192.168.20.20` — Somente pacotes do Servidor FTP.
- `p tcp` — Protocolo TCP.
- `-dport 8080` — Somente para a porta de destino 8080. O Servidor FTP **não** poderá acessar outras portas na internet (como 80, 443, etc.).
- `j ACCEPT` — Permite o roteamento.

**Regra NAT — MASQUERADE:**

```bash
iptables -t nat -A POSTROUTING -s 192.168.20.20 -o eth0 -j MASQUERADE
```

**Linha por linha:**

- `t nat -A POSTROUTING` — Tabela NAT, cadeia POSTROUTING.
- `s 192.168.20.20` — Somente pacotes do Servidor FTP.
- `o eth0` — Quando sair pela interface WAN.
- `j MASQUERADE` — Traduz o IP privado 192.168.20.20 para o IP público do firewall.

---

## 13. Regra 5 — Servidor FTP acessa gerenciamento do Firewall

**Objetivo:** Permitir que o Servidor FTP (192.168.20.20) acesse a interface web de gerenciamento do firewall hospedada na **porta 443/TCP** (HTTPS).

**Cadeia utilizada:** `INPUT` — porque o destino **é o próprio firewall** (192.168.20.1), não um servidor em outra rede.

```bash
iptables -A INPUT -i eth2 \
    -s 192.168.20.20 -d 192.168.20.1 \
    -p tcp --dport 443 \
    -j ACCEPT
```

**Linha por linha:**

- `iptables -A INPUT` — Adiciona na cadeia INPUT (tráfego destinado ao próprio firewall).
- `i eth2` — Pacote deve entrar pela interface `eth2` (rede do Servidor FTP).
- `s 192.168.20.20` — Somente pacotes originados pelo Servidor FTP.
- `d 192.168.20.1` — Somente pacotes destinados ao IP do firewall na rede LAN2 (192.168.20.1).
- `p tcp` — Protocolo TCP (HTTPS usa TCP).
- `-dport 443` — Porta de destino 443 (HTTPS — página de gerenciamento do firewall).
- `j ACCEPT` — Permite o pacote.

**Por que é INPUT e não FORWARD?** Porque o Servidor FTP está se conectando **ao firewall em si** (IP 192.168.20.1), não a um servidor em outra rede. Tráfego destinado ao próprio servidor firewall sempre usa a cadeia INPUT.

---

## 14. Regra 6 — ICMP entre as redes

**Objetivo:** Permitir que os servidores das duas redes possam se "pingar" (testar conectividade) através do firewall.

**ICMP (Internet Control Message Protocol)** é o protocolo usado pelo comando `ping`. Ele verifica se um host está acessível na rede.

**Cadeia utilizada:** `FORWARD` — porque o ping de uma LAN para outra passa pelo firewall.

```bash
iptables -A FORWARD -i eth1 -o eth2 -p icmp --icmp-type echo-request -j ACCEPT
```

**Linha por linha:**

- `A FORWARD` — Cadeia FORWARD (tráfego roteado).
- `i eth1 -o eth2` — Entra pela rede do Web Server, sai para a rede do FTP.
- `p icmp` — Protocolo ICMP.
- `-icmp-type echo-request` — Tipo específico de ICMP: `echo-request` é o "ping" enviado. O retorno (`echo-reply`) é automaticamente permitido pela regra `ESTABLISHED,RELATED`.
- `j ACCEPT` — Permite.

```bash
iptables -A FORWARD -i eth2 -o eth1 -p icmp --icmp-type echo-request -j ACCEPT
```

**Diferença:** `-i eth2 -o eth1` — Direção inversa: o Servidor FTP pinga o Servidor Web (entra pela rede FTP, sai pela rede Web).

---

## 15. Passo 5 — Salvar as Regras

As regras criadas com `iptables` são armazenadas em memória. Quando o servidor reinicia, elas são perdidas. Para torná-las permanentes:

```bash
mkdir -p /etc/iptables
```

**O que faz:**

- `mkdir` — Cria um diretório.
- `p` — Cria os diretórios intermediários se não existirem, e não retorna erro se o diretório já existir.
- `/etc/iptables` — Diretório padrão para armazenar as regras persistentes.

```bash
iptables-save > /etc/iptables/rules.v4
```

**O que faz:**

- `iptables-save` — Exporta todas as regras atuais do iptables em formato texto.
- `>` — Redireciona a saída para o arquivo indicado (sobrescrevendo o conteúdo anterior).
- `/etc/iptables/rules.v4` — Arquivo onde o `iptables-persistent` busca as regras IPv4 na inicialização.

> O serviço `iptables-persistent` (instalado no Passo 4) carrega automaticamente o arquivo `/etc/iptables/rules.v4` durante o boot do sistema.
> 

**Para restaurar as regras manualmente:**

```bash
iptables-restore < /etc/iptables/rules.v4
```

---

## 16. Como Verificar as Regras

### Ver todas as regras da tabela filter

```bash
iptables -L -v --line-numbers
```

**Linha por linha:**

- `iptables -L` — **L**ista todas as regras.
- `v` — Modo **v**erboso: mostra contadores de pacotes e bytes, interfaces e outros detalhes.
- `-line-numbers` — Mostra o número de cada regra (útil para deletar uma regra específica).

### Ver regras da tabela NAT

```bash
iptables -t nat -L -v --line-numbers
```

- `t nat` — Especifica a tabela `nat`.

### Verificar IP Forwarding

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Resultado esperado: `1`

### Testar conectividade (após configurar o firewall)

```bash
# No firewall: testar DNS do Google (Regra 1)
ping -c 3 8.8.8.8
nslookup google.com 8.8.8.8

# No Servidor Web: testar acesso ao FTP (Regra 2)
telnet 192.168.20.20 21

# No Servidor Web: testar acesso à internet (Regra 3)
ping -c 3 8.8.8.8

# No Servidor FTP: testar porta 8080 (Regra 4)
curl -v <http://exemplo.com:8080>

# No Servidor FTP: testar gerenciamento do firewall (Regra 5)
curl -k <https://192.168.20.1>
```

---

## 17. Script Completo

```bash
#!/bin/bash
# ============================================================
# Firewall - iptables | Servidor Debian
# ============================================================

# --- Interfaces ---
WAN="eth0"    # Interface WAN (Internet)
LAN1="eth1"   # Rede do Servidor Web (192.168.10.0/24)
LAN2="eth2"   # Rede do Servidor FTP (192.168.20.0/24)

# --- IPs ---
IP_WEB="192.168.10.10"
IP_FTP="192.168.20.20"
IP_FW_LAN2="192.168.20.1"

# PASSO 1 - IP Forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sysctl -p

# PASSO 2 - Limpar regras
iptables -F && iptables -X
iptables -t nat -F && iptables -t nat -X
iptables -t mangle -F && iptables -t mangle -X

# PASSO 3 - Política padrão
iptables -P INPUT   DROP
iptables -P FORWARD DROP
iptables -P OUTPUT  DROP

# PASSO 4 - Regras base
iptables -A INPUT  -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
iptables -A INPUT   -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT  -m state --state ESTABLISHED,RELATED -j ACCEPT

# REGRA 1 - Firewall acessa DNS do Google
iptables -A OUTPUT -d 8.8.8.8 -p udp --dport 53 -j ACCEPT
iptables -A OUTPUT -d 8.8.8.8 -p tcp --dport 53 -j ACCEPT

# REGRA 2 - Servidor Web acessa Servidor FTP
iptables -A FORWARD -i $LAN1 -o $LAN2 \
    -s $IP_WEB -d $IP_FTP \
    -p tcp -m multiport --dports 20,21 \
    -j ACCEPT

# REGRA 3 - Servidor Web acessa a Internet
iptables -A FORWARD -i $LAN1 -o $WAN -s $IP_WEB -j ACCEPT
iptables -t nat -A POSTROUTING -s $IP_WEB -o $WAN -j MASQUERADE

# REGRA 4 - Servidor FTP acessa Internet na porta 8080
iptables -A FORWARD -i $LAN2 -o $WAN -s $IP_FTP -p tcp --dport 8080 -j ACCEPT
iptables -t nat -A POSTROUTING -s $IP_FTP -o $WAN -j MASQUERADE

# REGRA 5 - Servidor FTP acessa gerenciamento do Firewall (porta 443)
iptables -A INPUT -i $LAN2 -s $IP_FTP -d $IP_FW_LAN2 -p tcp --dport 443 -j ACCEPT

# REGRA 6 - ICMP entre as redes
iptables -A FORWARD -i $LAN1 -o $LAN2 -p icmp --icmp-type echo-request -j ACCEPT
iptables -A FORWARD -i $LAN2 -o $LAN1 -p icmp --icmp-type echo-request -j ACCEPT

# PASSO 5 - Salvar regras
mkdir -p /etc/iptables
iptables-save > /etc/iptables/rules.v4

echo "=== Firewall configurado com sucesso! ==="
iptables -L -v --line-numbers
```

---

*Documento gerado para fins educacionais — Configuração de Firewall iptables no Debian*