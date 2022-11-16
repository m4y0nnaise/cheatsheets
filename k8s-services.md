# Kubernetes - Les Services

Les services permettent d'exposer les pods d'une application via des règles réseaux. C'est à dire rendre disponible une application soit à d'autres pods (à l'intérieur du cluster) soit à l'extérieur vers des clients.

Un service permet de regouper des pods qui ont la même spécification grâce à des labels

Chaque service a sa propre adresse ip interne au cluster qui est résolue au niveau DNS à partir du nom du service. Quand une requête est envoyée sur l'adresse ip elle est automatiquement envoyée sur l'un des pods géré par ce service. On a donc une répartition de la charge entre les pods.

``kube-proxy`` est en charge du load balancing sur les Pods, il peut utiliser différentes technologies :
- userspaces
- iptables
- IPVS

Il existe plusieurs types de services :
- ClusterIP (défaut) : exposition à l'intérieur du cluster
- NodePort : exposition vers l'extérieur
- LoadBalancer : intégration avec un Cloud Provider
- ExternalName : associe le service à un nom DNS

---

## 1. ClusterIP

Ce service est utilisé pour rendre disponible un ensemble de pod à l'intérieur du cluster.

L'application qui tourne dans ces pods pourra seulement être consommée par les autres pods qui tournent dans le cluster.

Chaque requete qui arrive sur l'ip du service est redirigée vers l'un des pods du service.

### 1.1 Spécification

````yaml
apiVersion: v1
kind: Service
metadata:
  name: vote
spec:
  selector:
    app: vote
  type: ClusterIP
  ports:
  - port: 80        # le service expose le port 80 dans le cluster
    targetPort: 80  # les requêtes sont envoyées sur le port 80 d'un pod du groupe
````

Pour grouper des pods les services utilisent des labels. Le service décrit au dessus selectionne les pods ayant le label ``app: vote``.

### 2. Accéder au service : port-forward

Grâce au port forwarding on peut créer un tunnel entre notre machine et le service du cluster afin d'accéder à l'application qu'il contient.

````bash
# mapping du port local 8080 avec le port 80 du service
kubectl port-forward svc/my_service 8080:80
````

> Il faut fermer le tunnel quand on a terminé nos actions.

### 3. Accéder au service : proxy

Grace au proxy on n'est plus limité à un seul service mais on peut accéder à l'ensemble des ressources du cluster. Le proxy de k8s ouvre un tunnel entre la machine et l'API Server.

````bash
# lancement du proxy (port 8001)
kubectl proxy
````

On accède ensuite au service désiré en précisant les paramètres dans l'URL de la requête :

````bash
# atteindre l'application
curl http://localhost:8001/api/v1/namespaces/default/services/my_service:80/proxy/
````

> Il faut fermer le tunnel quand on a terminé nos actions.

## 2. NodePort

Un service de type NodePort sert à exposer les pods à l'extérieur du cluster. Mais il expose également les pods à l'intérieur du cluster car c'est aussi un service de type ClusterIP.

Le NodePort ouvre un port sur chaque machine du cluster. Le port est toujours situé entre 30000 et 32767.

### 2.1 Spécification

````yaml
apiVersion: v1
kind: Service
metadata:
  name: vote-np
spec:
  selector:
    app: vote
  type: NodePort
  ports:
  - port: 80        # le service expose le port 80 dans le cluster
    targetPort: 80  # les requêtes sont envoyées sur le port 80 d'un pod du groupe
    nodePort: 31000 # port ouvert sur l'exterieur
````

## 3. LoadBalancer

Le Loadbalancer est aussi un service de type NodePort mais il permet d'ajouter une fonctionnalité de load balancing. Cependant il nécessite l'utilisation d'un Cloud Provider.

````yaml
apiVersion: v1
kind: Service
metadata:
  name: vote-lb
spec:
  selector:
    app: vote
  type: LoadBalancer
  ports:
  - port: 80        # le service expose le port 80 dans le cluster
    targetPort: 80  # les requêtes sont envoyées sur le port 80 d'un pod du groupe
````


