# Yopass

Instance auto-hébergée de [Yopass](https://github.com/jhaals/yopass) pour partager des secrets chiffrés, à usage unique et à durée limitée.

```
Internet -> Cloudflare -> cloudflared -> Traefik -> yopass -> memcached
```

- L'entrée publique se fait uniquement par le tunnel Cloudflare, qui termine le TLS.
- Seul le port `9180` est publié, sur l'IP LAN `METRICS_BIND_IP` : il sert `/metrics`, pour le Prometheus interne.
- Traefik ne route que les conteneurs portant le label `yopass.expose=true`.
- memcached n'est joignable que depuis le réseau interne. Les secrets ne sont pas persistés et disparaissent au redémarrage.
- Le chiffrement se fait dans le navigateur. La clé reste dans le fragment `#` de l'URL et n'est jamais envoyée au serveur.

## Prérequis

- Docker et Docker Compose.
- Un tunnel Cloudflare et son token.
- Une route publique sur ce tunnel : `<domaine>` vers `http://traefik:80`.

## Configuration

```sh
cp .env.example .env
```

| Variable | Rôle |
|---|---|
| `YOPASS_VERSION`, `MEMCACHED_VERSION`, `TRAEFIK_VERSION`, `CLOUDFLARED_VERSION` | Tags des images |
| `YOPASS_DOMAIN` | Domaine public, identique à la route du tunnel |
| `CLOUDFLARE_TUNNEL_TOKEN` | Token du tunnel |
| `METRICS_BIND_IP` | IP LAN du serveur sur laquelle publier les métriques |

Si une variable manque, la stack refuse de démarrer et affiche son nom.

## Démarrage

```sh
docker compose up -d
docker compose ps
```

`cloudflared` attend que Traefik soit en bonne santé avant de démarrer.

## Mise à jour

Modifier le tag voulu dans `.env`, puis :

```sh
docker compose pull
docker compose up -d
```
