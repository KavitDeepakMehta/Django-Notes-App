# End-to-End Deployment of a Containerized Django Application on Kubernetes using KIND

## Aim

The aim of this project is to deploy a containerized Django-based Notes application on a Kubernetes cluster using **KIND (Kubernetes IN Docker)**. The objective is to understand the complete lifecycle of a Kubernetes workload—from cluster creation and container image management to application deployment, service exposure, and runtime access—while following Kubernetes best practices.

---

## Architecture Overview

![Image](https://github.com/user-attachments/assets/e086926b-088d-481f-a46d-7c760372810d)

---

## Problem Definition

Traditional application deployment methods tightly couple the application to a specific server, operating system, and runtime environment. This often leads to environment inconsistencies, manual recovery from failures, limited scalability, and complex deployment processes.

This project addresses the problem by designing a deployment approach that:

* Ensures environment consistency
* Enables self-healing and fault tolerance
* Decouples application runtime from underlying infrastructure
* Simplifies deployment and scaling
* Provides a foundation extendable to production-grade orchestration

Kubernetes solves these challenges through declarative configuration, container orchestration, automated recovery, and standardized networking.

---

## Components Used

* **Docker:** Containerizes the Django application to ensure consistent runtime across environments.
* **Docker Hub:** Acts as a remote container registry to store and distribute Docker images.
* **KIND (Kubernetes IN Docker):** Creates a local multi-node Kubernetes cluster for learning and testing.
* **Kubernetes Namespace:** Provides logical isolation and organization of application resources.
* **Kubernetes Deployment:** Manages application lifecycle with self-healing and replica control.
* **Kubernetes Service (ClusterIP):** Provides stable internal networking and service discovery.
* **kubectl:** CLI tool to deploy, manage, and monitor Kubernetes resources.
* **Port Forwarding:** Temporarily exposes Kubernetes services externally for testing.

---

## Step-by-Step Implementation

### STEP 1 — Install Docker

```bash
sudo apt-get update -y
sudo apt-get install -y docker.io
sudo usermod -aG docker $USER
```

> Log out and log back in after this step.

Verify:

```bash
docker --version
```

---

### STEP 2 — Install KIND

Check architecture:

```bash
uname -m
```

For **x86_64**:

```bash
curl -Lo kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-amd64
```

For **aarch64 (ARM)**:

```bash
curl -Lo kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-arm64
```

Install:

```bash
chmod +x kind
sudo mv kind /usr/local/bin/kind
```

Verify:

```bash
kind --version
```

---

### STEP 3 — Install kubectl

```bash
curl -Lo kubectl https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

---

### STEP 4 — Verify Setup

```bash
docker --version
kind --version
kubectl version --client --output=yaml
```

---

### STEP 5 — Create Kubernetes Cluster using KIND

```bash
mkdir kind-cluster && cd kind-cluster
```

Create `config.yml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.35.0
- role: worker
  image: kindest/node:v1.35.0
- role: worker
  image: kindest/node:v1.35.0
- role: worker
  image: kindest/node:v1.35.0
  extraPortMappings:
    - containerPort: 80
      hostPort: 80
    - containerPort: 443
      hostPort: 443
```

Create cluster:

```bash
kind create cluster --name kdm-cluster --config config.yml
kubectl get nodes
```

---

### STEP 6 — Clone Application & Build Image

```bash
git clone https://github.com/KavitDeepakMehta/Django-Notes-App.git
cd Django-Notes-App
git checkout dev
docker build -t notes-app-k8s .
docker images
```

---

### STEP 7 — Push Image to Docker Hub

```bash
docker login -u <DOCKERHUB-USERNAME>
docker image tag notes-app-k8s:latest itskdm14/notes-app-k8s:latest
docker push itskdm14/notes-app-k8s:latest
```

---

### STEP 8 — Create Kubernetes Manifests

```bash
mkdir k8s && cd k8s
```

**Namespace:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: notes-app
```

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-deployment
  namespace: notes-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: itskdm14/notes-app-k8s:latest
        ports:
        - containerPort: 8000
```

**Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: notes-app-service
  namespace: notes-app
spec:
  selector:
    app: notes-app
  ports:
    - port: 8000
      targetPort: 8000
  type: ClusterIP
```

---

### STEP 9 — Deploy to Kubernetes

```bash
kubectl apply -f namespace.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl get pods -n notes-app
kubectl get svc -n notes-app
```

---

### STEP 10 — Access the Application

```bash
sudo -E kubectl port-forward service/notes-app-service -n notes-app 8000:8000 --address=0.0.0.0
```

Access in browser:

```
http://<PUBLIC-IP>:8000
```

---

## Documentation Link :

---

## Conclusion

This project demonstrates an end-to-end Kubernetes deployment workflow using Docker, KIND, and Kubernetes core primitives. It provides hands-on understanding of container orchestration, declarative deployments, self-healing workloads, and Kubernetes networking, forming a strong foundation for production-grade Kubernetes deployments.
