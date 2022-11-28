# Kubernetes - Les ConfigMap

Permet de gérer la configuration d'une application en découpant une application de sa configuration. Cela permet de retirer toute forme de configuration dans le code d'une application.

Grâce à on va pouvoir assurer la portabilité en changeant facilement la configuration suivant l'environnement dans lequel on déploit l'application.

Simplifie la gestion par rapport à l'utilisation de variables d'environnement.

Comme pour un secret, une ConfigMap ça peut être crée à partir d'un fichier, d'un répertoire ou de valeurs littérales

Un objet ConfigMap contient une ou plusieurs paires de clef / valeur.

## Création d'une ConfigMap

###  Basée sur un fichier

````conf
# nginx.conf
user www-data;
worker_processes 4;
pid /run/nginx.pid;
events {
  worker_connections 768;
}
http {
  server {
    listen *:8000;
    location / {
      proxy_pass http://localhost;    # les requêtes arrivant sur le port 8000 son envoyées
    }                                 # au container tournant sur le port 80 dans le même Pod
  }                                   # la communication entre des containers au sein d'un même Pod passe par l'interface localhost
}
````

Création d'une ConfigMap à partir d'un fichier :

````bash
$ kubectl create configmap nginx-config --from-file=./nginx.conf
configmap "nginx-config" created
````

Si on affiche la spécification de la ConfigMap :

````yaml
# kubectl get cm nginx-config -o yaml
apiVersion: v1
data:
  nginx.conf: |             # contient en clair le contenu
    user www-data;          # du fichier nginx.conf
    worker_processes 4;
    # ...
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
  selfLink: /api/v1/namespaces/default/confimaps/nginx-config
  # ...
````

 ### Basée sur un fichier d'environnement
 
 ````env
# app.env
log_level=WARN
env=production
cache=redis
````

Création d'une ConfigMap à partir d'un fichier :

````bash
$ kubectl create configmap app-config-env --from-env-file=./app.env
configmap "app-config-env" created
````

Si on affiche la spécification de la ConfigMap :

````yaml
# kubectl get cm app-config-env -o yaml
apiVersion: v1
data:
  cache: redis             # contient en clair le contenu
  env: production          # du fichier app.env
  log_level: WARN
kind: ConfigMap
metadata:
  name: app-config-env
  namespace: default
  selfLink: /api/v1/namespaces/default/confimaps/app-config-env
  # ...
````








