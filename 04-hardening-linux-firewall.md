> ⚠️ **A prova de Hardening não permite consulta de NENHUM material — nem isso aqui.** Esse arquivo é só referência de estudo/revisão antes da prova, não é pra abrir durante o bloco de Hardening. Só o bloco de Red Team permite documentação/internet.

# Hardening / Firewall Linux (referência de estudo)

## IPTABLES | firewall Linux | chains/tabelas | "bloquear/liberar tráfego" (iptables / linux firewall)

Tabelas: `filter` (padrão, ACCEPT/DROP/REJECT) · `nat` (tradução de endereço) · `mangle` (edita cabeçalho, raro).

Chains da tabela filter:
- `INPUT` — pacote **destinado ao próprio servidor**
- `OUTPUT` — pacote **gerado pelo próprio servidor**
- `FORWARD` — pacote **passando pelo servidor** (roteado de uma rede pra outra)

```
iptables -F                      # limpa regras da tabela filter
iptables -X                      # remove chains customizadas
iptables -t nat -F && iptables -t nat -X
iptables -t mangle -F && iptables -t mangle -X

iptables -P INPUT   DROP         # política padrão: bloqueia tudo que não tiver regra explícita
iptables -P FORWARD DROP
iptables -P OUTPUT  DROP

iptables -A INPUT  -i lo -j ACCEPT       # loopback sempre liberado (senão serviço local quebra)
iptables -A OUTPUT -o lo -j ACCEPT
iptables -A INPUT   -m state --state ESTABLISHED,RELATED -j ACCEPT   # resposta de conexão já permitida
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT  -m state --state ESTABLISHED,RELATED -j ACCEPT

# regra específica: origem -d destino -p protocolo --dport porta -j AÇÃO
iptables -A FORWARD -s <IP_ORIGEM> -d <IP_DESTINO> -p tcp --dport <PORTA> -j ACCEPT
iptables -A OUTPUT  -d <IP> -p udp --dport 53 -j ACCEPT                # exemplo: DNS

# NAT — servidor de rede interna acessando internet precisa de MASQUERADE também
iptables -t nat -A POSTROUTING -s <IP_INTERNO> -o <INTERFACE_WAN> -j MASQUERADE

iptables -L -v --line-numbers     # listar regras (com número, pra deletar uma específica)
iptables -D <CHAIN> <numero>      # deletar regra pelo número

echo 1 > /proc/sys/net/ipv4/ip_forward                                          # liga roteamento (temporário)
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf        # permanente
sysctl -p

mkdir -p /etc/iptables
iptables-save > /etc/iptables/rules.v4     # salva regras (senão somem no reboot)
iptables-restore < /etc/iptables/rules.v4  # restaura manualmente
```
- Regra de origem/destino: `-s` = **s**ource (quem manda), `-d` = **d**estination (quem recebe). `-i` interface de entrada, `-o` de saída.
- Filosofia: começar com `DROP` geral e só abrir o necessário — nunca começar com `ACCEPT` geral tentando bloquear depois.
- Buscar: `iptables cheat sheet`, `iptables forward vs input vs output`

## UFW | firewall simplificado | alternativa ao iptables puro (uncomplicated firewall)

```
apt install ufw -y
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp        # libera porta específica
ufw enable
ufw status verbose
```
- Mais rápido de configurar que iptables puro quando a regra é simples (entrada bloqueada/saída liberada + exceções pontuais).

## SSH HARDENING | travar acesso remoto | desabilitar login root (SSH hardening)

```
sed -i -E 's/^#?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i -E 's/^#?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config   # se já tiver chave configurada
systemctl restart sshd
```
- `-i` edita o arquivo de verdade, `-E` regex estendida, `#?` acha a linha comentada ou não.
- Sempre reiniciar o serviço depois de editar config, senão não aplica.

## DESLIGAR SERVIÇO DESNECESSÁRIO | reduzir superfície de ataque | serviço que não devia estar rodando (disable unused service)

```
systemctl stop <serviço>.service <serviço>.socket
systemctl disable <serviço>.service <serviço>.socket
```
- Se o `stop` ficar "cancelado" voltando sozinho: o serviço tem **socket activation** (recebe tráfego de rede e religa numa corrida contra o stop) — usar `mask` antes:
```
systemctl mask <serviço>.service <serviço>.socket
systemctl stop <serviço>.service <serviço>.socket
```
- Checar o que tá escutando antes de decidir o que desligar: `ss -tulnp` (t=TCP, u=UDP, l=listening, n=numeric, p=processo).

## POLÍTICA DE SENHA | usuário/grupo/expiração (user & password policy)

```
useradd -m -s /bin/bash -G grupo usuario   # cria usuário (home -m, shell -s, grupo -G)
usermod -aG outrogrupo usuario             # adiciona a outro grupo sem tirar dos atuais
usermod -s /usr/sbin/nologin usuario       # bloqueia login interativo (conta de serviço)
passwd usuario                             # troca senha
chage -l usuario      # status de expiração
chage -d 0 usuario     # força troca de senha no próximo login
chage -M 90 usuario    # expira a cada 90 dias
chage -m 1 usuario     # mínimo 1 dia entre trocas
groupadd/groupdel grupo
userdel -r usuario     # remove usuário + home
```
```
# /etc/security/pwquality.conf — regra de COMPLEXIDADE (chage controla validade, pwquality controla qualidade)
minlen = 12
dcredit = -1     # exige dígito
ucredit = -1     # exige maiúscula
lcredit = -1     # exige minúscula
ocredit = -1     # exige símbolo
minclass = 3     # quantas classes de caractere diferentes exigir
retry = 3
```
```
tee -a /etc/security/pwquality.conf << 'EOF'
regra1
regra2
EOF
```
(`tee -a` escreve na tela E no arquivo ao mesmo tempo, sem apagar o que já tinha; heredoc `<<'EOF'` deixa digitar várias linhas até `EOF` sozinho numa linha fechar o bloco — junta os dois pra escrever regra em lote sem repetir `echo >>`.)

## AUDITORIA (auditd) | vigiar sem bloquear | log de quem fez o quê (audit logging)

```
auditctl -w /etc/passwd -p wa -k passwd_changes    # -w vigia arquivo, -p tipo de ação (w=write,a=append,x=execute), -k etiqueta
ausearch -k passwd_changes                          # busca depois pelo rótulo
```
- Não impede a ação, só registra — serve pra saber quem/quando fez algo depois.
