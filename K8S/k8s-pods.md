# Kubernetes - Les Pods

Plus petite unité applicative dans k8s c'est un groupe de containers qui ont le même contexte d'isolation.

Ces containers partagent la même stack réseau : ils peuvent communiquer entre eux sur l'interface localhost. Ils peuvent également partager les mêmes volumes.

Chaque pod à une adresse ip dédiée dans le cluster.

Souvent un pod utilise un seul container applicatif.

Un pod c'est un ensemble de container et une application est découpée en un ou plusieurs pods. On peut voir chaque pod comme un service métier ou technique d'un application.

---

## 1. Notion de spécification

Un pod est basé sur une spéficiation, qui définit notamment les containers qui tourneront dans le pod. Chaque spécification correspond à un service métier (microservice).
On peut déployer de nouveaux pods à partir d'une spécification, on parle ainsi de scaling horizontal.

Exemple de specification ``nginx-pod.yaml``:

````yaml
apiVersion: v1
kind: Pod       # type d'élément
metadata:       # namespace, labels...
  name: nginx 
spec:           # spécification des containers lancés dans le Pod
  containers:
  - name: www
    image: nginx:1.12.2
````

---

## 2. Cycle de vie d'un Pod

````bash
# créer le pod
$ kubectl create -f pod-spec.yaml
````
````bash
# lister les pods du namespace 'default'
$ kubectl get pod
````
````bash
# obtenir les détails associés au pod 'my_pod'
$ kubectl describe pod my_pod

# forme abrégée
$ kubectl describe po/my_pod
````
````bash
# logs d'un pod [d'un container en particulier]
$ kubectl logs my_pod [-c my_container]
````
````bash
# lancer une commande dans un container du pod
$ kubectl exec my_pod [-c my_container] -- echo "Hey"
````
````bash
# lancer un shell interactif dans un container en particulier
$ kubectl exec -ti my_pod [-c my_container] -- /bin/bash
````
````bash
# supprimer un pod
$ kubectl delete pod my_pod
````
````bash
# map le port du container à l'hote pour accéder à l'application sur l'hote
$ kubectl port-forward my_pod HOST_PORT:CONTAINER_PORT
````

---

## 3. Le container Pause

Un container **pause** est présent pour chaque pod et agit comme garant des namespaces pour isoler les processus du pod.

Pour communiquer les containers du pod s'appuie dessus.

---

## 4. Scheduling

L'étape de scheduling est la sélection du node sur lequel un pod sera déployé. Elle est effectuée par le composant ``kube-scheduler``.

````bash
# ajout d'un label sur le node my_node 
kubectl label nodes my_node disktype=ssd
````

On peut ajouter une contrainte dans la spécification d'un pod pour qu'il soit déployé sur un type de node en particulier.

````yaml
# Modification du pod-config.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my_pod
  labels:
    env: prod
spec:
  containers:
  - name: mysql
    image: mysql
  nodeSelector:     # ajout de la clef
    disktype: ssd   # contrainte
````

### 4.1 nodeAffinity

Les nodeAffinity permettent de schéduler les pods sur certains nodes seulement et de manière plus granulaire qu'vec nodeSelector.

On peut utiliser des opérateurs :
- In
- NotIn
- Exists
- DoesNotExist
- Gt
- Lt
- ...

On peut également filtrer les nodes par leurs labels.

Pour filtrer il existe différentes règles :
- requiredDuringSchedulingIgnoredDuringExecution => MUST HAVE 
- preferedDuringSchedulingIgnoredDuringExecution => SHOULD HAVE

Exemple :

````yaml
# ...
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: # MUST have
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name  # ce label
            operator: In                    # doit avoir
            values:                         # sa valeur
            - e2e-az1                       # = e2e-az1
            - e2e-az2                       # ou e2e-az2
      preferedDuringSchedulingIgnoredDuringExecution: # SHOULD have
      - preference:         # si possible
        matchExpressions:
        - key: disktype     # ce label
          operator: In      # doit avoir
          values:           # sa valeur
          - ssd             # = ssd
````

### 4.2 podAffinity / podAntiAffinity

Les podAffinity et podAntiAffinity permettent de schéduler des pods en fonctions de labels présents sur d'autres pods.

Il existe différentes règles :
- requiredDuringSchedulingIgnoredDuringExecution => MUST HAVE 
- preferedDuringSchedulingIgnoredDuringExecution => SHOULD HAVE

On utilise le champ **topologyKey** pour la spécification de domaines topologiques :
- hostname
- region
- az
- ...

````yaml
# ...
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:   # le pod devra être placé sur un node
          - key: security     # qui est dans la même zone de disponibilité
            operator: In      # d'un pod dont la valeur du label security
            values:           # est S1
            - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferedDuringSchedulingIgnoredDuringExecution:
      - podAffinityTerm:
        labelSelector:
          matchExpressions:   # De préférence, le pod ne devra pas
          - key: security     # être placé sur un node sur lequel
          operator: In        # tourne un pod dont le label security
          values:             # a la valeur S2
          - S2
        topologyKey: kubernetes.io/hostname
````

### 4.3 Taints et Tolerations

Un pod doit tolérer la taint d'un node pour pouvoir être exécuté sur celui-ci.

La taint se positionne sur un node tandis que la toleration se positionne sur un pod.

Une taint c'est une clef et un effet :

````yaml
# sur le node
spec:
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
````

````yaml
# sur le pod
spec:
  tolerations:
  - key: node-role.kubernetes.io/master
    effect: NoSchedule
````

### 4. Allocation des ressources

Best practice : toujours spécifier les ressources minimales et maximales pour d'un container. Cela aide le scheduler à positionner les pods.

````yaml
# ...
spec:
  containers:
  - name: db
    image: mysql
    env:
    name: MYSQL_ROOT_PASSWORD
      value: "password"
    ressources:
      requests:         # minimum requis
        memory: "64Mi"  # 64Mo de RAM
        cpu: "250m"     # 0,25 CPU
      limits:           # maximum nécessaire
        memory: "128Mi" # 128Mo de RAM
        cpu: "500m"     # 0,5 CPU
````
