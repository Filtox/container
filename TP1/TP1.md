# TP 1 - Docker

![](https://i.imgur.com/TflEfjv.png)

<br>

## Exécuter un serveur web (apache, nginx, …) dans un conteneur docker

-  Récupérer l’image sur le Docker Hub

```
docker pull nginx
```

![](https://i.imgur.com/KtG5Ajc.png)


-  Vérifier que cette image est présente en local

```
docker image ls
```

![](https://i.imgur.com/lrwuDuw.png)


-  Créer un fichier index.html simple

```
echo ' <!DOCTYPE html>
<html>
 <head>
    <title> Fichier HTML simple </title>
 </head>
<body>
 <h1>Fichier HTML simple</h1>
</body>
</html> ' >> index.html
```

![](https://i.imgur.com/R5ImkQW.png)


-  Démarrer un conteneur et servir la page html créée précédemment à l’aide d’un volume (option -v de docker run)

```
docker run -d -p 80:80 --name nginx-server -v /home/pierre/Documents/Conteneur/index.html:/usr/share/nginx/html/index.html nginx
```

![](https://i.imgur.com/HKaGcKR.png)

![](https://i.imgur.com/f0L1NCc.png)


-  Supprimer le conteneur précédent et arriver au même résultat que précédemment à l’aide de la commande docker cp

```
docker stop nginx-server
docker rm nginx-server
docker run -d --name server-nginx -p 80:80 nginx
docker cp /home/pierre/Documents/Conteneur/index.html server-nginx:/usr/share/nginx/html/index.html
```

![](https://i.imgur.com/on3zlEf.png)

![](https://i.imgur.com/a96n3GF.png)


<br>

## Builder une image
- A l’aide d’un Dockerfile, créer une image (commande docker build)

```
echo 'FROM nginx:latest
COPY index.html /usr/share/nginx/html' >> Dockerfile
docker build -t nginximage .
```

![](https://i.imgur.com/szUDinW.png)


- Exécuter cette nouvelle image de manière à servir la page html (commande docker run)

```
docker run -p 80:80 -d nginximage
```

![](https://i.imgur.com/GHWzLcO.png)


- Quelles différences observez-vous entre les procédures 5. et 6. ? Avantages et inconvénients de l’une et de l’autre méthode ? (Mettre en relation ce qui est observé avec ce qui a été présenté pendant le cours)

Docker peut créer des images automatiquement en lisant les instructions d'un Dockerfile. Un Dockerfile est un document texte qui contient toutes les commandes qu'un utilisateur peut appeler sur la ligne de commande pour assembler une image. Cette page décrit les commandes que vous pouvez utiliser dans un Dockerfile.

Docker cp, en revanche, est une commande utilisée pour copier des fichiers ou des répertoires entre l'hôte et un conteneur docker.

La différence la plus notable est le fait que la création d'une image se fait via un fichier pour le docker compose et via une commande pour le docker cp ou run -v.

<br>

## Utiliser une base de données dans un conteneur docker
- Récupérer les images mysql:5.7 et phpmyadmin/phpmyadmin depuis le Docker Hub

```
docker pull mysql:5.7
docker pull phpmyadmin/phpmyadmin
```

![](https://i.imgur.com/qvRMebN.png)


- Exécuter deux conteneurs à partir des images et ajouter une table ainsi que quelques enregistrements dans la base de données à l’aide de phpmyadmin

```
docker run --name mysql -e MYSQL_ROOT_PASSWORD=pwd -d mysql:5.7
docker run --name phpmyadmin --link mysql:db -d -p 8080:80 phpmyadmin/phpmyadmin
```

![](https://i.imgur.com/t7tcxVK.png)

<br>

## Faire la même chose que précédemment en utilisant un fichier docker-compose.yml
- Qu’apporte le fichier docker-compose par rapport aux commandes docker run ? Pourquoi est-il intéressant ? (cf. ce qui a été présenté pendant le cours)

```
version: '3'
services:
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: password
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    links:
      - mysql:db
    ports:
      - 8080:80
```

Le fichier docker-compose.yml permet de définir les conteneurs à exécuter, leurs configurations et leurs interactions. Cela peut être plus pratique que d'utiliser plusieurs commandes docker run car on peut définir toutes les configurations dans un seul fichier.

- Quel moyen permet de configurer (premier utilisateur, première base de données, mot de passe root, …) facilement le conteneur mysql au lancement ?

```
version: "3.9"
services:
  mysql:
    image: "mysql:5.7"
    environment:
      - MYSQL_USER=pierre
      - MYSQL_PASSWORD=pwd
      - MYSQL_DATABASE=mysql
      - MYSQL_ROOT_PASSWORD=root
```

<br>

## Observation de l’isolation réseau entre 3 conteneurs
- A l’aide de docker-compose et de l’image praqma/network-multitool disponible sur le Docker Hub créer 3 services (web, app et db) et 2 réseaux (frontend et backend).
Les services web et db ne devront pas pouvoir effectuer de ping de l’un vers
l’autre

```
version: '3'

services:
  web:
    image: praqma/network-multitool
    networks:
      - frontend
  app:
    image: praqma/network-multitool
    networks:
      - frontend
      - backend
  db:
    image: praqma/network-multitool
    networks:
      - backend

networks:
  frontend:
  backend:
```

- Quelles lignes du résultat de la commande docker inspect justifient ce comportement ?

Pour vérifier que les services web et db ne peuvent pas effectuer de ping l'un vers l'autre, vous pouvez utiliser la commande docker network inspect pour afficher des informations sur les réseaux et les services connectés à chacun d'eux. Le résultat de cette commande inclura des informations sur les adresses IP attribuées aux services et aux réseaux, ainsi que sur les réseaux auxquels chaque service est connecté.
Par exemple, si vous exécutez la commande suivante :

docker network inspect frontend
Le résultat inclura des informations sur les services web et app connectés au réseau frontend, ainsi que sur leurs adresses IP respectives. Si vous exécutez la commande suivante :

docker network inspect backend
Le résultat inclura des informations sur le service db connecté au réseau backend, ainsi que sur son adresse IP.

Les lignes du résultat de ces commandes qui justifient le fait que les services web et db ne peuvent pas effectuer de ping l'un vers l'autre sont les lignes qui indiquent que les services sont connectés à des réseaux différents (frontend et backend respectivement) et ont donc des adresses IP différentes.

- Dans quelle situation réelles (avec quelles images) pourrait-on avoir cette configuration réseau ? Dans quel but ?

Il est possible de créer une configuration réseau similaire en utilisant des images Docker différentes pour les services web, app et db. Cette configuration pourrait être utilisée dans un environnement d'application où l'application est divisée en différents services qui communiquent entre eux via des réseaux.

Par exemple, si l'application comporte une interface utilisateur (web), une couche d'application (app) qui gère les logiques métier et les communications avec la base de données (db), on pourrait utiliser cette configuration réseau pour séparer l'interface utilisateur et la couche d'application de la base de données. Cela permet de sécuriser les communications entre ces différents services en limitant les interactions entre eux uniquement aux échanges nécessaires pour l'application.
