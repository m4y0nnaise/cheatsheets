# Kubernetes entités - Les Deployments

Un Deployment permet de gérer un ensemble de pods identiques et de s'assurer qu'on a constamment un certain nombre de pods qui tournent sur le cluster.

Si 3 replicas sont définis dans la spécification du Deployment et qu'1 crash il est automatiquement remplacé.

## 1. Utilisation

Un Déployment ça permet de gérer le cycle de vie des pods :
- Création
- Suppression
- Scaling
- Rollout (maj)
- Rollback

## 2. ReplicaSet

Une ressource est présente entre un Deployment et ses pods c'est le ReplicaSet. Son rôle est de vérifier continuellement que les pods sont bien conformes à la spécification du Deployment. S'il y a divergence il lance des actions correctives comme par exemple redémarrer un pod qui ne répond pas.

> On ne touche quasiment jamais directement les ReplicaSet. Ils se modifient toujours par le biais de la spécification du Deployment.

## 3. Spécification

````yaml
apiVersion: apps/v1             # === 
kind: Deployment                #
metadata:                       # Deployment
  name: vote                    #
spec:                           # ===
  replicas: 3                   # ===
  selector:                     # ReplicaSet
    matchLabels:                #
      app: vote                 # ===
  template:                     # ===
    metadata:                   #
      labels:                   #
        app: vote               #
    spec:                       # Pod
      containers:               #
      - name: vote              #
        image: instavote/vote   #
        ports:                  #
        - containerPort: 80     # ===
````

## 4. Mise à jour

Il existe différentes stratégies de mise à jour d'une application. Celles utilisées dans k8s sont les **Rolling Update** et les **Recreate**.

### 4.1 Canary

La méthode Canary consiste à rediriger environ 5% des requêtes vers la v2 d'une application le temps de finaliser les tests. Lorsque la v2 est prête le trafic dirigé vers la v1 chute rapidement pour atteindre les 0%.

### 4.2 Rolling update

La Rolling update est la méthode par défaut sur les cluster k8s. Le pourcentage de traffic vers la v2 est progressivemment augmenté pour atteindre les 100% à la fin.

Dans la spécification d'un déploiement :

````yaml
# ...
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1         # nombre de replica en plus des replicas courants
      maxUnavailable: 0   # nombre de replicas en moins
````

### 4.3 Recreate

Méthode brutale qui vise à terminer tous les pods de la v1 puis à recréer les pods en v2. Entraine un arrêt de service.

### 4.4 Blue-Green

Fonctionne de la même manière que la Recreate à la différence que les pods de la v2 sont créés avant que ceux de la v1 soient terminés. Cela supprimer l'interruption de service.

## 5. Historique des mises à jour

````bash
# montre les 10 dernières modifications du Deployment
kubectl rollout history deploy/my_deploy
````

````bash
# pour enregistrer une modification dans l'historique
kubectl ... --record
````

### 5.? Rollback

...

## 6. HorizontalPodAutoscaler

Permet de scaler le nombre de Pods selon des contraintes définies.

````yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: www                 # nom du HPA
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment        # type et nom
    name: www               # de l'objet à surveiller
  minReplicas: 2
  maxRecplicas: 10
  targetCPUUtilizationPercentage: 50
````



