# nginxpm-portainer

Stack Docker Compose per una VPS: [Nginx Proxy Manager](https://nginxproxymanager.com/)
come reverse proxy pubblico (con gestione certificati Let's Encrypt) e
[Portainer](https://www.portainer.io/) come UI di gestione Docker.

I due servizi condividono una rete Docker esterna, `reverse-proxy`, a cui si
possono agganciare anche gli stack di altre applicazioni (deployate da altre
repo) per essere esposte tramite Nginx Proxy Manager senza pubblicare le
loro porte direttamente sull'host.

## Prerequisiti

- Docker Engine + Docker Compose plugin sulla VPS.
- Porte **80** e **443** libere e raggiungibili da internet (traffico HTTP/HTTPS pubblico).
- Porta **81** raggiungibile (almeno dalla tua rete) per l'admin UI di Nginx Proxy Manager.
- Porte **9000**/**9443** raggiungibili per la UI di Portainer.

## Avvio

```bash
git clone <url-di-questa-repo>
cd nginxpm-portainer
docker compose up -d
```

Questo crea anche la rete Docker esterna `reverse-proxy`, usata per collegare
altri stack (es. le app da esporre) a Nginx Proxy Manager senza pubblicarne le
porte sull'host — nel loro `docker-compose.yml` basta dichiararla come rete
esterna:

```yaml
networks:
  reverse-proxy:
    external: true
```

e in Nginx Proxy Manager creare un Proxy Host puntando al nome del container
dell'app e alla sua porta interna, con l'opzione "Docker Network" attiva.

## Primo accesso

- **Nginx Proxy Manager**: `http://<ip-vps>:81`
  Credenziali di default: `admin@example.com` / `changeme`.
  **Vanno cambiate subito al primo login.**
- **Portainer**: `https://<ip-vps>:9443`
  Alla primissima apertura chiede di creare l'utente amministratore.

## Persistenza dati

I dati applicativi sono in bind mount nella cartella del progetto (esclusi da
git via `.gitignore`, contengono certificati e stato interno):

- `./npm/data` — configurazione di Nginx Proxy Manager (proxy host, utenti, ecc.)
- `./npm/letsencrypt` — certificati TLS emessi
- `./portainer/data` — configurazione e stato di Portainer

Fanne un backup periodico: sono l'unico stato non ricostruibile dai file di
questa repo.

## Sicurezza

Repo pubblica: non committare mai le cartelle `npm/` e `portainer/` (già in
`.gitignore`) né altri file con credenziali, token o certificati. Portainer
qui monta `/var/run/docker.sock`, quindi ha accesso completo al Docker
dell'host: proteggi l'accesso alla sua UI con una password robusta e limita
l'esposizione della porta 9000/9443 a IP fidati quando possibile (es. tramite
firewall della VPS).
