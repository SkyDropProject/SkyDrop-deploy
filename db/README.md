# Déploiement de MongoDB

Ce document décrit les étapes mises en œuvre pour déployer MongoDB dans un environnement Kubernetes, conformément aux exigences du TP.  
L’objectif est d’assurer un déploiement modulaire, sécurisé et hautement disponible de la base de données.

---

## 🎯 Objectifs pédagogiques du TP

D’après l’énoncé, les deux exigences principales concernant la base de données sont :

1. **Déployer un serveur de base de données (MongoDB) dans un pod séparé**
2. **Assurer la haute disponibilité**
   - Utiliser un StatefulSet et configurer des réplicas
3. **Gérer les informations sensibles via des Secrets Kubernetes**

---

## ✅ Ce qui a été mis en place

### 1. Déploiement dans un pod séparé

- MongoDB est **déployé dans son propre StatefulSet**, indépendant des autres services applicatifs.
- Ce déploiement respecte la consigne de **séparation des composants** : aucun autre conteneur n'est couplé dans le même pod.

### 2. Haute disponibilité via ReplicaSet

- Un **StatefulSet** a été utilisé pour créer **3 pods MongoDB**, nommés `mongo-0`, `mongo-1`, et `mongo-2`.
- Un **Headless Service** (`mongo`) permet au cluster Mongo de détecter automatiquement ses membres via le DNS (`mongo-0.mongo`, etc.).
- Le ReplicaSet MongoDB est initialisé avec ces 3 membres, ce qui garantit une **répartition des rôles PRIMARY / SECONDARY** et assure la **tolérance aux pannes**.

### 3. Configuration via ConfigMap et Secret

- Un **Secret Kubernetes** est utilisé pour stocker de façon sécurisée :
  - Le nom d’utilisateur administrateur (`MONGO_INITDB_ROOT_USERNAME`)
  - Le mot de passe (`MONGO_INITDB_ROOT_PASSWORD`)
- Ces informations sont **montées dans les pods MongoDB via des variables d’environnement**, assurant que les credentials ne sont jamais écrits en clair dans les manifests.
- Un **ConfigMap** permet aussi de centraliser certains paramètres non sensibles.

---

## 📂 Structure des fichiers

- `mongo-secret.yaml` : définit les credentials root sous forme de Secret.
- `mongo-configmap.yaml` *(optionnel)* : contient des paramètres non sensibles.
- `mongo-service.yaml` : expose Mongo via un **Headless Service** (`ClusterIP: None`) pour l’usage avec StatefulSet.
- `mongo-statefulset.yaml` : déploie 3 pods MongoDB avec un volume persistant chacun (`PersistentVolumeClaim`).

---

## 🧪 Tests réalisés

- Vérification que le ReplicaSet MongoDB est bien initialisé avec un PRIMARY et deux SECONDARY via la commande Mongo Shell :
  ```bash
  kubectl exec -it mongo-0 -- mongo
  rs.status()
