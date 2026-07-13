# Comandos Nmap

Nmap — variações

nmap -sn 192.168.1.0/24 → descobre hosts vivos na rede (sem escanear portas)

nmap -sV -sC <IP> → padrão: versão dos serviços + scripts básicos

nmap -p- <IP> → escaneia TODAS as 65535 portas (não só top 1000)

nmap -p 21,22,80,443 <IP> → só portas específicas

nmap -sU <IP> → portas UDP (padrão do nmap é só TCP)

nmap -A <IP> → agressivo: -sV -sC -O (detecção de SO) + traceroute

nmap -Pn <IP> → pula o ping inicial (usar se o host bloqueia ICMP mas responde nas portas)

nmap -oN saida.txt <IP> → salva o resultado num arquivo

nmap --script vuln <IP> → roda scripts de detecção de vulnerabilidade conhecida

Ordem de uso típica: primeiro -sn na rede toda pra achar o alvo, depois -sV -sC no IP dele pra primeira olhada rápida, e só se precisar de mais detalhe usar -p- (todas as portas) ou -A (agressivo). Se o host não responder ao ping padrão, adicionar -Pn.