# TP Helm charts - 26/01/2023
# TP1 - Utilisation de Helm

#### Consignes : 

+ Installer Helm en utilisant la documentation : https://helm.sh/docs/intro/install/

Pour l'installation de Helm, exécutez les commandes suivantes :

```shell
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```
Téléchargement du fichier : https://get.helm.sh/helm-v3.11.0-linux-amd64.tar.gz
Et on extrait le fichier de notre fichier source.

+ Déployer un Nextcloud en utilisant la chart helm officielle

+ Installation du Chart NextCloud avec les commandes suivantes : 

```
helm repo add nextcloud https://nextcloud.github.io/helm/
helm install my-release nextcloud/nextcloud
```

Pour voir si on accède bien à NextCloud :


1. Depuis votre VM : 

```
kubectl port-forward service/my-release-nextcloud 8080:8080
```

Sur votre VM, saisissez l'IP localhost et vous arrivez sur la page web de NextCloud.

![](https://i.imgur.com/Rvqt5n3.png)

2. Depuis votre machine local :

```
kubectl port-forward service/my-release-nextcloud --address <IP de votre VM> 8080:8080
kubectl port-forward service/my-release-nextcloud --address 172.21.20.121 8080:8080
```

Dans le navigateur en local, saisissez l'IP de votre VM.
Une fois sur la page NextCloud, vous allez avoir une erreur :

![](https://i.imgur.com/qojavNw.png)

+ Créer un fichier `values.yaml` pour augmenter le nombre de replicas.
  Deployer la mise à jour

Création d'un fichier `values.yaml` : 

```
# Number of replicas to be deployed
replicaCount: 3
```

Ce fichier a pour but de déployer 3 Pods au lieu de 1 par défaut de notre NextCloud.

Pour la 2nd partie de la question "Deployer la mise à jour" :
    
1. Si vous voulez, vous pouvez supprimer votre premier chart NextCloud et refaire une installation avec votre fichier `values.yaml` avec les commandes suivantes :

```
helm delete my-release
helm install my-release -f values.yaml nextcloud/nextcloud
```

2. Sinon l'autre option possible est de directement faire une mise à jour de votre chart précédement créé avec la commande suivante :

```
helm upgrade my-release -f values.yaml nextcloud/nextcloud
```

Voici le résultat que l'on obtient : 

![](https://i.imgur.com/vK3XjcL.png)

![](https://i.imgur.com/hy51wLF.png)

![](https://i.imgur.com/xmHNJVx.png)


# TP 2 - Créer sa première chart

#### Consignes : 

+ La chart devra déployer un serveur nginx

Afin de créé notre propre chart il faut exécuter la commande suivante :

```
helm create nginx
tree nginx/
```

La commande `tree` va permettre de lister les différents fichiers de notre chart que l'on vient de créer précédement. Il permet de nous fournir tous les fichiers nécessaires à l'établissement d'un chart complet.

![](https://i.imgur.com/f83cFiZ.png)

Dans le fichier `nginx` racine, on va ajouter 2 nouveaux fichiers values-prod ou values-dev qui vont servir pour réaliser 2 nouvelles installations de charts.

+ Les déploiements, services, ingress, … seront donc créés par cette chart.
  L’image, tag, ports, type de service doivent être modifiables à l’aide d’un fichier values.yaml.
+ **2 fichiers** values-(prod / dev).yaml devront être créés.
  Le fichier prod déploiera un serveur nginx en version 1.22 avec comme domaine srv-prod.test dans le namespace **production**.
  Le fichier dev déploiera un serveur nginx en version 1.23 avec comme domaine srv-dev.test dans le namespace **development**.
  
Création de 2 nouveaux fichiers : 

+ **values-prod.yaml**

```
image:
  repository: nginx
  tag: "1.22"

ingress:
  enabled: true
  hosts:
    - host: srv-prod.test
      paths:
        - path: /
          pathType: ImplementationSpecific
```

+ **values-dev.yaml**

```
image:
  repository: nginx
  tag: "1.23"

ingress:
  enabled: true
  hosts:
    - host: srv-dev.test
      paths:
        - path: /
          pathType: ImplementationSpecific
```

Avant d'installer nos charts, il faut faire des modifications dans le fichier host de la VM dans le chemin suivant `/etc/hosts` : 

![](https://i.imgur.com/weM7yiP.png)

Sur la VM, exécutez les commandes suivantes :

```
helm install -f values-prod.yaml nginx-prod nginx --namespace production --create-namespace
helm install -f values-dev.yaml nginx-dev nginx --namespace development --create-namespace
```
Les commandes pour vérifier sont `kubectl get ing -A` et/ou `kubectl get svc -A`

![](https://i.imgur.com/l67P9Ye.png)

![](https://i.imgur.com/BKFdIAE.png)

Pour supprimer un service, il faut utiliser la commande `kubectl delete namespace <namespace>`.

On peut vérifier avec la commande `curl` si cela fonctionne et qu'on peut arriver sur la page web nginx :

![](https://i.imgur.com/bUkOqd6.png)

![](https://i.imgur.com/QudIx4G.png)

Sur la VM, saisissez le nom de domaine que vous avez décidé d'utiliser dans votre navigateur. Une fois fait, vous pourrez voir les pages nginx :

![](https://i.imgur.com/k9i7Lpb.png)

**Il est interdit de réutiliser une chart existante.**

# TP 2 SUITE - Configmap

#### Consignes : 

+ Reprenons le TP corrigé ce matin et celui fait précédemment.
+ Sur chaque fichier values.yaml, ajoutez un bloc permettant d’insérer le contenu de votre page  html. 
  Les pages doivent être différentes en prod et en dev.
  Cette variable devra être importée dans une configmap, qui sera elle même montée dans le pod nginx.



+ Déployer la mise à jour.

