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
smbclient //<IP>/nome_share -N # entra num compartilhamento específico
enum4linux -a <IP>             # enumeração completa: usuários, shares, políticas
```
- Vetor comum: share sem senha com arquivo de config/credencial esquecido dentro.
- Buscar: `smb enumeration cheat sheet`, `enum4linux usage`

## WORDLIST | lista de senha | dicionário | "lista de palavras pra tentar" (password list / dictionary)

```
/usr/share/wordlists/rockyou.txt                                   # +14 milhões de senhas reais vazadas — a mais usada
gunzip /usr/share/wordlists/rockyou.txt.gz                         # se vier compactada
/usr/share/wordlists/dirb/common.txt                                # padrão pra gobuster/dirb
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt        # maior, se common.txt não achar nada
/usr/share/seclists/                                                 # coleção enorme (usuários/senhas/subdomínios) — pode precisar apt install seclists
head -n 50000 /caminho/rockyou.txt > rapida.txt                    # corta a wordlist gigante pra testar mais rápido sob pressão de tempo
```
- Regra: `rockyou.txt` pra senha (hydra/hash cracking), `dirb/common.txt` pra achar diretório/arquivo em site.
- Buscar: `seclists github`, `rockyou.txt location kali`

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
- `ffuf` é a ferramenta rápida pra isso em web (parâmetro/URL).
- Buscar: `ffuf usage`, `web fuzzing wordlist`

## RECON ATIVO | mapear o alvo | descobrir o que existe antes de atacar (active reconnaissance)

- Interagir direto com o alvo (manda requisição/conexão) e ler a resposta — mesmo silêncio é informação (porta fechada, serviço não existe, ou firewall bloqueando).
- Ordem mental: varredura de porta → detecção de serviço → detecção de SO → enumeração a fundo do serviço aberto → cruzar com vulnerabilidade conhecida.
- É basicamente o "porquê" por trás de rodar nmap.
