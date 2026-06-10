# Express.js on Kubernetes

Deployed this project locally using Minikube on WSL2.

## What I did
- Cloned alexellis/expressjs-k8s
- Built Docker image inside Minikube using eval $(minikube docker-env)
- Deployed using kubectl and Helm
- Configured port-forwarding and accessed via WSL2 IP
- Practiced rolling updates and rollbacks

## Stack
- Node.js / Express.js
- Docker
- Kubernetes (Minikube)
- Helm
- WSL2 / Ubuntu
