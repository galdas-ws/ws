# Colinha de prova — WorldSkills Cyber Security

Anotações próprias de estudo, organizadas pra busca rápida com **Ctrl+F no arquivo Raw do GitHub** (a visualização normal corta arquivo grande e o Ctrl+F falha nela — sempre abrir em Raw primeiro).

Conteúdo público, sem IA embutida — só texto, do jeito que a prova de Red Team permite (documentação/internet, sem IA). O bloco de Hardening é referência de estudo geral: **na prova de Hardening não é permitido consultar nada, nem isso aqui** — ver aviso no topo do arquivo 04.

## Regra de 10 segundos — qual arquivo abrir

| Sintoma / o que eu tô pensando | Arquivo |
| --- | --- |
| "não sei o que tá rodando nessa máquina", preciso escanear/mapear primeiro | [`01-recon-enumeracao.md`](01-recon-enumeracao.md) |
| já sei a porta/serviço, preciso de senha/login/acesso inicial (foothold) | [`02-exploracao-shells.md`](02-exploracao-shells.md) |
| já tenho shell de usuário comum, "e agora?", preciso virar root | [`03-pos-exploit-escalada.md`](03-pos-exploit-escalada.md) |
| firewall/iptables/hardening de Linux (só referência — **não usar ao vivo na prova de Hardening**) | [`04-hardening-linux-firewall.md`](04-hardening-linux-firewall.md) |
| travei, não sei se peço dica, quanto tempo já gastei, o que fazer | [`05-estrategia-diagnostico.md`](05-estrategia-diagnostico.md) |

## Como usar no dia da prova
1. Abrir o repo no navegador (sem login — checar antes numa aba anônima).
2. Entrar no arquivo do tema certo (tabela acima).
3. Clicar em **Raw**.
4. `Ctrl+F` e buscar pela palavra que eu usaria sob pressão (os títulos dos blocos têm sinônimo/sintoma, não só o nome técnico).
