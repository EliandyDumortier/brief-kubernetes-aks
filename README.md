<div align="center">

# 🚀 Kubernetes – Python API & MySQL

### Architecture Backend Cloud-Native

[![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)](https://www.python.org/)
[![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)

**Une architecture backend déployée sur Kubernetes avec API Python et base MySQL**

[Objectif](#-objectif-du-projet) • [Architecture](#-architecture) • [Déploiement](#-déploiement-local-docker-desktop) • [Endpoints](#-endpoints-exposés)

</div>

---

## 📌 Objectif du projet

Ce projet consiste à déployer une architecture backend composée :
- d'une **API Python**
- d'une **base de données MySQL**

L'architecture est déployée sur un **cluster Kubernetes** à l'aide de manifests YAML, en respectant les bonnes pratiques de :
- ✅ Conteneurisation
- ✅ Persistance des données
- ✅ Exposition réseau via Ingress

### 💡 Points clés

👉 Pour des raisons de **simplicité et de reproductibilité**, le déploiement est réalisé **en local avec Kubernetes fourni par Docker Desktop**.

👉 Les manifests sont **compatibles avec Azure Kubernetes Service (AKS)** moyennant des ajustements minimes (notamment la `StorageClass`).

---

## 🧱 Architecture

### Vue d'ensemble

```
                    Utilisateur
                        ↓
                Ingress (NGINX)
                        ↓
                   Service API
                        ↓
                   ┌────────────┐
                   │ Pods API   │
                   └────────────┘
                        ↓
                  Service MySQL
                        ↓
            ┌──────────────────────┐
            │ Pod MySQL + PVC      │
            │ (Données persistées) │
            └──────────────────────┘
```

### Composants

| Composant | Description |
|-----------|-------------|
| **API Python** | Exposée via un Ingress NGINX |
| **MySQL** | Base de données interne au cluster |
| **Services K8s** | Communication API ↔ MySQL |
| **PersistentVolumeClaim** | Persistance des données MySQL |

---

## 🐳 Images Docker utilisées

| Service | Image |
|---------|-------|
| **API** | `sengsathit/brief-api:latest` |
| **MySQL** | `sengsathit/brief-mysql:latest` |

---

## 🌐 Endpoints exposés

### Health Check
```
GET /kubernetes-brief/health
```

### Clients
```
GET  /kubernetes-brief/clients          (Lister tous les clients)
POST /kubernetes-brief/clients          (Créer un nouveau client)
GET  /kubernetes-brief/clients/{id}     (Récupérer un client par ID)
DELETE /kubernetes-brief/clients/{id}   (Supprimer un client)
```

---

## 💾 Persistance des données

La base MySQL utilise un **PersistentVolumeClaim (PVC)**.

<table>
<tr>
<td width="50%">

### 🖥️ Déploiement local (Docker Desktop)

- `StorageClass` par défaut : **hostpath**
- Utilisée automatiquement
- Données conservées tant que le PVC existe

</td>
<td width="50%">

### ☁️ Déploiement AKS

- `StorageClass` : **standard**
- Provisionne un **Azure Managed Disk**
- Haute disponibilité et scalabilité

</td>
</tr>
</table>

---

## ❤️ Health Checks

Les sondes de santé assurent la qualité du service :

| Type | Fonction | Impact |
|------|----------|--------|
| **Readiness probe** | Vérifie si l'API est prête à recevoir du trafic | ✓ Pod inclus dans l'équilibreur de charge |
| **Liveness probe** | Redémarre le pod si l'API ne répond plus | ✓ Maintient la disponibilité |

**L'Ingress ne route le trafic que vers les pods en état *Ready*.**

---

## 🚀 Déploiement local (Docker Desktop)

### Prérequis

```bash
✓ Docker Desktop
✓ Kubernetes activé dans Docker Desktop
✓ kubectl installé
```

### Installation

<details>
<summary><b>Étape 1 : Namespace</b></summary>

```bash
kubectl apply -f manifests/00-namespace.yaml
```
Crée un namespace isolé pour le déploiement.

</details>

<details>
<summary><b>Étape 2 : Secrets</b></summary>

```bash
kubectl apply -f manifests/01-secrets.yaml
```
Configure les identifiants MySQL sécurisés.

</details>

<details>
<summary><b>Étape 3 : Persistance MySQL</b></summary>

```bash
kubectl apply -f manifests/02-mysql-pvc.yaml
```
Crée le volume persistent pour les données.

</details>

<details>
<summary><b>Étape 4 : Déploiement MySQL</b></summary>

```bash
kubectl apply -f manifests/03-mysql-deployment.yaml
kubectl apply -f manifests/04-mysql-service.yaml
```
Déploie la base de données et expose le service.

</details>

<details>
<summary><b>Étape 5 : Déploiement API</b></summary>

```bash
kubectl apply -f manifests/05-api-deployment.yaml
kubectl apply -f manifests/06-api-service.yaml
```
Déploie l'API Python et expose le service.

</details>

<details>
<summary><b>Étape 6 : Ingress</b></summary>

```bash
kubectl apply -f manifests/07-ingress.yaml
```
Configure l'accès externe via NGINX Ingress.

</details>

### Commande complète

```bash
kubectl apply -f manifests/00-namespace.yaml && \
kubectl apply -f manifests/01-secrets.yaml && \
kubectl apply -f manifests/02-mysql-pvc.yaml && \
kubectl apply -f manifests/03-mysql-deployment.yaml && \
kubectl apply -f manifests/04-mysql-service.yaml && \
kubectl apply -f manifests/05-api-deployment.yaml && \
kubectl apply -f manifests/06-api-service.yaml && \
kubectl apply -f manifests/07-ingress.yaml
```

---

## ✅ Test du déploiement

### Health Check

```bash
curl http://localhost/kubernetes-brief/health
```

Réponse attendue :
```json
{
  "status": "healthy",
  "timestamp": "2024-01-16T10:30:00Z"
}
```

### Lister les clients

```bash
curl http://localhost/kubernetes-brief/clients
```

### Créer un client

```bash
curl -X POST http://localhost/kubernetes-brief/clients \
  -H "Content-Type: application/json" \
  -d '{"name": "Client Test", "email": "test@example.com"}'
```

---

## 🔍 Gestion et Monitoring

### Vérifier le statut

```bash
# Tous les pods
kubectl get pods

# Services
kubectl get svc

# Ingress
kubectl get ingress

# PersistentVolumeClaim
kubectl get pvc
```

### Logs

```bash
# Logs de l'API
kubectl logs -l app=api

# Logs de MySQL
kubectl logs -l app=mysql
```

### Description détaillée

```bash
kubectl describe pod <pod-name>
kubectl describe svc <service-name>
```

---

## 📁 Structure des manifests

```
manifests/
├── 00-namespace.yaml              # Namespace du projet
├── 01-secrets.yaml                # Identifiants MySQL
├── 02-mysql-pvc.yaml              # Volume persistant
├── 03-mysql-deployment.yaml       # Déploiement MySQL
├── 04-mysql-service.yaml          # Service MySQL
├── 05-api-deployment.yaml         # Déploiement API
├── 06-api-service.yaml            # Service API
└── 07-ingress.yaml                # Ingress NGINX
```

---

## 🔒 Bonnes pratiques de sécurité

- ✅ Identifiants stockés dans **Kubernetes Secrets**
- ✅ **Health Checks** pour la stabilité
- ✅ **Resource Limits** pour éviter les débordements
- ✅ **PersistentVolumeClaim** pour la persistance
- ✅ **Services** pour l'isolation réseau

---

## 🛠️ Troubleshooting

<details>
<summary><b>Les pods ne démarrent pas</b></summary>

```bash
# Vérifier le statut
kubectl get pods

# Voir les logs
kubectl logs <pod-name>

# Détails complets
kubectl describe pod <pod-name>
```

</details>

<details>
<summary><b>Problème de connexion API ↔ MySQL</b></summary>

```bash
# Tester la connectivité
kubectl exec -it <api-pod-name> -- nc -zv mysql-service 3306

# Vérifier les logs MySQL
kubectl logs -l app=mysql
```

</details>

<details>
<summary><b>L'API n'est pas accessible</b></summary>

```bash
# Vérifier les endpoints
kubectl get endpoints

# Vérifier l'Ingress
kubectl get ingress
kubectl describe ingress <ingress-name>
```

</details>

---

<div align="center">

**Déployé avec ❤️ sur Kubernetes**

[⬆ Retour au début](#-kubernetes--python-api--mysql)

</div>
