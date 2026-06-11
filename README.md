# 🚀 Deployment of Express.js App on Kubernetes

> A hands-on Kubernetes deployment project — containerizing a Node.js/Express.js microservice and orchestrating it on a local Minikube cluster running inside WSL2 on Windows.

---

## 👩‍💻 Author

**Khaula Azhar**
Aspiring Cloud & DevOps Engineer
🔗 [GitHub: Khaula-cloud](https://github.com/Khaula-cloud)

---

## 📖 Overview

This project demonstrates how to deploy a production-style Node.js microservice on Kubernetes from scratch using Minikube as the local cluster. It covers the full deployment lifecycle — from cloning an open-source Express.js app, building its Docker image, writing Kubernetes manifests, deploying via both `kubectl` and Helm, configuring services, and practicing rolling updates with zero downtime.

The base application is the open-source **[alexellis/expressjs-k8s](https://github.com/alexellis/expressjs-k8s)** — a minimal Express.js microservice purpose-built for Kubernetes learning. All infrastructure configuration, deployment steps, and operational work documented here was performed independently.

---

## 🏗️ Architecture

```
                         ┌─────────────────────────────────────────┐
                         │           Minikube Cluster               │
                         │                                          │
  Windows Browser  ──►   │   ┌──────────────┐                      │
  (via WSL2 IP)          │   │   Ingress /   │                      │
                         │   │ port-forward  │                      │
                         │   └──────┬───────┘                      │
                         │          │                               │
                         │   ┌──────▼───────┐                      │
                         │   │   Service     │  ClusterIP           │
                         │   │  (expressjs)  │                      │
                         │   └──────┬───────┘                      │
                         │          │  load balances                │
                         │   ┌──────▼──────────────────────┐       │
                         │   │        Deployment            │       │
                         │   │  ┌─────────┐  ┌─────────┐   │       │
                         │   │  │  Pod 1  │  │  Pod 2  │   │       │
                         │   │  │expressjs│  │expressjs│   │       │
                         │   │  │:8080    │  │:8080    │   │       │
                         │   │  └─────────┘  └─────────┘   │       │
                         │   └─────────────────────────────┘       │
                         │                                          │
                         │   Namespace: expressjs                   │
                         └─────────────────────────────────────────┘

  Host: WSL2 Ubuntu on Windows 11
  Local Docker daemon shared with Minikube via eval $(minikube docker-env)
```

---

## ✨ Features

- ✅ Containerized Node.js/Express.js app using a **non-root Docker user** (security best practice)
- ✅ Local Kubernetes cluster via **Minikube** running inside **WSL2**
- ✅ Deployment managed via both **raw kubectl manifests** and **Helm chart**
- ✅ **Readiness probe** on `/health` endpoint ensures traffic only routes to healthy pods
- ✅ **Rolling updates** with zero downtime — new pods start before old ones terminate
- ✅ **Instant rollback** with a single `kubectl rollout undo` command
- ✅ **Namespace isolation** — all resources scoped to the `expressjs` namespace
- ✅ Resource `requests` and `limits` set on every container
- ✅ Multi-replica deployment (2 replicas by default)
- ✅ Three live endpoints: `/` (HTML), `/links` (JSON), `/health` (status)

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| Node.js + Express.js | Application runtime and web framework |
| Docker | Containerization |
| Kubernetes (K8s) | Container orchestration |
| Minikube | Local Kubernetes cluster |
| Helm | Kubernetes package manager |
| kubectl | Kubernetes CLI |
| WSL2 (Ubuntu) | Linux environment on Windows |
| Git + GitHub | Version control and portfolio |

---

## 📁 Project Structure

```
expressjs-k8s/
│
├── Dockerfile              # Multi-stage Docker build, non-root user
├── index.js                # Express.js app entry point
├── package.json            # Node.js dependencies
├── routes/                 # Express route handlers
│   └── links.js
│
├── yaml/                   # Raw Kubernetes manifests
│   ├── deployment.yaml     # Deployment — replicas, image, probes, resources
│   └── service.yaml        # ClusterIP Service — internal load balancer
│
├── chart/                  # Helm chart
│   └── expressjs-k8s/
│       ├── Chart.yaml      # Chart metadata
│       ├── values.yaml     # Default values (image, replicas, resources)
│       └── templates/      # Templated K8s manifests
│
└── README.md               # This file
```

---

## ⚙️ Configuration

### Dockerfile highlights

```dockerfile
FROM node:21-alpine                           # Lightweight Alpine base
RUN addgroup -S app && adduser -S -g app app  # Non-root user
WORKDIR /home/app
COPY package.json ./
RUN npm i                                     # Dependencies cached as separate layer
COPY index.js ./
COPY routes routes
USER app                                      # Run as non-root
CMD ["node", "index.js"]                      # App listens on port 8080
```

### Kubernetes Deployment (key settings)

```yaml
replicas: 2                        # Two pods for high availability
imagePullPolicy: Never             # Use locally-built Minikube image
resources:
  requests: { cpu: 10m, memory: 128Mi }
  limits:   { cpu: 10m, memory: 128Mi }
readinessProbe:
  httpGet: { path: /health, port: 8080 }
  initialDelaySeconds: 5           # Wait 5s before first probe
```

### Helm values (values.yaml)

```yaml
replicaCount: 2
image:
  repository: expressjs-k8s
  tag: latest
  pullPolicy: Never
service:
  type: ClusterIP
  port: 8080
```

---

## 🚀 Deployment Guide

### Prerequisites

| Requirement | Install |
|-------------|---------|
| WSL2 + Ubuntu | Windows Features → WSL |
| Docker Desktop | [docker.com](https://docker.com) |
| Minikube | `brew install minikube` or [minikube.sigs.k8s.io](https://minikube.sigs.k8s.io) |
| kubectl | `brew install kubectl` |
| Helm | `brew install helm` |
| Git | `sudo apt install git` |

---

### Step 1 — Clone the repository

```bash
git clone https://github.com/Khaula-cloud/Deployment-of-expressjs-k8s-app-on-Kubernetes.git
cd Deployment-of-expressjs-k8s-app-on-Kubernetes
```

### Step 2 — Start Minikube

```bash
minikube start --driver=docker --cpus=2 --memory=3g
minikube addons enable ingress
kubectl get nodes   # confirm status is Ready
```

### Step 3 — Build Docker image inside Minikube

```bash
# Critical: point Docker CLI at Minikube's daemon
eval $(minikube docker-env)

# Build the image
docker build -t expressjs-k8s:latest .
docker images | grep expressjs   # confirm image exists
```

### Step 4 — Create namespace

```bash
kubectl create namespace expressjs
kubectl config set-context --current --namespace=expressjs
```

### Step 5A — Deploy via kubectl

```bash
kubectl apply -f yaml/ -n expressjs
kubectl get pods -n expressjs -w   # watch pods come up
```

### Step 5B — Deploy via Helm (alternative)

```bash
helm upgrade --install expressjs ./chart/expressjs-k8s/ \
  --namespace expressjs \
  --set image.repository=expressjs-k8s \
  --set image.tag=latest \
  --set image.pullPolicy=Never
```

### Step 6 — Access the app (WSL2)

```bash
# Port-forward bound to all interfaces
kubectl port-forward deploy/expressjs 7070:8080 -n expressjs --address 0.0.0.0
```

Get your WSL2 IP:

```bash
hostname -I | awk '{print $1}'
# Example output: 172.21.144.5
```

Open in Windows browser: **`http://172.21.144.5:7070`**

### Step 7 — Verify endpoints

```bash
curl http://localhost:7070/          # HTML homepage
curl http://localhost:7070/links     # JSON links response
curl http://localhost:7070/health    # Should return: OK
```

---

## 🔄 Rolling Updates & Rollback

```bash
# Build a new version
docker build -t expressjs-k8s:v2 .

# Trigger rolling update (zero downtime)
kubectl set image deployment/expressjs expressjs=expressjs-k8s:v2 -n expressjs
kubectl rollout status deployment/expressjs -n expressjs

# Rollback if something goes wrong
kubectl rollout undo deployment/expressjs -n expressjs

# View rollout history
kubectl rollout history deployment/expressjs -n expressjs
```

---

## 🔧 Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `ImagePullBackOff` | Docker image not in Minikube | Re-run `eval $(minikube docker-env)` then rebuild |
| `address already in use` on port-forward | Port taken by another process (e.g. Jenkins on 8080) | Use a different port: `kubectl port-forward ... 7070:8080` |
| Browser can't reach `localhost` | WSL2 has a separate network namespace from Windows | Use WSL2 IP (`hostname -I`) in Windows browser, or add `--address 0.0.0.0` |
| `src refspec main does not match any` | Branch is named `master` not `main` | Run `git push -u origin master` |
| Pods in `Pending` state | Insufficient Minikube resources | Restart with `minikube start --cpus=2 --memory=3g` |
| `CrashLoopBackOff` | App crashing on startup | Check logs: `kubectl logs -l app=expressjs -n expressjs` |

---

## 📚 Lessons Learned

1. **WSL2 networking is isolated from Windows** — `localhost` inside WSL2 is not the same as `localhost` in the Windows browser. Always use the WSL2 IP or `--address 0.0.0.0` with port-forward.

2. **`eval $(minikube docker-env)` must run before every build** — Without it, Docker builds into your local daemon, not Minikube's. The pod then fails with `ImagePullBackOff` because Kubernetes can't find the image.

3. **`imagePullPolicy: Never` is essential for local development** — It tells Kubernetes to never try pulling from a remote registry and use the locally available image instead.

4. **Rolling updates are the default and they're powerful** — Kubernetes replaces pods one by one, keeping the app live throughout. Combined with readiness probes, bad deployments are caught before they affect all traffic.

5. **Namespaces should be used from day one** — They keep resources organized and make cleanup (`kubectl delete namespace expressjs`) trivially easy.

6. **Helm vs raw YAML** — Raw YAML is easier to understand at first; Helm becomes essential when you need to deploy the same app to multiple environments (dev/staging/prod) with different configs.

7. **Port conflicts are common** — Tools like Jenkins, databases, and dev servers all grab common ports. Always check with `ss -tlnp` before port-forwarding.

---

## 🔮 Future Improvements

- [ ] Push Docker image to **Docker Hub** or **GitHub Container Registry** and remove `imagePullPolicy: Never`
- [ ] Add **Horizontal Pod Autoscaler (HPA)** to scale based on CPU usage
- [ ] Set up **Ingress with a custom domain** instead of port-forwarding
- [ ] Add **TLS/HTTPS** via cert-manager and Let's Encrypt
- [ ] Deploy to a real cloud cluster (**AWS EKS** or **GKE**)
- [ ] Set up a **CI/CD pipeline** with GitHub Actions to auto-build and deploy on every push
- [ ] Add **Prometheus + Grafana** for metrics and monitoring dashboards
- [ ] Implement **Kubernetes Secrets** for environment-based configuration
- [ ] Add **PodDisruptionBudget** to guarantee minimum availability during node maintenance

---

## 📜 Credits

Base application by **[Alex Ellis](https://github.com/alexellis)** — [alexellis/expressjs-k8s](https://github.com/alexellis/expressjs-k8s) (MIT License).
All Kubernetes deployment configuration, documentation, and operational work by **Khaula Azhar**.

---

<p align="center">Made with 💙 while learning Kubernetes on WSL2</p>
