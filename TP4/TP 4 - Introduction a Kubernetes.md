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

:warning: Avant de démarrer Minikube, vous allez surement rencontrer une erreur. Si c'est le cas, saisissez la commande suivante avant de ré-essayer. :warning: 

```
sudo usermod -aG docker $USER && newgrp docker
```

Commande de démarrage de Minikube : 

```
minikube start
```

Le téléchargement du pré-téléchargement de Kubernetes est assez long.

Maintenant, on va intérragir avec notre cluster Kube : 

```
# Faire cette commande si Kubectl est installé
kubectl get po -A
```

```
# Alternative, Minikube peut télécharger la version de Kubectl
minikube kubectl -- get po -A
```

Vérification final en accédant au Dashboard : 

```
minikube dashboard
```

Lors de l'exécution de la commande, on vous demande de lancer la commande suivante avant de refaire la commande précédente (option) : 
*Afin d'activer certains fonctionnalités du Dashboard*

```
minikube addons enable metrics-server
```

De notre côté, on rencontre une erreur si on tente de lancer la commande directement sur un logiciel permettant d'exécuter des commandes sur notre machine hôte (comme MobaXtern).
Si on tente la commande directement sur la machine hôte, la navigateur s'ouvre automatiquement.

![](https://i.imgur.com/xMRWZme.png)

Par contre, si on prend l'url et qu'on le saisie dans le navigateur, on arrive sur le Dashboard Web de Kube : 

![](https://i.imgur.com/Jq6Q4Hq.png)

## 2. Pod Nginx

#### a. Héberger un premier Pod Nginx

Dans un premier temps, il faut générer un conteneur pour Minikube avec la commande `minikube start`.

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

Puis créer le Pod en exécutant la commande `kubectl apply -f nginx-pod.yaml` ou `kubectl create -f nginx-pod.yaml`.

![](https://i.imgur.com/gDErwMv.png)

On peut vérifier son fonctionnement avec la commande `kubectl get pod`.

Si le résultat affiche la Pod prête 1/1 et status en cours, c'est que la Pod est fonctionnel.

![](https://i.imgur.com/g2ZKAxc.png)

#### b. A l’aide de la commande kubectl port-forward et d’un navigateur accéder à la page par défaut de votre Pod Nginx.

Pour accéder à la page par défaut du Pod Nginx, il faut utiliser la commande `kubectl port-forward nginx-pod 8080:80`.

Une fois sur la page web de Nginx, une 3ème ligne apparait sur votre terminal.

![](https://i.imgur.com/CdYKhrA.png)

![](https://i.imgur.com/Jev4nBS.png)

A la fin pour arrêter et supprimer un Pod en cours, il faut utiliser la commande `kubectl delete pod <nom du Pod>`.

## 3. Connexion entre plusieurs Pods

#### a. A l’image du TP 1 sur Docker (question 7 et 8), héberger un Pod PHPMyAdmin et MySQL, cette fois-ci en utilisant Minikube

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

Puis créer le Pod MySQL en exécutant la commande `kubectl apply -f mysql-pod.yaml` ou `kubectl create -f mysql-pod.yaml`.
Faire la même manipulation pour créer le Pod PHPMyAdmin en exécutant la commande `kubectl apply -f phpmyadmin-pod.yaml` ou `kubectl create -f phpmyadmin-pod.yaml`.

![](https://i.imgur.com/pnQb0jf.png)

On peut vérifier la création des Pods avec la commande `kubectl get pod`.

![](https://i.imgur.com/hAOfbfi.png)

Si lors de l'exécution de la commande précédente un des Pod n'est pas prêt, il est conseillé de refaire la commande quelques secondes après. Il apparaîtra prêt, c'est le temps que le Pod se créé correctement.

On retrouve aussi nos Pods directement sur notre Dashboard.

![](https://i.imgur.com/9l8vptV.png)

Pour accéder à la page par défaut du Pod PHPMyAdmin, il faut utiliser la commande `kubectl port-forward phpmyadmin-pod 8080:80`.

Dans la barre de recherche du navigateur, saisisser `localhost:8080`.

Sauf qu'il y a une erreur, il manque un service nécessaire pour pouvoir accéder à l'interface de PHPMyAdmin.

#### b. Créer un service associé au Pod MySQL

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

Puis créer le service MySQL en exécutant la commande `kubectl apply -f mysql-service.yaml` ou `kubectl create -f mysql-service.yaml`.

#### c. Connecter phpmyadmin avec le Service MySQL et vérifier que vous pouvez administrer cette base de données

![](https://i.imgur.com/l3JC3XK.png)

Comme vous le constatez sur la capture précédente sans le service MySQL supplémentaire, il est impossible de se connecter sur PHPMyAdmin.



#### d. Avec la commande kubectl-port forward, vérifier que phpmyadmin arrive à contacter et administrer votre base de données mysql

Il faut effectuer un port forwarding à nouveau afin de se connecter à la page web de phpmyadmin avec `kubectl port-forward phpmyadmin-pod 8080:80`.

![](https://i.imgur.com/wX5sPOj.png)

![](https://i.imgur.com/T4CE2LD.png)

#### e. Ajouter un Ingress pour accéder à phpmyadmin sans utiliser la commande kubectl port-forward

Je n'ai pas réussi à faire cette partie.