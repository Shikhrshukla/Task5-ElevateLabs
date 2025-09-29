
# TASK 5 — Build a Kubernetes Cluster Locally with Minikube

## Project overview
This repository demonstrates how to run a simple NGINX app on a local Kubernetes cluster using Minikube.  
You will find two manifests in this folder:

- `deployment.yml` — Deployment for an `nginx:stable-alpine` pod with a readinessProbe.
- `service.yaml` — NodePort Service exposing the deployment.

This README includes:
- installation / setup steps (Ubuntu 24.04)
- exact commands used to start Minikube (recorded on this machine)
- how to deploy, verify, scale, and collect logs
- required screenshots to submit
- the contents of the provided YAML files (below)

---

## Environment (from this session)
```
minikube version: v1.35.0
commit: dd5d320e41b5451cdf3c01891bc4e13d189586ed-dirty

Minikube start output (abbreviated):
😄  minikube v1.35.0 on Ubuntu 24.04
✨  Using the docker driver based on existing profile
👍  Starting "minikube" primary control-plane node in "minikube" cluster
...
🐳  Preparing Kubernetes v1.32.0 on Docker 27.4.1 ...
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

minikube status:
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

---

## Prerequisites (Ubuntu 24.04)
- Docker installed and running.
- `kubectl` installed and on `PATH`.
- `minikube` installed and on `PATH`.
- A terminal with sudo access for installing packages.

> If you don't have these installed, you can install Docker, kubectl and minikube following official docs. This README assumes Docker + kubectl + minikube are already installed.

---

## Start Minikube (commands used)
```bash
# Start minikube using Docker driver
minikube start --driver=docker

# Verify
minikube status
kubectl config current-context   # should show 'minikube'
```

Example output from this session is included in the "Environment" section above.

---

## Deploy the app
From the folder that contains `deployment.yml` and `service.yaml`:

```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yaml

# Wait a few seconds and verify
kubectl get deployments
kubectl get pods -o wide
kubectl get svc
```

To open the service in a browser (minikube helper):
```bash
minikube service myserver-service
# OR get the URL directly
minikube service myserver-service --url
```

Alternative (port-forward):
```bash
kubectl port-forward svc/myserver-service 8080:80
# then open http://localhost:8080
```

---

## Inspect, logs and describe
```bash
# Describe a specific pod (replace <pod-name> with a real name)
kubectl describe pod <pod-name>

# Get logs from a pod
kubectl logs <pod-name>

# Get logs for pods using a label
kubectl logs -l env=test --all-containers=true
```

---

## Scaling
```bash
# Scale deployment to 3 replicas
kubectl scale deployment/demo-server --replicas=3

# Verify
kubectl get deployments
kubectl get pods -o wide
```

---

## Common debug commands
```bash
kubectl get all
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get nodes -o wide
kubectl describe svc myserver-service
```

---

## Required screenshots (how to capture)
You need to include the following screenshots with your submission:

1. `pods_YYYY-MM-DD.png` — Output of `kubectl get pods -o wide` (terminal screenshot).
2. `svc_YYYY-MM-DD.png` — Output of `kubectl get svc` (terminal screenshot).
3. `describe_pod_YYYY-MM-DD.png` — Output of `kubectl describe pod <pod-name>` showing events.
4. `browser_service_YYYY-MM-DD.png` — Browser screenshot showing the NGINX welcome page (from `minikube service` or when port-forwarded).

Tips:
- On Ubuntu you can copy terminal output into a text file with `kubectl get pods -o wide > pods.txt` and take a screenshot of the text editor or terminal window.
- You can also use `script` to record and save session output.

---

## `deployment.yml` (contents)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-server
  labels:
    env: test
spec:
  replicas: 1
  selector:
    matchLabels:
      env: test
  template:
    metadata:
      labels:
        env: test
    spec:
      containers:
        - name: demo-pod-nginx
          image: nginx:stable-alpine
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```

Notes:
- The `readinessProbe` performs an HTTP GET on `/` port `80` inside the container to determine readiness.
- If you see an error such as `initialDeplaySeconds` it usually indicates a typo and you must fix the field name to `initialDelaySeconds`.

---

## `service.yaml` (contents)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myserver-service
spec:
  type: NodePort
  selector:
    env: test
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

## Example kubectl outputs you can save (copy & paste these to your submission if you ran commands)
```
# kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
demo-server-xxxxx-xxxxx             1/1     Running   0          2m    172.17.0.7   minikube   <none>           <none>

# kubectl get svc
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
myserver-service NodePort   10.111.163.28  <none>        80:XXXXX/TCP   2m
```

---

## Troubleshooting
- `ImagePullBackOff`: check image name or pull secret.
- `CrashLoopBackOff`: `kubectl logs` and `kubectl describe pod` to investigate the exit reason.
- `BadRequest` when creating manifests: often caused by typos (e.g., `initialDeplaySeconds`) — fix the spelling.
- If service is not reachable try: `minikube service myserver-service --url` and `kubectl port-forward`.

---

## Submission checklist
- [ ] `deployment.yml`
- [ ] `service.yaml`
- [ ] `pods_YYYY-MM-DD.png`
- [ ] `svc_YYYY-MM-DD.png`
- [ ] `describe_pod_YYYY-MM-DD.png`
- [ ] `browser_service_YYYY-MM-DD.png`
- [ ] Short notes about any errors you encountered and how you fixed them

---

## Notes / Next steps
If you want, I can:
- generate a `Dockerfile` + a sample Node.js app and provide updated manifests that use your local image,
- produce annotated sample screenshots (PNG) showing exactly what to capture,
- create a `README.md` in your repo with these contents and a short CI suggestion.
