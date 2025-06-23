# Déploiement sécurisé de SkyDrop dans Kubernetes

## 1. Installation des pré-requis et de Kubernetes

### Pré-requis

- Ubuntu 24.04
- 6 GB de mémoire vive
- 35 Gio de stockage

### Installation de Docker

Assurez vous que tous vos packages système sont à jours, puis installez curl

```shell
sudo apt update
sudo apt install -y apt-transport-https curl
```

Installez Docker :

```shell
sudo apt install docker.io
```

Permettez à l'utilisateur courant d'accéder à Docker :

```shell
sudo usermod -aG docker $USER
newgrp docker
```

Activez le service de Docker :

```shell
sudo systemctl enable docker
sudo systemctl start docker
```

### Installation de Minikube

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Installation de l'utilitaire kubectl

```shell
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Installation du CNI

```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Démarrage de Minikube

```shell
minikube start --network-plugin=cni --cni=calico
```

## 2. Installation de l'environnement

Afin d'organiser, isoler et gérer les ressources plus efficacement, on va créer un namespace dédié au projet.

```shell
kubectl create namespace skydrop
kubectl config set-context --current --namespace=skydrop
```
