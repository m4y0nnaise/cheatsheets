# Kubernetes - Les Namespaces

Les namespaces permettent de segmenter les clusters en isolant un certain nombre de ressources (pods, services, deployments...). 

La segmentation permet de partager un cluster entre plusieurs équipes, projets ou clients : c'est lapproche _multi-tenant_.

Par défaut on a trois namespaces :
- default
- kube-public
- kube-system

Si on ne précise pas quel namespace est utilisé ce sera toujours le _default_.

## 1. Création

````bash
# création de my_namespace par commande
$ kubectl create namespace my_namespace
````

````bash
# suppression de my_namespace
$ kubectl delete namespace my_namespace
````

````yaml
# création par spécification dans my_namespace.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: my_namespace
  labels:
    name: development
````

````bash
# création avec fichier de spécification
$ kubectl create -f my_namespace.yaml
````

On peut également spéficier un namespace directement dans la spécification d'une autre ressource :

````yaml
# dans un pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: my_namespace
#...
````

## 2. Utilisation

Une fois notre Pod créé avec le namespace my_namespace précisé, on peut observer ceci :

````bash
$ kubectl get pods
No ressources found.
````

Car le namespace utilisé par défaut et toujours 'default'. Il faut donc préciser la commande. 

````bash
# tri par namespace
$ kubectl get pods --namespace=my_namespace
NAME    READY    STATUS    RESTARTS    AGE
nginx   1/1      Running   0           30s
````

````bash
# affiche toutes les ressources de tous les namespaces
$ kubectl get pods --all-namespace
NAME    READY    STATUS    RESTARTS    AGE
nginx   1/1      Running   0           30s
````

## 3. Context

Un context est une configuration qui permet de définir un cluster, un namespace et un user.

````yaml
# exemple de spécification de context
contexts:
- context:
    cluster: cl-minikube
    namespace: ns-minikube
    user: u-minikube
  name: c-minikube
current-context: c-minikube
````

Lors de la création de ressources elles seront automatiquement ajoutées :
- au cluster __cl-minikube__
- au namespace __ns-minikube__
- par l'user __u-minikube__

Car le contexte actuel est __c-minikube__

## 4. Utilisation des ressources

### 4.1 ResourceQuota

Il est possible de limiter l'utilisation des ressources qu'utilise chaque namespace du cluster.

Pour cela il faut créer un ResourceQuota et sa spécification :

````yaml
# quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
````

- chaque container devra spécifier des demandes et des limites pour la RAM et le CPU
- l'ensemble des containers ne pourra pas 
  - demander plus de 1GB de RAM
  - utiliser plus de 2GB de RAM
  - demander plus d'1 CPU
  - utiliser plus de 2 CPUs

Pour associer le ResourceQuota à un namespace :

````bash
# lier un ResourceQuota à un namespace
$ kubectl apply -f quota.yaml --namespace=my_namespace
````

### 4.2 LimitRange

Pour limiter l'usage de chaque ressources dans un namespac on utilise un LimitRange.

Comme pour un ResourceQuota on definit les limites via un fichier de spécification.

````yaml
# exemple de spécification
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limit-range
spec:
  limits:
  - default:
      memory: 128M
    defaultRequest:
      memory: 64M
    max:
      memory: 256M
    type: Container
````

Dans la spécification ci-dessus on précise une limitation de mémoire pour chaque container du namespace.