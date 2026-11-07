# Travelo — Orchestration Kubernetes (Devoir 2)
**Auteur :** Ayoub HAMOUDI — Filière ASEDS — INPT
**Encadrant :** Prof. Driss ALLAKI

Manifestes déclaratifs Kubernetes de l'application 3-tiers **Travelo** (namespace `travelo`).
Le préfixe numérique donne l'ordre d'application : `kubectl apply -f .` applique les fichiers dans l'ordre alphabétique.

## Images

| Image | Rôle |
| :--- | :--- |
| `travelo-backend:2.0` | API Spring Boot, image locale chargée dans Minikube |
| `travelo-frontend:2.0` | SPA React/Nginx, image locale construite avec `REACT_APP_API_URL=http://backend.exam.local` |
| `mysql:8.4` | Base de données (image officielle) |

Construction et chargement dans Minikube (depuis la racine du projet) :

```powershell
docker build -t travelo-backend:2.0 ./travelo_backend
docker build -t travelo-frontend:2.0 --build-arg REACT_APP_API_URL=http://backend.exam.local ./travelo_frontend
minikube image load travelo-backend:2.0
minikube image load travelo-frontend:2.0
```

## Correspondance avec les questions

| Fichier | Objets Kubernetes | Question |
| :--- | :--- | :--- |
| `00-namespace.yaml` | `Namespace` (`travelo`) | Q2 : isolation logique |
| `01-quotas.yaml` | `LimitRange` + `ResourceQuota` | Q3.b : plafonnement CPU / RAM |
| `02-secret.yaml.example` | Modèle de `Secret` (`travelo-secret`) | Q3.a : identifiants MySQL |
| `03-config.yaml` | `ConfigMap` (`travelo-config`) | Q3.a : configuration non sensible |
| `04-database.yaml` | `StatefulSet` + `Service` headless | Q3.a : MySQL 8.4 persistant (PVC 2Gi) ; Q3.e : sondes `exec` |
| `05-backend.yaml` | `Deployment` (2 réplicas) + `Service` ClusterIP | Q3.a : backend Spring Boot ; Q3.e : sondes HTTP |
| `06-frontend.yaml` | `Deployment` (2 réplicas) + `Service` ClusterIP | Q3.a : frontend React/Nginx ; Q3.e : sondes HTTP |
| `07-rbac.yaml` | `ServiceAccount` + `Role` + `RoleBinding` | Q3.c : moindre privilège (RBAC) |
| `08-pdb.yaml` | `PodDisruptionBudget` (backend, frontend) | Q3.d : haute disponibilité |
| `09-ingress.yaml` | `Ingress` (`frontend.exam.local`, `backend.exam.local`) | Q3.f : point d'entrée HTTP unique |
| `10-seed.yaml` | `ConfigMap` + `Job` | Q3.f : données de test (vols, sièges) |

## Secret : à créer avant le déploiement

Le fichier réel `k8s/02-secret.yaml` n'est **pas versionné** (voir `.gitignore`). Pour le créer :

```powershell
cd k8s
Copy-Item 02-secret.yaml.example 02-secret.yaml
# puis remplacer les valeurs CHANGE_ME par vos propres mots de passe
```

## Déploiement dans Minikube

```powershell
cd k8s
kubectl apply -f 00-namespace.yaml
kubectl config set-context --current --namespace=travelo
kubectl apply -f 01-quotas.yaml
kubectl apply -f 02-secret.yaml
kubectl apply -f 03-config.yaml
kubectl apply -f 04-database.yaml
kubectl apply -f 05-backend.yaml
kubectl apply -f 06-frontend.yaml
kubectl apply -f 07-rbac.yaml
kubectl apply -f 08-pdb.yaml
kubectl apply -f 09-ingress.yaml
kubectl apply -f 10-seed.yaml

kubectl get all,pvc
kubectl describe resourcequota travelo-quota
```

## Démonstrations (dossier `demo/`)

| Fichier | Objectif |
| :--- | :--- |
| `defaults-pod.yaml` | Q3.b : Pod sans ressources, valeurs par défaut injectées par le LimitRange |
| `greedy-pod.yaml` | Q3.b : Pod demandant 2 CPU, refusé par le LimitRange |
| `liveness-demo.yaml` | Q3.e : redémarrage d'un conteneur après échec de la livenessProbe |

## Accès à l'application (Minikube, pilote Docker sous Windows)

```powershell
# Terminal administrateur, a laisser ouvert
minikube tunnel -p travelo
# Fichier hosts : 127.0.0.1 frontend.exam.local backend.exam.local
```

Application : http://frontend.exam.local — API : http://backend.exam.local/api/hello
