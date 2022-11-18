# Kubernetes - Les Secrets

Permettent de gérer les données sensibles.

On les utilisent pour la gestion des identifiants, des mots de passes, chaines de connexions à des services etc...

L'utilisation de Secrets permet de découpler les informations sensibles d'une application, ainsi on ne risque pas de trouver un mot de passe écrit en dur dans une image ou dans une spécification.

Un utilisateur peut créer un Secret avec `kubectl` mais le système en créé aussi automatiquement.

Les Secrets sont stockés dans etcd (pas encryptée par défaut).

## Différents types

### generic

Secret créé à partir d'un fichier, d'un répertoire ou d'une valeur littérale.

````bash
# création de secret avec valeurs littérales
$ kubectl create secret generic service-creds \
  --from-literal=username=admin \
  --from-literal=password=123456
secret "service-creds" created
````

````bash
# lister les Secrets
$ kubectl get secrets
NAME            TYPE     DATA   AGE
service-creds   Opaque   2      25s
````
 

### docker-registry

Secret utilisé pour l'authentification à une registry Docker.

### TLS

Secret utilisé pour la gestion des clefs privées et leur certificat associé (PKI).



