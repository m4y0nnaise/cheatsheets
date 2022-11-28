# Kubernetes commands

## I. kubectl

---

````bash
# liste les nodes créés
kubectl get node
````

````bash
# affiche les informations d'un cluster
kubectl cluster-info
````
````bash
# affiche les contexts existants (différents clusters)
kubectl config get-contexts
````

````bash
# affiche les informations d'un cluster dans un contexte donné
kubectl cluster-info --context kind-k8s
````

## II. minikube

---

````bash
# crée un cluster de 1 node
minikube start
````

````bash
# crée un cluster de 3 nodes
minikube start --nodes 3
````

````bash
# détruit le cluster
minikube delete
````

## III. Multipass

Permet de lancer des VMs Ubuntu en quelques secondes.

---

````bash
# lance une VM
multipass launch -n vm1
````
````bash
# lance une VM en spécifiant les specs
multipass launch -n node2 -c 2 -m 3G -d 10G
````
- -n = name
- -c = cpu
- -m = ram
- -d = disk

````bash
# affiche infos de la VM vm1
multipass info vm1
````

````bash
# liste les VMs lancées
multipass list
````

````bash
# lance un shell dans la VM vm1
multipass shell vm1
````

````bash
# execute une commande : installer docker
multipass exec vm1 -- /bin/bash -c "curl -sSL https://get.docker.com | sh"
````

## IV. kind

Kubernetes in Docker

---

````bash
# crée un cluster d'1 node appelé k8s
kind create cluster --name k8s
````

### 1- Créer un cluster à plusieurs nodes
Nécessité de créer un fichier ``config.yaml``. Exemple :

````yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
````

Ensuite spécifier le fichier dans la commande de création.

````bash
# crée un cluster k8s-2 selon les specs de config.yaml
kind create cluster --name k8s-2 --config config.yaml
````

````bash
# delete le cluster k8s-2
kind delete cluster --name k8s-2
````

## V. Microk8s

---
````bash
# installe microk8s dans une VM
sudo snap install microk8s --classic
````

## VI. k3s

---


````bash
# 
````

