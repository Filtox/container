# TP5 - Kubernetes

### 1. Installer Kind et créer votre premier cluster
Suivre cette documentation : https://kind.sigs.k8s.io/docs/user/quick-start/



### 2. Installer le Nginx ingress Controller
Suivre cette documentation : https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx


### 3. Compléter le schéma suivant avec des objets Kubernetes

![](https://i.imgur.com/GTqvGsN.png)

### 4. Builder et publier (à partir de l’image nginx) sur le DockerHub, une image docker pour chacun des sites web présent sur le schéma précédent. Vous devez avoir 3 images (une par magasin tacos, pizzas et burgers)

Déploiement du cluster kind en exposant le port 80 et 443

```yaml
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

Puis installation de nginx ingress controller avec la commande `kubectl apply -f`.

```shell
https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

Et avec la commande suivante également :

```shell
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

```
pod/ingress-nginx-controller-6bccc5966-j9frl condition met
```

Une fois l’ingress Nginx controller installer nous pouvons commencer la construction et la publications des images docker.

D’abord nous allons créer un dockerfile avec un index.html afin d’afficher “Tacos”, “Burgers” ou “Pizzas”

**Dockerfile :**

```dockerfile
FROM nginx:1.16.1

COPY index.html /usr/share/nginx/html/

CMD ["nginx", "-g", "daemon off;"]
```

**Index.html :**

```htmlembedded
<html>
  <body>
    <h1>Pizzas</h1>
  </body>
</html>
```

Ensuite on va construire notre image et la publir sur docker hub.

```
#Connexion à son compte docker
docker login -u username

#Build l'image docker
docker build -t pizzas-image .

#Tag l'image docker
docker tag pizzas-image:latest pierredasilva/pizzas-image:latest

#Publication de l'image docker
docker push pierredasilva/pizzas-image
```

Il faut répéter cette procédure pour chaque image que l’on souhaite publier.

![](https://i.imgur.com/ccanrEr.png)

### 5. Ecrire les fichiers yaml vous permettant de déployer sur votre cluster kind installé en local les composants décrits sur le schéma de la question 3 et les images crées à la question 4

Exemple de fichier yaml pour déployer les pizzas, tacos et burgers :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: pizzas-services
spec:
  selector:
    app: pizzas
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pizzas-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pizzas
  template:
    metadata:
      labels:
        app: pizzas
    spec:
      containers:
      - name: pizzas
        image: pierredasilva/pizzas-image
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: tacos-services
spec:
  selector:
    app: tacos
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tacos-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tacos
  template:
    metadata:
labels:
        app: tacos
    spec:
      containers:
      - name: tacos
        image: pierredasilva/tacos-image
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: burgers-services
spec:
  selector:
    app: burgers
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: burgers-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: burgers
  template:
    metadata:
      labels:
        app: burgers
    spec:
      containers:
      - name: burgers
        image: pierredasilva/burgers-image
        ports:
        - containerPort: 80
```

Exemple de l’ingress :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: pizzas.eatsout.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: pizzas-services
            port:
              number: 80
  - host: tacosandburgers.eatsout.com
    http:
      paths:
      - pathType: Prefix
        path: "/tacos"
        backend:
          service:
            name: tacos-services
            port:
              number: 80
      - pathType: Prefix
        path: "/burgers"
        backend:
          service:
            name: burgers-services
            port:
              number: 80
```              

L'application de mes fichiers se fait via les commandes :
```shell
kubectl apply -f services.yaml
kubectl apply -f ingress.yaml
```

Il ne faut surtout pas oublier de modifier le fichier `/etc/hosts` afin de pouvoir accéder aux différentes URL.

![](https://i.imgur.com/I6WXcnq.png)

### 6. Votre magasin de tacos devient très populaire (il va avoir 3 fois plus de commandes). Il va vous falloir gérer une charge importante sur le Service de commande des tacos. Comment gérez-vous cela ? Comment vérifier que la charge est bien répartie (avec quelle commande kubectl ?) ?

On peut choisir d'augmenter le nombre de ReplicaSet. Pour ce faire, on modifie le nombre attribué au champ `replicas` dans le fichier **services.yaml** puison applique les modifications.
Les pods seront automatiquement gérer et reconnu par le services tacos-services grâce aux labels que nous avons défini afin de faire correspondre les critères.
On peut également utiliser la commande `kubectl scale` afin de modifier le nombre de pods déployer pour un déploiement

Afin de voir si on a bien nos nouveaux pods de déployés :

`kubectl get pods --namespace=default`

![](https://i.imgur.com/I0jWxza.png)

On peut voir les logs du déploiement afin de savoir quel pod réponds aux requêtes avec la commande `kubectl logs -f  deploy/tacos-deployment`.

