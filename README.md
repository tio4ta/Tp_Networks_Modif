TP Docker

Le but de ce TP était de mettre en place une petite infrastructure Docker composée de trois services : une base de données MariaDB (`db`), une application Flask (`app`) qui interroge la base via PyMySQL, et un serveur Nginx (`proxy`) jouant le rôle de reverse proxy.  

Pour respecter les consignes, j’ai organisé les services sur deux réseaux Docker distincts. Le réseau `backend_net` contient `app` et `db` afin qu’ils puissent communiquer entre eux, tandis que le réseau `frontend_net` permet au proxy de servir l’application vers l’extérieur tout en restant connecté à `backend_net` pour accéder à `app`. Cela permet de sécuriser la base de données, qui n’est pas exposée directement à l’hôte.

---

1️. Étapes réalisées

Cloner le dépôt initial

git clone https://github.com/Sylvain6/tp-networks
cd tp-networks

2️. Fichier Docker Compose

J’ai créé le fichier docker-compose.yml pour définir les trois services, les volumes et les réseaux. Voici son contenu :

version: "3.9"

services:

  db:
    image: mariadb:latest
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: tpnetworks
      MYSQL_DATABASE: appdb
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppass
    networks:
      - backend_net
    volumes:
      - db_data:/var/lib/mysql
    ports: []

  app:
    build: ./app
    container_name: app
    environment:
      DB_HOST: db
      DB_USER: appuser
      DB_PASSWORD: apppass
      DB_NAME: appdb
    networks:
      - backend_net
    depends_on:
      - db

  proxy:
    build: ./proxy
    container_name: proxy
    ports:
      - "80:80"
    networks:
      - frontend_net
      - backend_net
    depends_on:
      - app

volumes:
  db_data:

networks:
  backend_net:
  frontend_net:
  
La DB n’a aucun port publié, ce qui empêche l’accès direct depuis l’hôte.

L’application app n’expose pas de port non plus. Seul le proxy Nginx est accessible via le port 80.

3️. Service App (Flask)
Le fichier app.py existait déjà. Il contient deux routes principales :

/ qui retourne "Hello from app!"

/health qui teste la connexion à la base et renvoie un JSON indiquant si elle est accessible :

{"status":"ok","db":"reachable"}

J’ai créé un Dockerfile pour l’application dans le dossier app/ :

FROM python:3.11-slim

WORKDIR /app
COPY app.py /app
RUN pip install flask pymysql

CMD ["python", "app.py"]

Cette image installe Flask et PyMySQL, puis lance l’application Flask sur le port 5000 à l’intérieur du conteneur.

4️. Service Proxy (Nginx)
Pour le proxy, j’ai créé un Dockerfile dans le dossier proxy/ :

FROM nginx:latest

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf

Et j’ai écrit un fichier nginx.conf pour rediriger tout le trafic vers l’application Flask :

server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://app:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
Grâce à cette configuration, quand on accède à http://localhost:80, on tombe sur "Hello from app!" au lieu de la page par défaut de Nginx.

Le proxy agit comme point d’entrée unique vers l’application, respectant la consigne de sécurité.

5️. Lancement de l’infrastructure

Pour démarrer tous les services :

docker compose up -d --build
Vérification de l’application :

curl http://localhost/
# Réponse : Hello from app!

curl http://localhost/health
# Réponse : {"status":"ok","db":"reachable"}
Vérification que la base n’est pas accessible directement depuis l’hôte :

bash
Copier le code
mysql -h 127.0.0.1 -P 3306 -u appuser -p

# Doit échouer car DB non exposée

6️. Résultat attendu
L’application Flask est accessible via le proxy Nginx.

La route /health confirme que l’application peut atteindre la base de données.

La base de données n’est accessible que depuis l’application et non depuis l’hôte.
