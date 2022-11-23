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

````yaml
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

Après l'étape d'Authentification qui permet de vérifier qu'un utilisateur ou un processus à le droit de s'adresser à l'API Server vient l'étape d'Autorisation qui vérifie si l'utilisateur ou le process en question a bien le droit d'effectuer l'action demandée.

Pour ça kubernetes implémente un système de RBAC (Role Based Access Control). C'est un ensemble de règle qui s'applique aux utilisateurs, aux groupes d'utilisateurs ou au ServiceAccount qui permet l'accès aux ressources du Cluster de façon grannulaire.

### Ressources

Pour mettre en place ce RBAC il existe différentes ressouces :

- les Roles
Permettent de définir les règles dans un namespace. Ils sont toujours limités à 1 namespace.

- les ClusterRoles
Permettent de définir des règles dans le Cluster en entier.

- les RoleBinding
Permettent d'associer les Roles avec des utilisateurs ou ServiceAccount.

- les ClusterRolesBinding
Permettent d'associer les ClusterRoles avec des utilisateurs ou ServiceAccount.

#### Role & RoleBinding

````yaml
# exemple de spécification pour un Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development    # namespace concerné par les règles
  name: dev-pod-reader
rules:
- apiGroups: [""]
  ressources: ["pods"]      # sur quelles ressources on
  verbs: ["get", "list"]    # autorise quelles actions
````

````yaml
# exemple de spécification pour un RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development    # namespace concerné par les règles
  name: dev-pod-reader
subjects:                   # attends une liste de users ou de ServiceAccount
- kind: User                                
  name: luc                 # donne au user luc
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role                # les droits associé au role
  name: dev-pod-reader      # dev-pod-reader
  apiGroup: rbac.authorization.k8s.io
````

#### ClusterRole & ClusterRoleBinding

````yaml
# exemple de spécification pour un ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:                   # pas de namespace car s'applique à tous
  name: secret-reader
rules:
- apiGroups: [""]
  ressources: ["secret"]    # permet d'accéder aux secrets
  verbs: ["get", "list"]    # de les lister et visualiser
````

````yaml
# exemple de spécification pour un ClusterRoleBinding/
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets
subjects:
- kind: ServiceAccount
  name: manager
  apiGroup: rbac.autorization.k8s.io
roleRef:
  kind: Clusterrole
  name: secret-reader
  apiGroup: rbac.autorization.k8s.io
````


