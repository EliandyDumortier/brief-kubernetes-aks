# Kubernetes AKS – Python API & MySQL

## 📌 Objectif du projet

Ce projet consiste à déployer une architecture backend composée :
- d’une API Python
- d’une base de données MySQL

L’ensemble est déployé sur **Azure Kubernetes Service (AKS)** à l’aide de manifests Kubernetes, en respectant les bonnes pratiques de conteneurisation, de persistance des données et d’exposition réseau.

---

## 🧱 Architecture

- API Python exposée via un Ingress
- Base MySQL interne au cluster
- Communication API ↔ MySQL via Services Kubernetes
- Données MySQL persistées grâce à un PersistentVolumeClaim

Utilisateur
↓
Ingress (NGINX)
↓
Service API
↓
Pods API
↓
Service MySQL
↓
Pod MySQL + PVC


---

## 🐳 Images Docker utilisées

- **API** : `sengsathit/brief-api:latest`
- **MySQL** : `sengsathit/brief-mysql:latest`

---

## 🌐 Endpoints exposés

- Health check  
  `GET /kubernetes-brief/health`

- Clients  
  `GET /kubernetes-brief/clients`  
  `POST /kubernetes-brief/clients`

- Client par ID  
  `GET /kubernetes-brief/clients/{id}`  
  `DELETE /kubernetes-brief/clients/{id}`

---

## 💾 Persistance des données

La base MySQL utilise un **PersistentVolumeClaim** basé sur la `StorageClass` `standard`.

Sur AKS, cela correspond à un **Azure Managed Disk** créé dynamiquement par Kubernetes.
Les données sont conservées même si le pod MySQL est recréé.

---

## ❤️ Health checks

- **Readiness probe** : vérifie si l’API est prête à recevoir du trafic
- **Liveness probe** : redémarre le pod si l’API ne répond plus

L’Ingress ne route le trafic que vers les pods en état *Ready*.

---

## 🚀 Déploiement

```bash
kubectl apply -f manifests/00-namespace.yaml
kubectl apply -f manifests/01-secrets.yaml
kubectl apply -f manifests/02-mysql-pvc.yaml
kubectl apply -f manifests/03-mysql-deployment.yaml
kubectl apply -f manifests/04-mysql-service.yaml
kubectl apply -f manifests/05-api-deployment.yaml
kubectl apply -f manifests/06-api-service.yaml
kubectl apply -f manifests/07-ingress.yaml

