# Kubernetes - Authentification & Autorisation

## Authentification

### Niveau Utilisateurs

- Certificat client

--client-ca-file=FILE au démarrage de l'API Server

- Bearer tokens

--token-auth-file=FILE au démarrage de l'API Server

- HTTP basic auth

--basic-auth-file=FILE au démarrage de l'API Server

### Niveau Processus

Pour authentifier les processus il existe un ServiceAccount pour chaque namespace mais ils ont des droits très limités. On peut créer nos propres ServiceAccount avec des droits très précis.

Quand on lance un Pod on lui associe un ServiceAccount. Si on le ne fais pas il utilise celui par défaut. Grâce au token jwt associé au ServiceAccount il pourra s'authentfier et faire des requêtes à l'API Server. 

````bash
# lister les ServiceAccount
$ kubectl get sa --all-namespaces
NAMESPACE       NAME        SECRETS     AGE
default         default     1           10d
kube-public     default     1           10d
kube-system     default     1           10d
````

### Inspection d'un ServiceAccount :

````yaml
# $ kubectl get sa/default -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  # ...
secrets:
- name: default-token-jfp6k     # secret contenu
# ...
````

### Inspection du Secret :

`````yaml
# $ kubectl get secret default-token-jfp6k -o yaml
apiVersion: v1
kind: Secret
metadata:
  name: default-token-jfp6k
  namespace: default
data:
  ca.crt: LS0tLS1CRU...0tLS0tCg==
  namespace: ZGVmYXVsdA==
  token: RGHfvlm6polHF...3DfrZS4==    # JWT base64 encoded
# ...
````

## Autorisation



RBAC RoleBasedAccessControl
