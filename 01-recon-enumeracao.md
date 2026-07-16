# Recon / Enumeração

## NMAP | escaneamento de rede | "o que tá aberto" | scan de portas | primeiro passo (nmap / port scan / host discovery)

```
nmap -sn 192.168.1.0/24        # descobre hosts vivos na rede, sem escanear porta (host discovery)
nmap -sV -sC <IP>              # padrão: versão dos serviços + scripts básicos — usar isso depois do -sn
nmap -p- <IP>                  # todas as 65535 portas (não só top 1000)
nmap -p 21,22,80,443 <IP>      # só portas específicas
nmap -sU <IP>                  # portas UDP (padrão do nmap é só TCP)
nmap -A <IP>                   # agressivo: -sV -sC -O + traceroute
nmap -Pn <IP>                  # pula ping inicial — usar se o host não responde ping mas responde nas portas
nmap -oN saida.txt <IP>        # salva resultado em arquivo
nmap --script vuln <IP>        # scripts de vulnerabilidade conhecida
```
- Ordem: `-sn` na rede toda → `-sV -sC` no alvo → só se precisar, `-p-` ou `-A`.
- Host não responde ping padrão? Adicionar `-Pn`.
- Buscar na doc: `nmap cheat sheet`, `nmap scripts vuln`

## DESCOBERTA DE DIRETÓRIO WEB | gobuster/dirb | pasta escondida | achar página que não tá no menu (directory brute force / web fuzzing)

```
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html   # testa extensão junto
gobuster vhost -u http://<IP> -w <wordlist>     # virtual hosts diferentes no mesmo IP
gobuster dns -d <dominio> -w <wordlist>         # subdomínios
gobuster fuzz -u http://<IP>/FUZZ -w <wordlist> # modo genérico, marca FUZZ onde quiser testar
dirb http://<IP> /usr/share/wordlists/dirb/common.txt   # alternativa mais simples
```
- Regra: porta 80/443 aberta → começar com `dir`.
- Achou caminho novo → acessar pela **URL no navegador**, não pelo terminal local (não é `cd`).
- Wordlist padrão não achou nada → subir de tamanho (`seclists`) antes de desistir.
- Buscar: `gobuster modes`, `dirb vs gobuster`

## SMB / COMPARTILHAMENTO WINDOWS | Samba | portas 139/445 | pasta compartilhada de rede (SMB enumeration)

```
smbclient -L //<IP>/ -N        # lista compartilhamentos sem autenticação (-N = sem senha)
smbclient //<IP>/nome_share -N # entra num compartilhamento específico (get/put dentro, igual FTP)
smbmap -H <IP>                 # mostra shares JÁ com permissão (leitura/escrita) sem precisar entrar um por um
enum4linux -a <IP>             # enumeração completa: usuários, shares, políticas
```
- Vetor comum: share sem senha com arquivo de config/credencial esquecido dentro.
- `smbmap` é mais rápido que testar `smbclient` manualmente em cada share — já mostra READ/WRITE de cada uma na primeira tela.
- Buscar: `smb enumeration cheat sheet`, `enum4linux usage`, `smbmap usage`

## NETCAT (`nc`) | canivete suíço de rede | testar porta na mão | pegar banner | mandar arquivo (netcat multi-use)

```
nc -zv <IP> 21-100              # testa rapidamente quais portas nessa faixa respondem (scan simples, sem nmap)
nc <IP> <porta>                 # conecta e mostra o banner (versão/mensagem inicial) de QUALQUER serviço, não só FTP
nc -lvnp <porta>                # fica ESCUTANDO nessa porta no meu Kali (usado pra receber reverse shell)
nc -lvnp <porta> > recebido.txt # recebe um arquivo que o alvo mandar
nc <IP> <porta> < arquivo.txt   # manda um arquivo pro alvo que estiver escutando do outro lado
```
- `nc <IP> <porta>` sem mais nada é o jeito mais rápido de "cutucar" uma porta e ver o que ela responde, antes de decidir qual ferramenta específica usar.
- `-l` = listen (escuta), `-v` = verbose (mostra o que tá acontecendo), `-n` = não resolve DNS (mais rápido), `-p` = porta.
- Buscar: `netcat cheat sheet`, `nc banner grabbing`

## WORDLIST | lista de senha | dicionário | "lista de palavras pra tentar" (password list / dictionary)

```
/usr/share/wordlists/rockyou.txt                                   # +14 milhões de senhas reais vazadas — a mais usada
gunzip /usr/share/wordlists/rockyou.txt.gz                         # se vier compactada
/usr/share/wordlists/dirb/common.txt                                # padrão pra gobuster/dirb (pequena, rápida)
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt        # maior, se common.txt não achar nada
/usr/share/seclists/                                                 # coleção enorme (usuários/senhas/subdomínios) — pode precisar apt install seclists
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt   # top senhas mais usadas — MUITO mais rápido que rockyou inteiro
/usr/share/seclists/Usernames/top-usernames-shortlist.txt           # lista curta de usuários mais comuns (admin, root, user...)
/usr/share/seclists/Usernames/Names/names.txt                       # nomes próprios — útil quando o site tem página "Equipe/Sobre" com nomes reais
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt   # alternativa pro gobuster/ffuf quando common.txt/medium não acham nada
/usr/share/wordlists/fasttrack.txt                                  # wordlist pequena e "esperta" (senhas comuns + variações), boa pra testar rápido antes da grande
head -n 1000 /usr/share/wordlists/rockyou.txt > rapida.txt          # corta a wordlist gigante pra testar mais rápido sob pressão de tempo
```
- Regra geral: `rockyou.txt` pra senha (hydra/hash cracking) quando não tem pressa; **sob tempo de prova, preferir as listas "top"/"short" do seclists ou `fasttrack.txt`** — acham a senha certa quase tão bem, só que em segundos em vez de minutos/horas.
- `dirb/common.txt` pra achar diretório/arquivo em site; se não achar nada, subir pra `raft-medium-directories.txt` antes de desistir.
- Não sabe se o pacote `seclists` tá instalado? `locate seclists` ou `apt install seclists` (precisa internet/sudo).
- Buscar: `seclists github`, `rockyou.txt location kali`, `seclists wordlist categories`

## COMANDOS BÁSICOS DE TERMINAL | não lembro o comando | shell no alvo (basic linux commands)

```
ls / cd / mkdir / clear / man <comando>
pwd            # em que pasta eu tô
whoami         # qual usuário eu sou
touch arquivo  # cria arquivo vazio
nano arquivo   # editor de texto (Ctrl+X salva)
cat arquivo    # mostra conteúdo do arquivo
chmod ...      # muda permissão (r=ler, w=escrever, x=executar; "-" = sem permissão)
tail -f log    # últimas 10 linhas, -f atualiza ao vivo
grep "ERRO" arquivo.log   # procura texto dentro do arquivo
```
- `chmod` MUDA permissão, `ls -l` LISTA a permissão — não confundir os dois.
- Buscar: `linux basic commands cheat sheet`

## FUZZING | jogar lixo pra ver o que quebra | travar o programa de propósito (fuzzing)

- Manda entrada aleatória/corrompida num campo/arquivo/API até achar falha ou travamento.
- Tipos: black-box (sem ver código) / white-box (usa o código pra guiar) / mutation-based (corrompe input válido) / generation-based (cria do zero seguindo gramática).
- `ffuf` é a ferramenta rápida pra isso em web (parâmetro/URL) — mais rápida que gobuster pra ver diferença de tamanho/status de resposta.
- **Só funciona em web (porta 80/443)** — não existe fuzzing pra FTP puro; se só tem FTP/SSH sem web, ver seção "SÓ FTP/SSH" no arquivo `02-exploracao-shells.md`.

```
ffuf -u http://<IP>/FUZZ -w /usr/share/wordlists/dirb/common.txt              # achar diretório/arquivo escondido
ffuf -u http://<IP>/login.php -X POST -d "username=FUZZ&password=teste" -w usuarios.txt -fc 401   # achar USUÁRIO certo (varia usuário, senha fixa)
ffuf -u http://<IP>/login.php -X POST -d "username=admin&password=FUZZ" -w /usr/share/wordlists/rockyou.txt -fc 401  # achar SENHA (usuário já sabido, varia senha)
```
- `-X POST` = envia como formulário. `-d` = corpo da requisição com `FUZZ` no lugar do valor que tá testando. `-fc 401` = esconde da tela as tentativas com esse código de erro (só mostra o que for diferente = achou).
- Por que é mais rápido que hydra pra web: hydra também serve, mas `ffuf` já é feito pra HTTP e mostra tamanho de resposta na hora, facilitando notar a diferença visualmente.
- Buscar: `ffuf usage`, `ffuf post form fuzzing`, `web fuzzing wordlist`

### FFUF — opções extras que ajudam sob tempo de prova

```
ffuf -u http://<IP>/FUZZ -w wordlist.txt -t 100          # -t = threads simultâneas (mais rápido; padrão é 40)
ffuf -u http://<IP>/FUZZ -w wordlist.txt -mc 200,301,302 # -mc = só mostra esses códigos de resposta (match code)
ffuf -u http://<IP>/FUZZ -w wordlist.txt -fs 1234        # -fs = esconde respostas de tamanho X (útil quando toda página "não existe" devolve o mesmo tamanho, aí o diferente é o achado)
ffuf -u http://<IP>/FUZZ.php -w wordlist.txt             # testa uma extensão fixa direto no nome
ffuf -u http://<IP>/FUZZ -w wordlist.txt -recursion -recursion-depth 2   # entra sozinho nas pastas que achar (não precisa rodar de novo manualmente em cada uma)
```
- **`-fs` (filtrar por tamanho) costuma ser mais confiável que olhar código de status** — muita página de erro devolve `200 OK` mesmo não existindo de verdade, mas o tamanho do conteúdo quase sempre é diferente do achado real.
- `-t 100` acelera bastante — útil quando o tempo de prova tá curto e a wordlist é grande.
- Buscar: `ffuf flags`, `ffuf recursive scan`

## RECON ATIVO | mapear o alvo | descobrir o que existe antes de atacar (active reconnaissance)

- Interagir direto com o alvo (manda requisição/conexão) e ler a resposta — mesmo silêncio é informação (porta fechada, serviço não existe, ou firewall bloqueando).
- Ordem mental: varredura de porta → detecção de serviço → detecção de SO → enumeração a fundo do serviço aberto → cruzar com vulnerabilidade conhecida.
- É basicamente o "porquê" por trás de rodar nmap.
