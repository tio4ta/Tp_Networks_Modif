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

<img width="1098" height="872" alt="image" src="https://github.com/user-attachments/assets/a2ff8057-a4a1-4e18-b937-fa16c30fbcae" />
<img width="427" height="283" alt="image" src="https://github.com/user-attachments/assets/6174caf9-8bbb-494f-937c-48322f893662" />

  
La DB n’a aucun port publié, ce qui empêche l’accès direct depuis l’hôte.

L’application app n’expose pas de port non plus. Seul le proxy Nginx est accessible via le port 80.

3️. Service App (Flask)
Le fichier app.py existait déjà. Il contient deux routes principales :

/ qui retourne "Hello from app!"

/health qui teste la connexion à la base et renvoie un JSON indiquant si elle est accessible :

{"status":"ok","db":"reachable"}

J’ai créé un Dockerfile pour l’application dans le dossier app/ :

<img width="1067" height="224" alt="image" src="https://github.com/user-attachments/assets/20f4e7ed-5650-4d69-b93e-93990bed920d" />


Cette image installe Flask et PyMySQL, puis lance l’application Flask sur le port 5000 à l’intérieur du conteneur.

4️. Service Proxy (Nginx)
Pour le proxy, j’ai créé un Dockerfile dans le dossier proxy/ :

<img width="1066" height="235" alt="image" src="https://github.com/user-attachments/assets/c72a2e06-2a9c-4eb2-b373-e4b47792d533" />


Et j’ai écrit un fichier nginx.conf pour rediriger tout le trafic vers l’application Flask :

<img width="1042" height="291" alt="image" src="https://github.com/user-attachments/assets/20614b8f-b68d-44c2-9e3e-5a25cfa8c4b1" />


Grâce à cette configuration, quand on accède à http://localhost:80, on tombe sur "Hello from app!" au lieu de la page par défaut de Nginx.

<img width="330" height="132" alt="image" src="https://github.com/user-attachments/assets/5bb5b0b8-3d3c-4db6-961f-f1a006c0dddc" />

Le proxy agit comme point d’entrée unique vers l’application, respectant la consigne de sécurité.

5️. Lancement de l’infrastructure

Pour démarrer tous les services :

docker compose up -d --build

Vérification de l’application :

curl http://localhost/

**Réponse : Hello from app!**
<img width="330" height="132" alt="image" src="https://github.com/user-attachments/assets/a349a906-3273-428e-8f19-50291a8da3dc" />



curl http://localhost/health

**Réponse : {"status":"ok","db":"reachable"}**
<img width="384" height="145" alt="image" src="https://github.com/user-attachments/assets/0fdf30bd-da8e-4e0a-9ade-c0958a0a309c" />


Vérification que la base n’est pas accessible directement depuis l’hôte :

mysql -h 127.0.0.1 -P 3306 -u appuser -p

**Doit échouer car DB non exposée**

6️. Résultat attendu

L’application Flask est accessible via le proxy Nginx.

La route /health confirme que l’application peut atteindre la base de données.

La base de données n’est accessible que depuis l’application et non depuis l’hôte.


**Le push sur le docker hub :**

<img width="1212" height="646" alt="image" src="https://github.com/user-attachments/assets/e943ff82-5a13-40f4-8b12-911255b74e0d" />


<img width="1875" height="612" alt="image" src="https://github.com/user-attachments/assets/2ce8d8ee-b61e-4555-b1cd-a26f48ee25df" />

