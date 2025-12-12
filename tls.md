TLS Auto-signé avec Traefik
1. Commande OpenSSL utilisée

Pour générer un certificat auto-signé pour le domaine app.localhost :

openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout certs/app.localhost.key \
  -out certs/app.localhost.crt \
  -subj "/CN=app.localhost"


Explications :

req -x509 : création d’un certificat auto-signé X.509

-nodes : pas de mot de passe sur la clé privée

-days 365 : validité du certificat d’un an

-newkey rsa:2048 : génération d’une clé RSA 2048 bits

-keyout et -out : fichiers générés

-subj "/CN=app.localhost" : nom commun correspondant au domaine

2. Emplacement des certificats

Dans le projet Docker :

/tp-networks/certs/
├─ app.localhost.crt  # certificat public
└─ app.localhost.key  # clé privée


Ces fichiers seront montés dans le conteneur Traefik via le docker-compose.yml :

volumes:
  - ./certs:/certs

3. Fonctionnement du router TLS avec Traefik

Dans Traefik, on définit :

Entrypoints : points d’entrée du trafic HTTP et HTTPS

entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"


Certificat custom : le certificat auto-signé est utilisé via un tls configuration dans le router :

tls:
  certResolver: selfsigned
  domains:
    - main: "app.localhost"


Router Traefik :

labels:
  - "traefik.enable=true"
  - "traefik.http.routers.app.rule=Host(`app.localhost`)"
  - "traefik.http.routers.app.entrypoints=websecure"
  - "traefik.http.routers.app.tls=true"
  - "traefik.http.routers.app.tls.certresolver=selfsigned"


Redirection HTTP → HTTPS :

- "traefik.http.routers.app-http.rule=Host(`app.localhost`)"
- "traefik.http.routers.app-http.entrypoints=web"
- "traefik.http.routers.app-http.middlewares=redirect-to-https"
- "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"

4. Vérification

Accéder à : https://app.localhost

Le navigateur indique un certificat auto-signé (avertissement normal)

HTTP (http://app.localhost) redirige automatiquement vers HTTPS
