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
