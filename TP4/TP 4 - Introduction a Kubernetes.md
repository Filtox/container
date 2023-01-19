# TP4 - Introduction à Kubernetes

## 1. Installer Minikube et Kubectl

Suivre cette documentation : https://minikube.sigs.k8s.io/docs/start/

- Installation sur Linux

Voici la procédure à suivre : 

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Une fois l'installation terminé, on peut démarrer Minikube : 

```
minikube start
```

Afin de vérifier que le cluster Kube fonctionne, on peut éxécuter cette commande :

```
# Alternative, Minikube peut télécharger la version de Kubectl
minikube kubectl -- get po -A
```

On peut éxécuter cette commande afin  d'accéder à l'administration web de Kube :

```
minikube dashboard
```

Une page web va s'ouvrir à la fin du chargement de la commande :

![](https://i.imgur.com/Jq6Q4Hq.png)

## 2. Pod Nginx

### a. Héberger un premier Pod Nginx

<br>

Il est nécessaire de générer un fichier de configuration nommé `nginx-pod.yaml`.

Voici son contenu : 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

Il faut ensuite créer et éxécuter le Pod en exécutant la commande `kubectl apply -f nginx-pod.yaml`.

On peut vérifier son fonctionnement avec la commande `kubectl get pod` ou directement dans le navigateur avec le Dashboard.

Si le résultat affiche la Pod prête 1/1 et status en cours, c'est que la Pod est fonctionnel.

![](https://i.imgur.com/g2ZKAxc.png)

### b. A l’aide de la commande kubectl port-forward et d’un navigateur accéder à la page par défaut de votre Pod Nginx.

Pour accéder à la page par défaut du Pod Nginx, il faut utiliser la commande `kubectl port-forward nginx-pod 8080:80`.

![](https://i.imgur.com/CdYKhrA.png)

![](https://i.imgur.com/Jev4nBS.png)

## 3. Connexion entre plusieurs Pods

### a. A l’image du TP 1 sur Docker (question 7 et 8), héberger un Pod PHPMyAdmin et MySQL, cette fois-ci en utilisant Minikube

<br>

Il est nécessaire de générer un fichier de configuration nommé `mysql-pod.yaml`.

```
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  labels:
    app: mysql
spec:
  containers:
  - name: mysql-container
    image: mysql:latest
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "root"
    - name: MYSQL_USER
      value: "test"
    - name: MYSQL_PASSWORD
      value: "password"
    ports:
    - containerPort: 3306
```

Il faut aussi générer un fichier de configuration nommé `phpmyadmin-pod.yaml`.

```
apiVersion: v1
kind: Pod
metadata:
  name: phpmyadmin-pod
spec:
  containers:
  - name: phpmyadmin-container
    image: phpmyadmin/phpmyadmin
    env:
    - name: PMA_HOST
      value: mysql-service
    - name: PMA_PORT
      value: "3306"
    ports:
    - containerPort: 80
```

Il faut ensuite créer et éxécuter le Pod MySQL en exécutant la commande `kubectl apply -f mysql-pod.yaml`.

Il faut faire la même manipulation pour créer le Pod PHPMyAdmin en exécutant la commande `kubectl apply -f phpmyadmin-pod.yaml`.

Les Pods sont visible depuis le Dashboard.

![](https://i.imgur.com/9l8vptV.png)

Pour accéder à la page par défaut du Pod PHPMyAdmin, il faut utiliser la commande `kubectl port-forward phpmyadmin-pod 8080:80`.

Dans la barre de recherche du navigateur, il faut se rendre sur l'url `localhost:8080`.


### b. Créer un service associé au Pod MySQL

<br>

Il est nécessaire de générer un fichier de configuration nommé `mysql-service.yaml`.

```
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
```

Il faut ensuite créer et éxécuter le service MySQL en exécutant la commande `kubectl apply -f mysql-service.yaml`.

### c. Connecter phpmyadmin avec le Service MySQL et vérifier que vous pouvez administrer cette base de données

### d. Avec la commande kubectl-port forward, vérifier que phpmyadmin arrive à contacter et administrer votre base de données mysql

<br>

Il faut effectuer un port forwarding à nouveau afin de se connecter à la page web de phpmyadmin avec `kubectl port-forward phpmyadmin-pod 8080:80`.

![](https://i.imgur.com/wX5sPOj.png)

![](https://i.imgur.com/T4CE2LD.png)

#### e. Ajouter un Ingress pour accéder à phpmyadmin sans utiliser la commande kubectl port-forward

<br>

Je n'ai pas réussi à faire cette partie.