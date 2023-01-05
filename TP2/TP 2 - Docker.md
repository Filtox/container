# TP 2 - Docker

![](https://i.imgur.com/TflEfjv.png)

<br>

## Mise en pratique - Docker Application Express JS

<br>

- Compléter le Dockerfile afin de builder correctement l'application contenu dans src/
    - Une option de npm vous permet de n’installer que ce qui est nécessaire. Quelle est cette option ? Quelle bonne pratique Docker permet t-elle derespecter ?

<br>

Afin d'installer uniquement les modules nécessaires à l'application, on peut utiliser dans le Dockerfile une option `production` de npm.

**Dockerfile au départ :**
```
FROM node:12-alpine3.9

...

CMD ["node", "src/index.js"]
```

**Dockerfile après changement :**
```
FROM node:12-alpine3.9

COPY . .

# Permet d'installer les modules que l'on a besoin
RUN npm install --production

CMD ["node", "src/index.js"]
```

Cette option, va permettre d'installer uniquement les modules nécessaires à l'application qui sont listés dans la section `dependencies` du fichier `package.json`.

```
{
  "name": "docker-tp2",
  "version": "1.0.0",
  "description": "",
  "main": "src/index.js",
  "author": "Ynov",
  "dependencies": {
    "express": "^4.16.1",
    "mysql": "2.18.1"
  },
  "devDependencies": {
    "eslint": "^6.3.0",
    "jest": "^24.9.0"
  }
}
```

<br>

- A l'aide de la commande docker build, créer l'image ma_super_app

<br>

Afin de pouvoir créer l'image avec la commande ```docker build```, il faut utiliser une option supplémentaire ```-f``` qui va permettre d'utiliser le fichier Dockerfile mis à notre disposition. Pour cette option, il faut indiquer le chemin menant à notre fichier Dockerfile.
Et l'option ```-t``` permet de donner un nom à notre image.

```
docker build -f Dockerfile -t ma_super_app .
```

Après utilisation de la commande, on obtient notre image ma_super_app . Si l'on veut, on peut démarrer un conteneur à partir de cette image en utilisant la commande ```docker run```

```
docker run ma_super_app
```

<br>

- Compléter le fichier docker-compose.yml afin d'exécuter ma_super_app avec sa base de données

<br>

Afin de pouvoir exécuter une application qui va nécessiter une base de données en utilisant Docker Compose, il va falloir séparer le docker-compose.yml en 2 services (`db` et `app`).

**docker-compose.yml au départ :**
```
version: '3.9'
  services:
    node:
      ...

    mysql:
      ...

```

**docker-compose.yml après changement :**
```
version: "3.9"
services:
  mysql:
    image: "mysql:5.7"
    environment:
      - MYSQL_USER=valentin
      - MYSQL_PASSWORD=malo
      - MYSQL_DATABASE=mysql
      - MYSQL_ROOT_PASSWORD=root
  app:
    image: "ma_super_app"
    links:
      - mysql:db
    ports:
      - "3000:3000"
    depends_on:
      - mysql:db
```

Une fois le fichier prêt, pour l'exécuter, il faut utiliser la commande `docker-compose up` dans le même répertoire que le fichier. Cela va permettre de créer le conteneur utilisant les 2 services en simultanés.
Pour arrêter notre application, on réutilise la commande ci-dessus mais on remplace `up`  par `down`.