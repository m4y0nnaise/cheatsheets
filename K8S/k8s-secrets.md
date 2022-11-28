# Kubernetes - Les Secrets

Permettent de gérer les données sensibles.

On les utilise pour la gestion des identifiants, des mots de passes, chaines de connexions à des services etc...

L'utilisation de Secrets permet de découpler les informations sensibles d'une application, ainsi on ne risque pas de trouver un mot de passe écrit en dur dans une image ou dans une spécification.

Un utilisateur peut créer un Secret avec `kubectl` mais le système en créé aussi automatiquement.

Les Secrets sont stockés dans etcd (pas encryptés par défaut).

## Différents types

### generic

Secret créé à partir d'un fichier, d'un répertoire ou d'une valeur littérale.

````bash
# création de secret avec valeurs littérales
$ kubectl create secret generic my_secret \
  --from-literal=username=admin \
  --from-literal=password=123456

secret "service-creds" created
````

````bash
# lister les Secrets
$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-ps46p   kubernetes.io/service-account-token   3      80d  # secret généré par k8s pour accéder à l'api server
my_secret             Opaque                                2      25s
````

````bash
# afficher les secrets
$ kubectl get secrets my_secret -o yaml
apiVersion: v1
data:
  password: MTIzNDU2Cg==    # base64 encoded
  username: YWRtaW4K        # base64 encoded
kind: Secret
metadata:
  # ...
````

````bash
# décoder les secrets
$ echo "YWRtaW4K" | base64 -d
admin
````

#### Utilisation dans un Pod

Utilisation du secret entier :

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: apine
spec:
  containers:
  - name: alpine
    image: alpine
    volumeMounts:                   # montage du volume creds
    - name: creds                   # dans le container
      mounthPath: "/etc/creds"      # à l'emplacement spécifié
      readOnly: true
  volumes:
  - name: creds                     # definitiotn d'un volume
    secret:                         # basé sur le secret
      secretName: my_secret         # my_secret
````

Possibilité de spécifier les clefs à la main :

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: apine
spec:
  containers:
  - name: alpine
    image: alpine
    volumeMounts:                   # montage du volume creds
    - name: creds                   # dans le container
      mounthPath: "/etc/creds"      # à l'emplacement spécifié
      readOnly: true
  volumes:
  - name: creds
    secret:
      secretName: my_secret
      items:                        # possibilité de spécifier
      - key: username               # à la main certaines ou
        path: service/auth          # toutes les clefs d'un secret
      - key: password
        path: service/pass
````

Possibilité de transmettre une variable d'environnement :

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: apine
spec:
  containers:
  - name: alpine
    image: alpine
    env:
    - name: API_URL           # définition de la variable API_URL
      valueFrom:
        secretKeyRef:
          name: my_secret     # lecture du secret
          key: username       # récupération de la valeur
````


### docker-registry

Secret utilisé pour l'authentification à une registry Docker, par exemple si une application doit récupérer des images privées depuis DockerHub.

Création littérale :

````bash
$ kubectl create secret docker-registry registry-creds \
  --docker-server=REGISTRY \
  --docker-username=USERNAME \
  --docker-password=PASSWORD \
  --docker-email=EMAIL \
````

````yaml
# $ kubectl get secret registry-creds -o yaml
apiVersion: v1
data:
  .dockercfg: eyFGHethDV5ghHJgd7X=      # base64 encoded
kind: Secret
metadata:
  # ...
````

### TLS

Secret utilisé pour la gestion des clefs privées et leur certificat associé (PKI).

 

