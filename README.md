# TASK 5 - Build a Kubernetes Cluster Locally with Minikube

## Project overview
This repository demonstrates how to run a simple NGINX app on a local Kubernetes cluster using Minikube.  
You will find two manifests in this folder:

- `deployment.yml` - Deployment for an `nginx:stable-alpine` pod with a readinessProbe.
- `service.yaml` - NodePort Service exposing the deployment.

This README includes:
- installation / setup steps (Ubuntu 24.04)
- exact commands used to start Minikube (recorded on this machine)
- how to deploy, verify, scale, and collect logs
- required screenshots

---

## Prerequisites
- Linux System
- Docker installed and running.

> If you don't have Docker installed, you can install Docker from the official Docs. This README assumes Docker is already installed.

---

## MiniKube & Kubectl Installation

Kubectl
```bash
# curl method (recommended so you can pick a version)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
or
```bash
# OR install via snap:
# sudo snap install kubectl --classic
```
MiniKube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```
---

## Start Minikube (commands used)
```bash
# Start minikube using Docker driver
minikube start --driver=docker

# Verify
minikube status
kubectl config current-context   # should show 'minikube'
```
### Output
<img width="1920" height="1080" alt="Screenshot from 2025-09-29 15-06-24" src="https://github.com/user-attachments/assets/95f1fd26-a92f-4188-93e5-2f61976e967d" />

---

## Deploy the app
From the folder that contains `deployment.yml` and `service.yaml`:

```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yaml

# Wait a few seconds and verify
kubectl get deployments
kubectl get pods
kubectl get svc -o wide
kubectl get nodes -o wide
```
#### Optional if needed
Alternative (port-forward):
```bash
kubectl port-forward svc/myserver-service 8080:80
# then open http://localhost:8080
```
### Output
<img width="1920" height="1080" alt="Screenshot from 2025-09-29 16-17-43" src="https://github.com/user-attachments/assets/9d5eee02-b294-4d5e-bdcd-a2a7e5be7d73" />
<img width="1920" height="1080" alt="Screenshot from 2025-09-29 16-21-59" src="https://github.com/user-attachments/assets/a334c3dd-fdf9-4d74-ba0c-0d6d0fa08f5d" />

---

## Inspect, logs and describe
```bash
# Describe a specific pod (replace <pod-name> with a real name)
kubectl describe pod <pod-name>

# Get logs from a pod
kubectl logs <pod-name>
```
OR
```bash
# Get logs for pods using a label
kubectl logs -l env=test
```

### Output
<img width="1920" height="1080" alt="Screenshot from 2025-09-29 16-22-34" src="https://github.com/user-attachments/assets/39ba1d57-5604-4e4a-8b5a-090a7aed7963" />
<img width="1920" height="1080" alt="Screenshot from 2025-09-29 16-23-51" src="https://github.com/user-attachments/assets/deffa0b7-aeb4-4ce4-b7ee-082969e7324d" />


---

## Scaling
```bash
# Scale deployment to 3 replicas
kubectl scale deployment demo-server --replicas=3

# Verify
kubectl get pods
kubectl get deployments
```

### Output
<img width="1920" height="1080" alt="Screenshot from 2025-09-29 16-24-43" src="https://github.com/user-attachments/assets/0ecd5022-ed67-48b7-be0b-076e110d4929" />

---

## Common debug commands
```bash
kubectl get all
kubectl get nodes -o wide
kubectl describe svc myserver-service
kubectl logs <pod name>
```

---

## Troubleshooting
- `ImagePullBackOff`: check image name or pull secret.
- `CrashLoopBackOff`: `kubectl logs` and `kubectl describe pod` to investigate the exit reason.
- `BadRequest` when creating manifests: often caused by typos (e.g., `initialDeplaySeconds`) - fix the spelling.
- If service is not reachable try: `minikube service myserver-service --url` and `kubectl port-forward`.

---
