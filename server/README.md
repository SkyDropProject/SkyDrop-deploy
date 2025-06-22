# Déploiement de l’API `skydrop-server`

Ce document décrit les étapes nécessaires au déploiement de l'API `skydrop-server` dans un cluster Kubernetes. L’objectif est de respecter les exigences du TP, notamment en matière de séparation des responsabilités, sécurisation des informations sensibles et interconnexion avec MongoDB.

## 🎯 Objectif

Déployer une API Node.js (port 3001) dans un namespace dédié `skydrop`, avec :

- Un `Deployment` pour l'API
- Un `Service` interne pour l'exposer dans le cluster
- Un `Secret` pour les variables sensibles (MongoDB, JWT, etc.)

---

## 📁 Fichiers utilisés

- `server-secret.yaml` : Définit les variables d’environnement sensibles
- `server-deployment.yaml` : Définit le déploiement de l’API
- `server-service.yaml` : Crée un service de type `ClusterIP` pour exposer l’API

---

## 🧩 Étapes de déploiement

### 1. Création du Secret

On commence par définir un `Secret` Kubernetes pour stocker les variables d’environnement sensibles telles que :

- `PORT` : port d'écoute de l’API (3001)
- `MONGO_URL` : URI de connexion à la base MongoDB avec nom d’utilisateur, mot de passe, nom de réplica et authSource
- `JWT_SECRET` : clé utilisée pour signer les tokens JWT
- `SALT_ROUNDS` : niveau de salage des mots de passe

Ce `Secret` est injecté dans le conteneur via `envFrom.secretRef`.

### 2. Déploiement de l’API

Un `Deployment` est utilisé pour garantir la scalabilité et la robustesse du service. Il définit :

- L’image Docker utilisée (`skydrop-server:latest`)
- Le port d’écoute (3001)
- L’environnement chargé depuis le Secret
- Le label `app: skydrop-server` utilisé pour le selector du `Service`

L’image doit être disponible localement sur le nœud (image buildée avec `imagePullPolicy: IfNotPresent`) ou sur un registry privé.

### 3. Création du Service

Un `Service` de type `ClusterIP` est créé pour exposer l'API uniquement à l’intérieur du cluster. Il permet aux autres pods/services (ex. : une application front ou des jobs) de communiquer avec l’API via un nom DNS interne : http://skydrop-server-service.skydrop.svc.cluster.local:3001

---

## ✅ Commandes d’application

Voici l’ordre des commandes à exécuter dans le namespace `skydrop` :

```bash
# Appliquer les variables d’environnement sensibles
kubectl apply -f server-secret.yaml -n skydrop

# Déployer l’API
kubectl apply -f server-deployment.yaml -n skydrop

# Créer le service pour exposer l’API
kubectl apply -f server-service.yaml -n skydrop

## 🔍 Vérification du déploiement

Pour vérifier que l’API est correctement déployée :

```shell
kubectl get pods -n skydrop
kubectl logs -n skydrop <nom-du-pod>
```

Depuis un autre pod dans le cluster :

```shell
curl http://skydrop-server-service.skydrop.svc.cluster.local:3001/
```

## 📌 Exigences du TP respectées

## 📌 Exigences du TP respectées

| Exigence                            | Mise en œuvre                                     |
|-------------------------------------|---------------------------------------------------|
| Utilisation de Secrets              | ✅ Utilisation de `server-secret.yaml`            |
| Variables sensibles non hardcodées | ✅ Injectées via `envFrom.secretRef`              |
| Déploiement modulaire              | ✅ Séparation des fichiers YAML (`Deployment`, `Service`, `Secret`) |
| Portabilité et interopérabilité    | ✅ Déploiement dans un namespace dédié `skydrop`  |
| Connexion MongoDB sécurisée        | ✅ URI avec utilisateur, mot de passe, replicaSet et `authSource=admin` |
| Conformité au modèle 12-factor     | ✅ Configuration externe, gestion de l’environnement, stateless, etc. |


## 🧠 Remarque

Avant le déploiement de l’API, assurez-vous que :
- MongoDB est déployé et prêt à accepter les connexions (auth activée + réplication OK)
- Le nom DNS mongo-service.skydrop.svc.cluster.local est résolu correctement
- Le namespace skydrop est bien créé

