# Estratégia / Diagnóstico ("travei, e agora?")

## MAPA DE SINTOMA → CAUSA | mensagem de erro | por onde procurar (error message → root cause map)

| Sintoma / mensagem | Provável causa | Onde ir |
| --- | --- | --- |
| `Connection refused` | chegou na máquina, mas a porta/serviço recusou (nada escutando ali) | conferir se é a porta certa, tentar outra |
| `Network is unreachable` | nem chegou a tentar — sem rota pra aquela faixa de IP | conferir rede/interface, CIDR, se tá na mesma sub-rede |
| `Permission denied` | falta permissão de arquivo, ou tentando ação sem privilégio | `ls -l`, `sudo -n -l`, ver se é o usuário certo |
| Login trava/nunca aceita | credencial errada ou brute-force sem wordlist certa | voltar pro recon (código-fonte, FTP anônimo) antes de insistir no hydra |
| Upload bloqueado por extensão | filtro de extensão ativo | [`02-exploracao-shells.md`](02-exploracao-shells.md) seção webshell — bypass de extensão |
| Tenho shell mas comando comum não existe | shell restrita/limitada | tentar `python3 -c 'import pty; pty.spawn("/bin/bash")'` pra virar shell completa |
| `sudo: command not found` mas `sudo -l` funcionou | binário não tá no PATH padrão | usar caminho completo do binário |

## RECURSOS PERMITIDOS NA PROVA (Red Team) | o que posso consultar (allowed resources — no AI)

Documentação/internet livre, **exceto IA** (nada de ChatGPT/Copilot/assistente embutido no navegador):
- **hacktricks.xyz** — metodologia geral, o mais completo
- **gtfobins.github.io** — abuso de binário SUID/sudo
- **PayloadsAllTheThings** (GitHub) — webshells/reverse shells prontos
- **revshells.com** — gerador de reverse shell (só IP/porta)
- **exploit-db.com** — exploits públicos

Hardening: **sem internet, sem nada consultável** — nem esse repositório.

## ERROS QUE JÁ TRAVARAM ANTES | não cair de novo nisso (known personal mistakes)

- Confundir o terminal do Kali com o terminal já dentro da máquina alvo — o prompt muda de aparência quando a conexão troca de lado.
- `/usr/...` (pasta de sistema) vs `/home/<usuario>/` (pasta pessoal, onde geralmente fica a flag).
- Pipe `|` (Alt Gr + `\` no ABNT) não é a mesma coisa que barra `/`.
- Ler CIDR (`/24` etc) e a saída do `nmap -sV -sC` com calma antes de agir.

## SE NÃO DEU CERTO, O QUE TENTAR DEPOIS | plano B de cada etapa | não travar sem próximo passo (fallback per technique)

**Nmap só achou o básico (poucas portas, nada interessante):**
```
nmap -p- <IP>                  # escaneia TODAS as portas, não só as ~1000 mais comuns (às vezes o serviço importante tá numa porta alta)
nmap -sU --top-ports 20 <IP>   # portas UDP mais comuns (nmap padrão só olha TCP)
nmap --script vuln <IP>        # roda scripts de vulnerabilidade conhecida contra o que já achou
```

**Gobuster/ffuf com wordlist padrão não achou nada no site:**
```
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   # wordlist maior
gobuster vhost -u http://<IP> -w <wordlist>          # pode ter OUTRO site escondido no mesmo IP (nome diferente)
nikto -h http://<IP>                                  # scanner automático que já aponta vulnerabilidade/arquivo comum conhecido
whatweb http://<IP>                                   # identifica tecnologia/framework do site (ajuda a saber que exploit procurar)
```
- Se nem wordlist maior achar nada, o caminho provavelmente não é web — voltar pra outros serviços abertos.

**Hydra/força bruta rodando há muito tempo sem achar nada:**
- Parar e reconsiderar: o usuário tá certo mesmo? Testar só 1 usuário de cada vez em vez de lista, pra confirmar rápido se é esse o problema.
- Verificar se o serviço **bloqueia conta após X tentativas erradas** (lockout) — se sim, força bruta grande pode travar a conta em vez de ajudar.
- Reduzir a wordlist (top 100/1000 senhas mais comuns) em vez de rodar a `rockyou.txt` inteira até o fim.

**Ganhei shell, mas comando trava ou "não existe" (shell limitada):**
```
python3 -c 'import pty; pty.spawn("/bin/bash")'   # vira shell completa (interativa), destrava comando que shell crua não suporta
```

**Checklist pós-shell (env/sudo -l/SUID/crontab/shadow) não achou caminho de escalada:**
```
find / -writable -type d 2>/dev/null | grep -v proc   # pastas que eu tenho permissão de escrever (pode achar script gravável fora do crontab)
getcap -r / 2>/dev/null                                # "capabilities" — outro jeito de binário rodar com poder extra, além de SUID/sudo (menos óbvio, GTFOBins também lista isso)
ps aux | grep root                                     # processo rodando como root pode ter porta interna/serviço que só o localhost acessa
```
- `getcap` é o item que mais gente esquece de checar — parecido com SUID, mas é outro mecanismo (ver `gtfobins.github.io` também tem seção "Capabilities").

**Travei em TUDO, sem ideia nenhuma do próximo passo:**
1. Reler o enunciado da tarefa do zero — muita trava é por perder um detalhe pedido (nome de arquivo, usuário específico, porta mencionada no texto).
2. Confirmar que o IP/alvo ainda é o certo (`ping`, `nmap -sn` de novo — às vezes a VM reinicia e muda de IP).
3. Ver se tem OUTRA máquina/alvo na mesma rede que eu ainda não escaneei (`nmap -sn <rede>/24`).
4. Só depois de tentar os 3 acima: considerar pedir dica (desconta ponto, ver seção abaixo).

## QUANDO PEDIR DICA / QUANDO PULAR | gestão de tempo na prova (time management / hint policy)

- Pedir dica ao avaliador **desconta ponto** — só pedir se travou de verdade (não achou nem por recon nem por busca na documentação em ~15min naquela etapa).
- Timer sugerido: **15 minutos por tarefa/etapa**. Pesquisar `timer 15 minutes` no navegador abre um cronômetro direto no buscador, sem precisar de app.
- Estourou o tempo numa etapa sem avançar nada: anotar mentalmente onde parou, pular pra próxima tarefa se possível, voltar depois se sobrar tempo — não travar o resto da prova numa etapa só.

## FLUXO GERAL DE RACIOCÍNIO (methodology overview)

```
nmap (recon)
  → identificar serviço/porta interessante
    → testar acesso (anônimo / credencial padrão / upload) → foothold
      → shell de usuário comum
        → checklist pós-shell (env, sudo -l, SUID, crontab, shadow)
          → escalar pra root
```

## PORTAS DE CABEÇA (common ports)

| Porta | Serviço | Primeira coisa a tentar |
| --- | --- | --- |
| 21 | FTP | login anônimo |
| 22 | SSH | força bruta só depois de ter usuário/wordlist |
| 80 / 443 | HTTP/HTTPS | código-fonte + gobuster + credencial padrão |
| 139 / 445 | SMB | smbclient / enum4linux |
| 8080 | Tomcat | `tomcat_mgr_login` (Metasploit) antes de hydra manual |
