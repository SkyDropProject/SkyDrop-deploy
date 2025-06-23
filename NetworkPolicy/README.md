# Mise en place de NetworkPolicies - Projet SkyDrop

Dans ce projet, nous avons mis en œuvre des **NetworkPolicies Kubernetes** pour contrôler précisément les communications réseau entre les différents pods du namespace `skydrop`. L’objectif principal est de **limiter les accès au serveur backend**, et de **sécuriser les flux sortants vers la base de données MongoDB**.

---

## Objectifs

- **Restreindre l'accès au serveur (`skydrop-server`)** uniquement aux pods frontend et web.
- **Empêcher tout autre pod** non explicitement autorisé d’émettre ou de recevoir du trafic réseau.
- **Limiter les connexions sortantes du serveur** uniquement vers la base de données MongoDB.
- **Assurer un modèle par défaut "zero trust"** en bloquant tous les flux entrants et sortants.

---

## Fichiers de configuration déployés

### 🔐 1. `default-deny-all.yaml`

Le but est d'appliquer  une stratégie par défaut qui bloque tous les flux réseau (entrants et sortants) pour tous les pods du namespace skydrop.
Cela garantit que seuls les flux explicitement autorisés par d'autres policies sont permis.

### 🔐 2. `allow-front-web-to-server.yaml`

Le but est d'autoriser uniquement les pods skydrop-frontend et skydrop-web à accéder au serveur (skydrop-server) sur le port TCP 3001.
Tous les autres pods n'ont pas le droit d'émettre de requêtes vers ce pod.

### 🔐 3. `allow-server-to-mongo.yaml`

Le but est de permettre uniquement au serveur d’émettre des connexions sortantes vers le pod MongoDB (avec le label app: mongo-headless).
Ainsi, le serveur ne peut pas communiquer avec d'autres pods, ni accéder à Internet par défaut.

## Tests de vérification

Pour s’assurer du bon fonctionnement de ces règles, nous avons :
- Utilisé un pod de test (kubectl run test-curl) pour tenter de faire un curl vers skydrop-server-service:3001 depuis un pod non autorisé → connexion refusée ✅
- Vérifié que curl depuis le pod frontend ou web fonctionne correctement → connexion acceptée ✅
- Observé que le serveur arrive à communiquer avec MongoDB sans souci → connexion MongoDB OK ✅

## Conclusion

Grâce à ces NetworkPolicies, nous avons mis en place une stratégie de sécurité réseau fine et contrôlée au sein de notre cluster Kubernetes.
Les communications sont autorisées uniquement si elles sont nécessaires, selon le principe du least privilege (moindre privilège). Cela renforce considérablement l’isolation entre les composants applicatifs.


