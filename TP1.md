# TP 1 - Docker
Initialiser un nouveau repository git qui vous permettra de sauvegarder les fichiers créés pendant le TP. Vous enverrez un zip du repository à la fin du TP avec vos
réponses aux questions / exécutions et résultats sur la console dans des fichiers
texte (Markdown par exemple) par e-mail.

Utilisez git progressivement ! (Ne pas faire qu’un seul commit à la fin)

<br>

## Exécuter un serveur web (apache, nginx, …) dans un conteneur docker

-  Récupérer l’image sur le Docker Hub

```
docker pull nginx
```

-  Vérifier que cette image est présente en local

```
docker image ls
```

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

-  Démarrer un conteneur et servir la page html créée précédemment à l’aide
d’un volume (option -v de docker run)

```
docker run -d -p 80:80 --name nginx-server -v /home/pierre/Documents/Conteneur/index.html:/usr/share/nginx/html/index.html nginx
```

-  Supprimer le conteneur précédent et arriver au même résultat que
précédemment à l’aide de la commande docker cp

```
docker stop nginx-server
docker rm nginx-server
docker run -d --name server-nginx -p 80:80 nginx
docker cp /home/pierre/Documents/Conteneur/index.html server-nginx:/usr/share/nginx/html/index.html
```

<br>

## Builder une image
- A l’aide d’un Dockerfile, créer une image (commande docker build)
- Exécuter cette nouvelle image de manière à servir la page html (commande
docker run)
- Quelles différences observez-vous entre les procédures 5. et 6. ? Avantages
et inconvénients de l’une et de l’autre méthode ? (Mettre en relation ce qui est
observé avec ce qui a été présenté pendant le cours)

<br>

## Utiliser une base de données dans un conteneur docker
- Récupérer les images mysql:5.7 et phpmyadmin/phpmyadmin depuis le
Docker Hub
- Exécuter deux conteneurs à partir des images et ajouter une table ainsi que
quelques enregistrements dans la base de données à l’aide de phpmyadmin

<br>

## Faire la même chose que précédemment en utilisant un fichier
docker-compose.yml
a. Qu’apporte le fichier docker-compose par rapport aux commandes docker run
? Pourquoi est-il intéressant ? (cf. ce qui a été présenté pendant le cours)
b. Quel moyen permet de configurer (premier utilisateur, première base de
données, mot de passe root, …) facilement le conteneur mysql au lancement ?

<br>

## Observation de l’isolation réseau entre 3 conteneurs
a. A l’aide de docker-compose et de l’image praqma/network-multitool
disponible sur le Docker Hub créer 3 services (web, app et db) et 2 réseaux
(frontend et backend).
Les services web et db ne devront pas pouvoir effectuer de ping de l’un vers
l’autre
b. Quelles lignes du résultat de la commande docker inspect justifient ce
comportement ?
c. Dans quelle situation réelles (avec quelles images) pourrait-on avoir cette
configuration réseau ? Dans quel but ?