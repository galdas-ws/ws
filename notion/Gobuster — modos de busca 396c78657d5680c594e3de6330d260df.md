# Gobuster — modos de busca

---

gobuster <modo> -u <alvo> -w <wordlist>

- dir — acha diretórios/arquivos escondidos num site (ex: /admin, /backup.zip). Usar quando porta 80/443 tá aberta. O mais usado.
- vhost — acha virtual hosts diferentes no mesmo servidor/IP
- dns — acha subdomínios (ex: [admin.site.com](http://admin.site.com/))
- s3 — acha buckets AWS S3 expostos
- gcs — acha buckets Google Cloud Storage expostos (igual s3, mas da Google)
- fuzz — modo genérico, marca FUZZ na URL/parâmetro e testa a wordlist ali. Mais flexível

Regra prática: porta web aberta → começar com dir.

---