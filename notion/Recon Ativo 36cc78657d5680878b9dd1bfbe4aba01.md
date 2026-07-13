# Recon Ativo

---

## O que é

- É quando vc investiga algo(alvo) interagindo diretamente com ele. Vc coleta informaçoes.

## Como funciona

- Vc tenta mandar requisiçoes e conexoes cada resposta é uma pista. Mesmo q nao envie nd quer dizer que, a porta esta fechada, o serviço nn extiste, ou algo q esta bloqueando (como o firewall)…

## Para oq ele serve

- Serve para mapear, como um guia e quanto mais vc sabe sobre ele mais certeiro vc vai.
- Vc sabe oq tem la, oq existe e oq pode ser explorado.

## Tipos

- **Varredura de portas -** descobre quais portas estão abertas no alvo, quais portas(janelas) ele tem
- **Detecção de serviços -** descobre o que está rodando em cada porta (um servidor web, banco de dados,...)
- **Detecção de sistema operacional** - tenta adivinhar qual sistema o alvo usa com base em como ele responde
- **Enumeração** - vai fundo em um serviço específico para extrair o máximo de informação, como usuários, versoes, configurações
- **Varredura de vulnerabilidades** - já cruza o que encontrou com falhas conhecidas para ver o que pode ser explorado

---

# RESUMO

- Nos usamos ele para descobrir oq existe dentro do programa, igual quando usamos nmap para descobrir portas. Se ele te respondeu ou nao, entao existe algo ali q pode ser explorado, algo nn respondido pode ser q nao exista ou esta bloqueado.