# Kubernetes Mastery Roadmap (Beginner → Advanced)

> A complete, hands-on guide to mastering Kubernetes — from core concepts to production-grade deployments.

---

## Table of Contents

1. [What is Kubernetes](#1-what-is-kubernetes)
2. [Kubernetes Core Concepts](#2-kubernetes-core-concepts)
3. [Installing Kubernetes for Learning](#3-installing-kubernetes-for-learning)
4. [kubectl Command Cheat Sheet](#4-kubectl-command-cheat-sheet)
5. [Pods Deep Dive](#5-pods-deep-dive)
6. [Deployments](#6-deployments)
7. [Services](#7-services)
8. [Ingress](#8-ingress)
9. [ConfigMaps and Secrets](#9-configmaps-and-secrets)
10. [Volumes and Persistent Storage](#10-volumes-and-persistent-storage)
11. [Kubernetes Networking](#11-kubernetes-networking)
12. [Auto Scaling](#12-auto-scaling)
13. [Monitoring and Logging](#13-monitoring-and-logging)
14. [CI/CD with Kubernetes](#14-cicd-with-kubernetes)
15. [Kubernetes Security](#15-kubernetes-security)
16. [Production Best Practices](#16-production-best-practices)
17. [Troubleshooting Kubernetes](#17-troubleshooting-kubernetes)
18. [Real World Project — MERN Stack](#18-real-world-project--mern-stack)
19. [Advanced Kubernetes](#19-advanced-kubernetes)
20. [Kubernetes Learning Roadmap](#20-kubernetes-learning-roadmap)

---

## 1. What is Kubernetes

### Overview

Kubernetes (often abbreviated as **K8s**) is an open-source container orchestration platform originally developed by Google, based on their internal system called **Borg**. It was donated to the Cloud Native Computing Foundation (CNCF) in 2014 and has since become the industry standard for deploying, scaling, and managing containerized applications.

In simple terms: **Docker packages your app into containers. Kubernetes runs and manages those containers at scale.**

### Why Was Kubernetes Created?

Before container orchestration, teams faced significant operational challenges:

- **Manual scaling**: Spinning up more servers by hand when traffic spiked.
- **Single points of failure**: One crashed server could take down the whole app.
- **No self-healing**: If a container died, someone had to manually restart it.
- **Complex deployments**: Rolling out updates without downtime was error-prone.
- **Resource waste**: Servers were either over-provisioned (expensive) or under-provisioned (risky).

Kubernetes solves all of these problems automatically.

### Real-World Problems Kubernetes Solves

| Problem | Kubernetes Solution |
|---|---|
| App container crashes | Automatically restarts the container |
| Traffic spike | Horizontally scales pods up automatically |
| Deploy new version without downtime | Rolling updates with zero downtime |
| Multiple services need isolation | Namespaces and network policies |
| Config varies between environments | ConfigMaps and Secrets |
| Need storage across container restarts | PersistentVolumes |
| Health checks on services | Liveness and Readiness probes |

### Container Orchestration

Container orchestration is the automated management of the lifecycle of containers, including:

- **Scheduling** — deciding which node runs which container
- **Scaling** — adding or removing container replicas
- **Networking** — routing traffic between containers
- **Storage** — attaching persistent storage to containers
- **Health monitoring** — restarting failed containers
- **Rolling updates and rollbacks** — safe deployments

### Kubernetes Architecture

```
                     ┌─────────────────────────────────────────┐
                     │            CONTROL PLANE                 │
                     │                                          │
                     │  ┌──────────────┐   ┌────────────────┐  │
         kubectl ───────►  API Server  │   │      etcd      │  │
                     │  └──────┬───────┘   └────────────────┘  │
                     │         │                                 │
                     │  ┌──────┴───────┐   ┌────────────────┐  │
                     │  │  Controller  │   │   Scheduler    │  │
                     │  │  Manager     │   │                │  │
                     │  └──────────────┘   └────────────────┘  │
                     └─────────────────────────────────────────┘
                                      │
                    ┌─────────────────┼──────────────────┐
                    │                 │                   │
             ┌──────┴──────┐  ┌──────┴──────┐  ┌────────┴────┐
             │   Worker    │  │   Worker    │  │   Worker   │
             │   Node 1    │  │   Node 2    │  │   Node 3   │
             │             │  │             │  │            │
             │  ┌───────┐  │  │  ┌───────┐  │  │  ┌──────┐ │
             │  │kubelet│  │  │  │kubelet│  │  │  │kubelet│ │
             │  └───────┘  │  │  └───────┘  │  │  └──────┘ │
             │  ┌───────┐  │  │  ┌───────┐  │  │  ┌──────┐ │
             │  │kube-  │  │  │  │kube-  │  │  │  │kube- │ │
             │  │proxy  │  │  │  │proxy  │  │  │  │proxy │ │
             │  └───────┘  │  │  └───────┘  │  │  └──────┘ │
             │  ┌────────┐ │  │  ┌────────┐ │  │  ┌──────┐ │
             │  │ Pods   │ │  │  │ Pods   │ │  │  │ Pods │ │
             │  └────────┘ │  │  └────────┘ │  │  └──────┘ │
             └─────────────┘  └─────────────┘  └────────────┘
```

### Component Explanations

#### Control Plane Components

**API Server (`kube-apiserver`)**
The front door to the entire Kubernetes cluster. All communication — whether from `kubectl`, internal components, or external systems — goes through the API Server. It validates and processes REST requests, then updates the cluster state in etcd.

**etcd**
A distributed, reliable key-value store that holds the entire cluster state. Think of it as Kubernetes' brain/database. If etcd is lost, the cluster loses all state. It must be backed up regularly in production.

**Controller Manager (`kube-controller-manager`)**
Runs a collection of controllers that watch the state of the cluster and make changes to bring the actual state toward the desired state. For example, the ReplicaSet controller ensures the correct number of pods is always running.

**Scheduler (`kube-scheduler`)**
Watches for newly created pods that have no assigned node and assigns them to the most suitable node based on resource availability, affinity rules, taints/tolerations, and other policies.

#### Worker Node Components

**kubelet**
An agent that runs on every worker node. It receives pod specifications from the API Server and ensures the containers described in those specs are running and healthy.

**kube-proxy**
Maintains network rules on each node using iptables or IPVS. It enables the Service abstraction — routing traffic to the correct pods regardless of which node they're on.

**Container Runtime**
The software responsible for running containers. Kubernetes supports containerd, CRI-O, and Docker (via shim). containerd is the most common in modern clusters.

---

## 2. Kubernetes Core Concepts

### Cluster

A Kubernetes **cluster** is a set of machines (nodes) that run containerized applications managed by Kubernetes. It consists of at least one control plane and one or more worker nodes.

```
Kubernetes Cluster
├── Control Plane (master)
│   ├── API Server
│   ├── etcd
│   ├── Scheduler
│   └── Controller Manager
└── Worker Nodes
    ├── Node 1
    ├── Node 2
    └── Node 3
```

### Node

A **node** is a physical or virtual machine in the cluster. Each node can run multiple pods. Nodes are managed by the control plane.

```bash
# View all nodes
kubectl get nodes

# Get detailed info about a node
kubectl describe node <node-name>
```

### Pod

A **Pod** is the smallest deployable unit in Kubernetes. A pod wraps one or more containers that share:
- The same network namespace (same IP address)
- The same storage volumes
- The same lifecycle

Pods are **ephemeral** — they can be created, destroyed, and replaced at any time.

```
Pod
├── Container 1 (e.g., nginx)
├── Container 2 (e.g., log sidecar)
└── Shared Volume
```

### Container

Containers inside a pod share the pod's IP address and communicate via `localhost`. Each container has its own filesystem, but volumes can be shared between them.

### Namespace

**Namespaces** provide a way to logically divide a cluster into virtual sub-clusters. They're useful for:
- Separating environments (dev, staging, prod)
- Multi-team isolation
- Resource quota management

```bash
# View all namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace my-app

# Run commands in a specific namespace
kubectl get pods -n my-app
```

Default namespaces in a fresh cluster:
- `default` — where resources go if no namespace is specified
- `kube-system` — internal Kubernetes components
- `kube-public` — publicly readable resources
- `kube-node-lease` — node heartbeat information

### Labels

**Labels** are key-value pairs attached to Kubernetes objects. They're used for grouping, selecting, and filtering objects.

```yaml
metadata:
  labels:
    app: nginx
    environment: production
    version: "1.4"
```

Labels are used by Services and Deployments to select target pods via **label selectors**.

### Annotations

**Annotations** are also key-value pairs but meant for non-identifying metadata — things that tools and humans read, but Kubernetes doesn't use for selection.

```yaml
metadata:
  annotations:
    description: "Main web server pod"
    owner: "platform-team"
    prometheus.io/scrape: "true"
```

### Core YAML Example — A Pod

```yaml
apiVersion: v1          # Kubernetes API version
kind: Pod               # Resource type
metadata:
  name: nginx-pod       # Name of this pod
  namespace: default    # Which namespace it lives in
  labels:
    app: web            # Label for selecting this pod
    tier: frontend
spec:                   # Desired state specification
  containers:
  - name: nginx         # Container name
    image: nginx:1.25   # Docker image to use
    ports:
    - containerPort: 80 # Port the container listens on
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**Line-by-line explanation:**

| Field | Purpose |
|---|---|
| `apiVersion` | Specifies which Kubernetes API to use |
| `kind` | The type of resource (Pod, Deployment, Service, etc.) |
| `metadata.name` | Unique name within the namespace |
| `metadata.labels` | Key-value labels for grouping/selection |
| `spec.containers` | List of containers to run in this pod |
| `containers[].name` | Internal name for the container |
| `containers[].image` | Docker image (from a registry) |
| `containers[].ports` | Ports the container exposes |
| `resources.requests` | Minimum resources the pod needs |
| `resources.limits` | Maximum resources the pod can use |

### Mini Exercise 1

```bash
# 1. Create the pod
kubectl apply -f nginx-pod.yaml

# 2. Check it's running
kubectl get pods

# 3. Describe it
kubectl describe pod nginx-pod

# 4. Delete it
kubectl delete pod nginx-pod
```

---

## 3. Installing Kubernetes for Learning

### Option 1: Minikube (Recommended for Beginners)

Minikube runs a single-node Kubernetes cluster in a VM or container on your local machine.

**Install Minikube:**
```bash
# macOS
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Windows (via Chocolatey)
choco install minikube
```

**Start and use Minikube:**
```bash
# Start cluster
minikube start

# Start with specific Kubernetes version
minikube start --kubernetes-version=v1.28.0

# Start with more resources
minikube start --cpus=4 --memory=8192

# Check cluster info
kubectl cluster-info

# View all pods in all namespaces
kubectl get pods -A

# Access Kubernetes dashboard
minikube dashboard

# Stop cluster
minikube stop

# Delete cluster
minikube delete
```

### Option 2: Kind (Kubernetes IN Docker)

Kind runs Kubernetes nodes as Docker containers. Great for CI/CD and multi-node testing.

```bash
# Install kind
# macOS/Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create a cluster
kind create cluster

# Create multi-node cluster
cat > kind-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --config kind-config.yaml

# Delete cluster
kind delete cluster
```

### Option 3: Docker Desktop Kubernetes

Docker Desktop (Mac/Windows) includes built-in Kubernetes support.

1. Open Docker Desktop → Settings → Kubernetes
2. Check "Enable Kubernetes"
3. Click "Apply & Restart"
4. Wait for the Kubernetes indicator to turn green

```bash
# Verify it's running
kubectl config use-context docker-desktop
kubectl get nodes
```

### Verifying Your Installation

```bash
# Check kubectl version
kubectl version --client

# Check cluster nodes
kubectl get nodes

# Check all system pods
kubectl get pods -n kube-system

# Cluster information
kubectl cluster-info
```

---

## 4. kubectl Command Cheat Sheet

`kubectl` is the command-line tool for interacting with Kubernetes clusters.

### Basic Syntax

```
kubectl [command] [TYPE] [NAME] [flags]
```

### Getting Resources

```bash
# List pods in current namespace
kubectl get pods

# List pods in all namespaces
kubectl get pods -A

# List pods with extra info (IP, node, etc.)
kubectl get pods -o wide

# List pods and watch for changes
kubectl get pods -w

# List all resource types
kubectl get all

# Get pods in a specific namespace
kubectl get pods -n kube-system

# Get a specific pod
kubectl get pod nginx-pod

# Output as YAML
kubectl get pod nginx-pod -o yaml

# Output as JSON
kubectl get pod nginx-pod -o json

# Get nodes
kubectl get nodes

# Get services
kubectl get svc

# Get deployments
kubectl get deployments

# Get configmaps
kubectl get configmaps

# Get secrets
kubectl get secrets

# Get persistent volumes
kubectl get pv

# Get persistent volume claims
kubectl get pvc
```

### Describing Resources (Detailed Info)

```bash
# Describe a pod (shows events, status, container info)
kubectl describe pod POD_NAME

# Describe a node
kubectl describe node NODE_NAME

# Describe a service
kubectl describe svc SERVICE_NAME

# Describe a deployment
kubectl describe deployment DEPLOYMENT_NAME
```

### Creating and Applying Resources

```bash
# Apply a YAML file (create or update)
kubectl apply -f file.yaml

# Apply all YAML files in a directory
kubectl apply -f ./manifests/

# Create resource directly
kubectl create deployment nginx --image=nginx

# Create a namespace
kubectl create namespace staging
```

### Deleting Resources

```bash
# Delete a pod
kubectl delete pod POD_NAME

# Delete using YAML file
kubectl delete -f file.yaml

# Delete all pods in a namespace
kubectl delete pods --all -n staging

# Delete a deployment
kubectl delete deployment DEPLOYMENT_NAME

# Force delete a stuck pod
kubectl delete pod POD_NAME --grace-period=0 --force
```

### Logs and Debugging

```bash
# View pod logs
kubectl logs POD_NAME

# Follow logs in real time
kubectl logs -f POD_NAME

# Logs from a specific container in a multi-container pod
kubectl logs POD_NAME -c CONTAINER_NAME

# Previous container logs (after crash)
kubectl logs POD_NAME --previous

# Execute a command inside a running container
kubectl exec -it POD_NAME -- bash

# Execute in a specific container
kubectl exec -it POD_NAME -c CONTAINER_NAME -- sh

# Run a one-off debug pod
kubectl run debug --image=busybox --rm -it -- sh
```

### Port Forwarding

```bash
# Forward pod port to local
kubectl port-forward POD_NAME 8080:80

# Forward service port to local
kubectl port-forward svc/my-service 8080:80

# Forward deployment port
kubectl port-forward deployment/nginx-deployment 8080:80
```

### Scaling and Rollouts

```bash
# Scale a deployment
kubectl scale deployment nginx-deployment --replicas=5

# Check rollout status
kubectl rollout status deployment nginx-deployment

# Rollout history
kubectl rollout history deployment nginx-deployment

# Undo last rollout
kubectl rollout undo deployment nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment nginx-deployment --to-revision=2

# Pause a rollout
kubectl rollout pause deployment nginx-deployment

# Resume a paused rollout
kubectl rollout resume deployment nginx-deployment
```

### Context and Config

```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context my-cluster

# Set default namespace for current context
kubectl config set-context --current --namespace=my-app

# View kubeconfig
kubectl config view
```

### Resource Usage

```bash
# CPU and memory usage for pods
kubectl top pods

# CPU and memory for nodes
kubectl top nodes

# Top pods in specific namespace
kubectl top pods -n production
```

### Quick Reference Table

| Task | Command |
|---|---|
| List pods | `kubectl get pods` |
| Describe pod | `kubectl describe pod NAME` |
| View logs | `kubectl logs NAME` |
| Exec into pod | `kubectl exec -it NAME -- bash` |
| Apply YAML | `kubectl apply -f file.yaml` |
| Delete resource | `kubectl delete -f file.yaml` |
| Scale deployment | `kubectl scale deploy NAME --replicas=N` |
| Port forward | `kubectl port-forward NAME 8080:80` |
| Rollback | `kubectl rollout undo deploy NAME` |
| View events | `kubectl get events --sort-by='.lastTimestamp'` |

---

## 5. Pods Deep Dive

### Pod Lifecycle

A pod goes through several phases during its lifetime:

```
┌──────────┐    ┌─────────┐    ┌──────────┐    ┌──────────────┐
│ Pending  │───►│ Running │───►│Succeeded │    │   Failed     │
└──────────┘    └─────────┘    └──────────┘    └──────────────┘
                     │                               ▲
                     └───────────────────────────────┘
                          (container crashes)

Also: Unknown (node communication lost)
```

| Phase | Meaning |
|---|---|
| `Pending` | Pod accepted but containers not yet started (scheduling, image pull) |
| `Running` | At least one container is running |
| `Succeeded` | All containers exited with code 0 |
| `Failed` | At least one container exited with non-zero code |
| `Unknown` | Pod state cannot be determined |

### Container States

Within a running pod, each container has its own state:
- **Waiting** — not yet running (pulling image, waiting for secrets)
- **Running** — container is executing
- **Terminated** — container finished execution

### Pod Hierarchy

```
User / CI Pipeline
        │
        ▼
   Deployment        ← Manages desired state
        │
        ▼
   ReplicaSet        ← Ensures N replicas of a pod template
        │
        ▼
      Pod            ← Unit of deployment
        │
        ▼
   Container(s)      ← Your actual application
```

### Restart Policies

```yaml
spec:
  restartPolicy: Always   # Always, OnFailure, Never
```

| Policy | Behavior |
|---|---|
| `Always` | Restart regardless of exit code (default for Deployments) |
| `OnFailure` | Only restart on non-zero exit |
| `Never` | Never restart |

### Multi-Container Pods

Use multiple containers in a pod when they are tightly coupled and need to share resources.

**Sidecar Pattern** — a helper container alongside the main app:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  - name: app
    image: my-app:1.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app

  - name: log-shipper      # Sidecar reads and ships logs
    image: fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app

  volumes:
  - name: shared-logs
    emptyDir: {}            # Temporary volume shared between containers
```

**Ambassador Pattern** — a proxy container:

```yaml
  containers:
  - name: app
    image: my-app:1.0
  - name: ambassador
    image: envoyproxy/envoy:latest
    # Handles all network traffic for the main app
```

### Init Containers

Init containers run **before** the main containers start. They must complete successfully before the pod proceeds. Use them for:
- Waiting for a service (DB, cache) to be ready
- Running migrations
- Downloading config files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nslookup postgres; do echo waiting; sleep 2; done']

  - name: run-migrations
    image: my-app:migrate
    command: ['sh', '-c', 'python manage.py migrate']

  containers:
  - name: app
    image: my-app:1.0
    ports:
    - containerPort: 8000
```

### Pod Affinity and Anti-Affinity

Control where pods are scheduled relative to other pods:

```yaml
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - web-frontend
          topologyKey: kubernetes.io/hostname
```

This spreads frontend pods across different nodes for high availability.

### Mini Exercise 2 — Multi-Container Pod

```bash
# Create and observe a multi-container pod
cat > multi-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: multi-container
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do echo hello; sleep 10; done']
EOF

kubectl apply -f multi-pod.yaml

# Exec into the nginx container specifically
kubectl exec -it multi-container -c nginx -- bash

# View logs from busybox
kubectl logs multi-container -c busybox
```

---

## 6. Deployments

A **Deployment** is the standard way to run stateless applications in Kubernetes. It manages a ReplicaSet, which in turn manages a set of identical pods.

### Why Use Deployments?

- Declare how many pod replicas you want
- Kubernetes ensures that number is maintained
- Supports rolling updates and rollbacks
- Self-heals if pods crash

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                      # Run 3 copies
  selector:
    matchLabels:
      app: nginx                   # Manages pods with this label
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1            # Max pods unavailable during update
      maxSurge: 1                  # Max extra pods during update
  template:                        # Pod template
    metadata:
      labels:
        app: nginx                 # Must match selector.matchLabels
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

### Deployment Commands

```bash
# Apply the deployment
kubectl apply -f deployment.yaml

# Check deployment status
kubectl get deployments

# Detailed deployment info
kubectl describe deployment nginx-deployment

# Watch rollout in real time
kubectl rollout status deployment nginx-deployment

# Scale up
kubectl scale deployment nginx-deployment --replicas=5

# Scale down
kubectl scale deployment nginx-deployment --replicas=2

# Update image (triggers rolling update)
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

# View rollout history
kubectl rollout history deployment nginx-deployment

# Rollback to previous version
kubectl rollout undo deployment nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment nginx-deployment --to-revision=2
```

### Rolling Updates

Rolling updates replace pods gradually, ensuring some are always available:

```
Before Update:
Pod v1.0 | Pod v1.0 | Pod v1.0

During Update (maxSurge=1, maxUnavailable=1):
Pod v1.0 | Pod v1.0 | Pod v2.0 | Pod v2.0  ← one new created
Pod v1.0 |           | Pod v2.0 | Pod v2.0  ← one old removed
          | Pod v2.0 | Pod v2.0 | Pod v2.0  ← last old removed

After Update:
Pod v2.0 | Pod v2.0 | Pod v2.0
```

### ReplicaSets

A Deployment creates and manages a ReplicaSet. You rarely interact with ReplicaSets directly, but knowing they exist helps with debugging.

```bash
# See the ReplicaSet created by a Deployment
kubectl get replicasets

# When you update a deployment, a new RS is created and old one scaled to 0
kubectl get rs
# nginx-deployment-abc123   3   3   3  (current)
# nginx-deployment-def456   0   0   0  (previous - kept for rollback)
```

### Mini Exercise 3 — Deployment

```bash
# 1. Deploy nginx
kubectl apply -f deployment.yaml

# 2. Watch 3 pods come up
kubectl get pods -w

# 3. Scale to 5
kubectl scale deployment nginx-deployment --replicas=5

# 4. Update the image
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

# 5. Watch the rolling update
kubectl rollout status deployment nginx-deployment

# 6. Rollback
kubectl rollout undo deployment nginx-deployment
```

---

## 7. Services

A **Service** provides a stable network endpoint for a set of pods. Since pods are ephemeral and their IPs change, a Service gives them a consistent DNS name and IP.

### Why Services Are Needed

```
Without Service:
Client → Pod IP (10.1.0.1) ← Pod dies, IP changes!

With Service:
Client → Service (my-service:80) → Pods (any healthy pod)
```

### Service Types

#### ClusterIP (Default)

Only accessible within the cluster. Used for internal communication between services.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx              # Routes traffic to pods with this label
  ports:
  - protocol: TCP
    port: 80                # Service port
    targetPort: 80          # Pod container port
```

```
Internal Pod → ClusterIP Service → Target Pods
(10.96.x.x — virtual IP, load balances across matching pods)
```

#### NodePort

Exposes the service on a static port on every node. Accessible externally via `NodeIP:NodePort`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080         # Must be 30000-32767
```

```
Internet
    │
    ▼
Node IP:30080 (any node)
    │
    ▼
  Service
    │
    ├── Pod 1
    ├── Pod 2
    └── Pod 3
```

#### LoadBalancer

Creates an external load balancer in cloud environments (AWS ELB, GCP LB, Azure LB). On bare metal, you need MetalLB.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```
Internet
    │
    ▼
External Load Balancer (cloud-provisioned)
    │
    ▼
  Service
    │
    ├── Pod 1
    ├── Pod 2
    └── Pod 3
```

#### ExternalName

Maps a service to a DNS name (useful for accessing external services from within the cluster):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: my-database.example.com
```

### Service Discovery with DNS

Kubernetes includes a DNS server (CoreDNS). Every service gets a DNS entry:

```
<service-name>.<namespace>.svc.cluster.local

Example:
nginx-service.default.svc.cluster.local
postgres.database.svc.cluster.local
```

From within a pod in the same namespace:
```bash
curl http://nginx-service        # short form
curl http://nginx-service.default.svc.cluster.local  # full form
```

### Headless Services

A headless service (ClusterIP: None) returns DNS records for individual pod IPs — useful for StatefulSets:

```yaml
spec:
  clusterIP: None
  selector:
    app: my-statefulset
```

### Mini Exercise 4 — Services

```bash
# Deploy nginx
kubectl create deployment nginx --image=nginx --replicas=3

# Expose as ClusterIP
kubectl expose deployment nginx --port=80 --target-port=80

# Check service
kubectl get svc

# Port-forward to test locally
kubectl port-forward svc/nginx 8080:80
# Now open http://localhost:8080

# Expose as NodePort instead
kubectl expose deployment nginx --type=NodePort --port=80 --name=nginx-np

# Get the NodePort
kubectl get svc nginx-np
# Access at http://$(minikube ip):NodePort
```

---

## 8. Ingress

### What is Ingress?

An **Ingress** is a Kubernetes resource that manages external HTTP/HTTPS access to services in a cluster. It provides:

- **Host-based routing**: `api.example.com` → Service A, `app.example.com` → Service B
- **Path-based routing**: `/api` → backend service, `/` → frontend service
- **TLS termination**: SSL certificates
- **Load balancing**

### Ingress vs. LoadBalancer Service

| Feature | LoadBalancer Service | Ingress |
|---|---|---|
| Cost | 1 LB per service (expensive) | 1 LB for all services |
| Routing | IP-based | Host/path-based |
| SSL termination | Manual | Built-in |
| Typical use | Simple services | Multiple services behind one IP |

### Architecture

```
Internet
    │
    ▼
┌─────────────────────────────────┐
│       Ingress Controller        │
│     (nginx, traefik, etc.)      │
└──────────────┬──────────────────┘
               │
               ├── /api  ──────► api-service:8080 → API Pods
               │
               ├── /      ──────► web-service:80   → Web Pods
               │
               └── admin.example.com ► admin-svc   → Admin Pods
```

### Installing an Ingress Controller (Nginx)

```bash
# For Minikube
minikube addons enable ingress

# For other clusters
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

# Verify installation
kubectl get pods -n ingress-nginx
```

### Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls-secret    # TLS certificate stored in a Secret

  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080

      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

### TLS with Cert-Manager

```bash
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Create a ClusterIssuer for Let's Encrypt
cat > cluster-issuer.yaml <<'EOF'
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: you@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
kubectl apply -f cluster-issuer.yaml
```

---

## 9. ConfigMaps and Secrets

### ConfigMaps

A **ConfigMap** stores non-sensitive configuration data as key-value pairs. Use them to decouple configuration from container images.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_PORT: "8080"
  LOG_LEVEL: "info"
  database.properties: |
    host=postgres
    port=5432
    name=myapp
```

#### Using ConfigMaps in Pods

**As environment variables:**
```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    envFrom:
    - configMapRef:
        name: app-config        # Inject all keys as env vars
```

**As a mounted file:**
```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### Secrets

A **Secret** stores sensitive data (passwords, API keys, TLS certs). Data is base64-encoded (not encrypted by default — enable encryption at rest in production).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_USER: cG9zdGdyZXM=      # base64("postgres")
  DB_PASS: c3VwZXJzZWNyZXQ=  # base64("supersecret")
```

**Create secret from command line:**
```bash
# Creates and base64-encodes automatically
kubectl create secret generic db-secret \
  --from-literal=DB_USER=postgres \
  --from-literal=DB_PASS=supersecret

# From file
kubectl create secret generic tls-secret \
  --from-file=tls.crt=./server.crt \
  --from-file=tls.key=./server.key

# View decoded secret
kubectl get secret db-secret -o jsonpath='{.data.DB_PASS}' | base64 --decode
```

**Using Secrets in Pods:**
```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_PASS
    envFrom:
    - secretRef:
        name: db-secret
```

### Secret Types

| Type | Usage |
|---|---|
| `Opaque` | Arbitrary data (default) |
| `kubernetes.io/tls` | TLS certificates |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials |
| `kubernetes.io/service-account-token` | Service account tokens |

### Best Practices for Secrets

- Enable encryption at rest in production
- Use external secret managers: **Vault**, **AWS Secrets Manager**, **Azure Key Vault**
- Use **External Secrets Operator** to sync from external stores
- Never commit secrets to Git
- Use RBAC to restrict who can access secrets

---

## 10. Volumes and Persistent Storage

### The Problem

Containers are stateless by default. If a container restarts, its filesystem is wiped. For databases and file storage, you need persistence.

### Volume Types

#### emptyDir

Temporary storage tied to pod lifetime. Shared between containers in a pod. Deleted when pod is deleted.

```yaml
volumes:
- name: temp-data
  emptyDir: {}
```

#### hostPath

Mounts a directory from the host node's filesystem. Use with caution — creates node affinity.

```yaml
volumes:
- name: host-volume
  hostPath:
    path: /data
    type: Directory
```

#### PersistentVolume (PV) — The Preferred Way

A PV is a piece of storage in the cluster provisioned by an admin or dynamically by a StorageClass.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce          # RWO: One node at a time
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data        # In production: NFS, EBS, GCE PD, etc.
```

#### PersistentVolumeClaim (PVC)

A PVC is a request for storage from a pod. Kubernetes binds a PVC to a matching PV.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 5Gi         # Requests 5Gi from the PV pool
```

**Using PVC in a Pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres-pod
spec:
  containers:
  - name: postgres
    image: postgres:15
    env:
    - name: POSTGRES_PASSWORD
      value: "mysecretpassword"
    volumeMounts:
    - mountPath: /var/lib/postgresql/data
      name: postgres-storage
  volumes:
  - name: postgres-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

### Storage Architecture

```
Pod
 │
 │ mounts
 ▼
PersistentVolumeClaim (PVC)
 │
 │ bound to
 ▼
PersistentVolume (PV)
 │
 │ backed by
 ▼
Physical Storage
(EBS, NFS, GCE Persistent Disk, Azure Disk, etc.)
```

### StorageClass

StorageClasses enable **dynamic provisioning** — PVs are automatically created when a PVC is submitted.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs    # Cloud provider provisioner
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
```

### Access Modes

| Mode | Short | Description |
|---|---|---|
| ReadWriteOnce | RWO | Mount as read-write by a single node |
| ReadOnlyMany | ROX | Mount as read-only by many nodes |
| ReadWriteMany | RWX | Mount as read-write by many nodes |

---

## 11. Kubernetes Networking

### The Networking Model

Kubernetes follows these fundamental rules:
1. Every pod gets its own unique IP address
2. All pods can communicate with all other pods without NAT
3. All nodes can communicate with all pods without NAT

```
Node 1 (10.0.0.1)          Node 2 (10.0.0.2)
├── Pod A (10.244.1.1)      ├── Pod C (10.244.2.1)
└── Pod B (10.244.1.2)      └── Pod D (10.244.2.2)

Pod A can reach Pod D at 10.244.2.2 directly
```

### Pod Networking

Each pod has:
- One network namespace
- One virtual ethernet interface (veth)
- Connected to the node's bridge network

```
Pod                    Node
┌──────────────┐      ┌─────────────────────┐
│  eth0        │      │  veth0  ←──── bridge │
│ 10.244.1.1   ├─────►│                      │
└──────────────┘      │  cbr0 (10.244.1.0/24)│
                      └─────────────────────┘
```

### Service Networking

Services get a virtual IP (ClusterIP) from a separate range. `kube-proxy` on each node creates iptables rules that DNAT traffic to one of the backend pod IPs.

```
Client Pod → iptables DNAT → Service ClusterIP → Pod IP
```

### DNS in Kubernetes

CoreDNS provides DNS resolution. Pods automatically get `/etc/resolv.conf` pointing to CoreDNS.

```
# Pod's /etc/resolv.conf
nameserver 10.96.0.10       # CoreDNS cluster IP
search default.svc.cluster.local svc.cluster.local cluster.local

# DNS resolution examples:
# Within same namespace:
my-service → my-service.default.svc.cluster.local

# Cross-namespace:
my-service.other-ns → my-service.other-ns.svc.cluster.local
```

### CNI Plugins

Container Network Interface (CNI) plugins implement the actual pod networking:

| Plugin | Features | Common Use |
|---|---|---|
| **Flannel** | Simple, VXLAN overlay | Learning, simple setups |
| **Calico** | BGP routing + NetworkPolicy | Production, fine-grained security |
| **Weave** | Mesh networking | Multi-cloud |
| **Cilium** | eBPF-based, L7 policies | Advanced performance/security |

### Network Policies

NetworkPolicies control traffic between pods (acts as a firewall):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend    # Only allow traffic from frontend pods
    ports:
    - protocol: TCP
      port: 8080
```

---

## 12. Auto Scaling

### Horizontal Pod Autoscaler (HPA)

HPA automatically scales the number of pod replicas based on CPU/memory usage or custom metrics.

```
Low Traffic:           High Traffic:
[Pod] [Pod]     →      [Pod] [Pod] [Pod] [Pod] [Pod]
```

**Installing Metrics Server (required for HPA):**
```bash
# Minikube
minikube addons enable metrics-server

# Other clusters
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

**HPA YAML:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # Scale up when CPU > 70%
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**HPA Commands:**
```bash
# Create HPA imperatively
kubectl autoscale deployment nginx-deployment --cpu-percent=70 --min=2 --max=10

# View HPA
kubectl get hpa

# Test autoscaling (create load)
kubectl run load-test --image=busybox --rm -it -- \
  sh -c "while true; do wget -q -O- http://nginx-service; done"

# Watch HPA react
kubectl get hpa -w
```

### Vertical Pod Autoscaler (VPA)

VPA automatically adjusts the CPU/memory requests and limits for containers:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  updatePolicy:
    updateMode: "Auto"
```

### Cluster Autoscaler

Cluster Autoscaler adds or removes worker nodes based on pod scheduling needs. It works with cloud providers:

```
Pending Pods (can't fit on existing nodes)
    │
    ▼
Cluster Autoscaler detects pending pods
    │
    ▼
Provisions new node from cloud provider
    │
    ▼
Pods scheduled on new node

(Reverse: idle nodes removed after scale-down delay)
```

```yaml
# Example for AWS EKS
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: cluster-autoscaler
        image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.28.0
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws
        - --nodes=1:10:my-node-group
        - --scale-down-delay-after-add=10m
```

---

## 13. Monitoring and Logging

### The Monitoring Stack

```
┌──────────────────────────────────────────────────┐
│                  Grafana (Visualization)          │
│                  http://grafana:3000              │
└─────────────────────┬────────────────────────────┘
                      │ queries
                      ▼
┌──────────────────────────────────────────────────┐
│               Prometheus (Metrics Store)          │
│               scrapes /metrics endpoints          │
└──────┬───────────────┬───────────────────────────┘
       │               │
       ▼               ▼
  Node Exporter   kube-state-metrics   App /metrics
  (node metrics)  (K8s object metrics) (custom)
```

### Prometheus

Prometheus is a time-series database that scrapes metrics from HTTP endpoints.

**Install via Helm:**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

**Prometheus annotations on a pod:**
```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

### Grafana

Grafana provides dashboards and visualization. Access after install:

```bash
# Port-forward to access Grafana
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring

# Default credentials: admin/prom-operator
```

Popular dashboards to import:
- **ID 6417** — Kubernetes cluster monitoring
- **ID 315** — Kubernetes overview
- **ID 1860** — Node Exporter full

### Logging with ELK / EFK Stack

```
Pods → Fluentd/Fluentbit (DaemonSet on every node)
           │
           ▼
      Elasticsearch (storage)
           │
           ▼
        Kibana (visualization)
```

**Install with Helm:**
```bash
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch -n logging --create-namespace
helm install kibana elastic/kibana -n logging
helm install filebeat elastic/filebeat -n logging
```

### Structured Logging Best Practices

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "service": "payment-api",
  "trace_id": "abc123",
  "message": "Payment processed",
  "amount": 99.99,
  "user_id": "user_456"
}
```

### Alerting with AlertManager

```yaml
# Example PrometheusRule
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: pod-crash-alert
spec:
  groups:
  - name: pod-alerts
    rules:
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod {{ $labels.pod }} is crash looping"
```

---

## 14. CI/CD with Kubernetes

### Full Pipeline Architecture

```
Developer writes code
        │
        ▼
  Git Push/PR
        │
        ▼
  CI Pipeline (GitHub Actions / GitLab CI / Jenkins)
        │
        ├── Run tests
        ├── Static analysis
        ├── Security scan
        │
        ▼
  Docker Build
        │
        ▼
  Push to Registry
  (Docker Hub / ECR / GCR / GHCR)
        │
        ▼
  Update Kubernetes Manifests
  (update image tag in deployment.yaml)
        │
        ▼
  kubectl apply / ArgoCD sync
        │
        ▼
  Kubernetes Rolling Update
        │
        ▼
  Health checks pass → Deployment complete
  Health checks fail → Automatic rollback
```

### GitHub Actions Example

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to DockerHub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        push: true
        tags: myorg/myapp:${{ github.sha }}

    - name: Set up kubectl
      uses: azure/setup-kubectl@v3

    - name: Deploy to Kubernetes
      env:
        KUBECONFIG: ${{ secrets.KUBECONFIG }}
      run: |
        kubectl set image deployment/myapp \
          myapp=myorg/myapp:${{ github.sha }}
        kubectl rollout status deployment/myapp
```

### GitOps with ArgoCD

ArgoCD is a GitOps continuous delivery tool for Kubernetes:

```
Git Repository (source of truth)
        │
        │ ArgoCD watches for changes
        ▼
  ArgoCD Controller
        │
        │ syncs
        ▼
  Kubernetes Cluster
```

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**ArgoCD Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-manifests
    targetRevision: HEAD
    path: k8s/
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## 15. Kubernetes Security

### RBAC — Role-Based Access Control

RBAC controls what users and service accounts can do in Kubernetes.

**Key objects:**
- **Role** — permissions within a namespace
- **ClusterRole** — permissions cluster-wide
- **RoleBinding** — bind a Role to a user/group/serviceaccount
- **ClusterRoleBinding** — bind ClusterRole cluster-wide

```yaml
# Role: read pods in "staging" namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: staging
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "watch", "list"]

---
# Bind role to a user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: staging
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Service Accounts

A ServiceAccount is an identity for pods that allows them to call the Kubernetes API:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: production

---
# Bind a role to the service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-sa-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# Use it in a pod
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: my-app-sa
  containers:
  - name: app
    image: my-app:1.0
```

### Pod Security

**Security Contexts:**
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000

  containers:
  - name: app
    image: my-app:1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

### Network Policies for Security

Deny all ingress by default, then allow only what's needed:

```yaml
# Default deny all ingress in a namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}       # Applies to all pods
  policyTypes:
  - Ingress

---
# Allow frontend → backend communication only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Image Security

```bash
# Scan images for vulnerabilities
trivy image nginx:latest

# Use minimal base images
FROM alpine:3.18    # Instead of ubuntu

# Don't run as root
USER nonroot

# Sign images with cosign
cosign sign myorg/myapp:v1.0
```

---

## 16. Production Best Practices

### Resource Requests and Limits

Always set resource requests and limits to prevent resource starvation:

```yaml
resources:
  requests:
    memory: "128Mi"     # Guaranteed minimum
    cpu: "100m"         # 0.1 CPU core
  limits:
    memory: "256Mi"     # Hard maximum (OOMKilled if exceeded)
    cpu: "500m"         # Throttled if exceeded
```

**CPU units:** `1000m = 1 CPU core`. `100m = 0.1 core`
**Memory units:** `Ki, Mi, Gi`

### Liveness and Readiness Probes

```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    
    livenessProbe:         # Is the container alive? Restart if not.
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      failureThreshold: 3

    readinessProbe:        # Is the container ready for traffic? Remove from LB if not.
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3

    startupProbe:          # For slow-starting containers
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30  # 30 * 10s = 5 minutes max startup time
      periodSeconds: 10
```

### Pod Disruption Budgets (PDB)

Ensure a minimum number of pods remain available during voluntary disruptions (node drains):

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 2       # Or use maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
```

### High Availability

```yaml
# Spread pods across nodes with topology spread
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: nginx
```

### Graceful Shutdown

```yaml
spec:
  terminationGracePeriodSeconds: 60    # Default is 30s
  containers:
  - name: app
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 5"]    # Allow in-flight requests to complete
```

### Namespace Resource Quotas

Limit resource consumption per namespace:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
    pods: "20"
    services: "10"
```

### LimitRange

Set default and max resource limits per namespace:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
```

---

## 17. Troubleshooting Kubernetes

### Debugging Workflow

```
Pod not working?
     │
     ▼
kubectl get pods
     │
     ├── Pending? ──► kubectl describe pod → check Events section
     │                 Common: insufficient resources, no matching node
     │
     ├── CrashLoopBackOff? ──► kubectl logs --previous
     │                          Common: app crash, bad config, missing env vars
     │
     ├── ImagePullBackOff? ──► kubectl describe pod → check image name/registry
     │                          Common: wrong image name, missing registry credentials
     │
     ├── Running but not accessible? ──► Check Service selector labels
     │                                   kubectl get endpoints my-service
     │
     └── OOMKilled? ──► Increase memory limits
```

### Essential Debugging Commands

```bash
# Check pod status
kubectl get pods -o wide

# Detailed pod information with events
kubectl describe pod POD_NAME

# Container logs
kubectl logs POD_NAME
kubectl logs POD_NAME --previous    # Logs from crashed container
kubectl logs -f POD_NAME            # Follow logs

# Execute commands inside container
kubectl exec -it POD_NAME -- bash
kubectl exec -it POD_NAME -- sh     # If bash not available

# Check cluster events (sorted by time)
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -n kube-system

# Resource usage
kubectl top pods
kubectl top nodes

# Describe a service
kubectl describe svc SERVICE_NAME

# Check endpoints (are pods registered with service?)
kubectl get endpoints SERVICE_NAME

# Check if DNS works (from inside pod)
kubectl exec -it POD_NAME -- nslookup kubernetes.default
kubectl exec -it POD_NAME -- curl http://other-service
```

### Common Issues and Solutions

#### CrashLoopBackOff

```bash
kubectl logs POD_NAME --previous
# Read the crash output — it's usually:
# - Missing environment variable
# - Failed database connection
# - Permission error
# - Application bug
```

#### ImagePullBackOff / ErrImagePull

```bash
kubectl describe pod POD_NAME
# Look for: "Failed to pull image"
# Solutions:
# - Fix image name/tag typo
# - Create docker-registry secret for private registries

kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

# Reference in pod:
spec:
  imagePullSecrets:
  - name: regcred
```

#### Pending Pod

```bash
kubectl describe pod POD_NAME
# Look for: "Insufficient cpu/memory" or "No matching node"
# Solutions:
# - Scale your cluster (add nodes)
# - Reduce resource requests
# - Remove taints or fix tolerations
```

#### Service Not Routing Traffic

```bash
# Check that pod labels match service selector
kubectl get pod --show-labels
kubectl describe svc SERVICE_NAME | grep Selector

# Check endpoints
kubectl get endpoints SERVICE_NAME
# If ENDPOINTS shows "<none>", labels don't match

# Test connectivity from inside cluster
kubectl run test --image=busybox --rm -it -- wget -qO- http://SERVICE_NAME
```

#### OOMKilled

```bash
kubectl describe pod POD_NAME | grep -A5 "Last State"
# OOMKilled = out of memory
# Solution: increase memory limit or optimize app
```

### Debug with Ephemeral Containers

```bash
# Attach a debug container to a running pod (K8s 1.23+)
kubectl debug -it POD_NAME --image=busybox --target=CONTAINER_NAME

# Create a copy of a pod with debug tools
kubectl debug POD_NAME -it --copy-to=debug-pod --image=ubuntu
```

---

## 18. Real World Project — MERN Stack

### Architecture Overview

Deploy a full MERN (MongoDB, Express, React, Node) application to Kubernetes.

```
Internet
    │
    ▼
┌──────────────────────────────────────────┐
│           Nginx Ingress Controller        │
└──────────────────────────────────────────┘
    │                         │
    ▼                         ▼
/             →       api.example.com →
Frontend Service               Backend Service
    │                              │
    ▼                              ▼
React Pods (x3)            Node.js API Pods (x3)
                                   │
                                   ▼
                           MongoDB StatefulSet
                           (with PersistentVolume)
```

### Project File Structure

```
mern-k8s/
├── frontend/
│   ├── Dockerfile
│   └── nginx.conf
├── backend/
│   └── Dockerfile
└── k8s/
    ├── namespace.yaml
    ├── frontend/
    │   ├── deployment.yaml
    │   └── service.yaml
    ├── backend/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── configmap.yaml
    ├── mongodb/
    │   ├── statefulset.yaml
    │   ├── service.yaml
    │   └── secret.yaml
    └── ingress.yaml
```

### Step 1: Namespace

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mern-app
  labels:
    env: production
```

### Step 2: MongoDB Secret and StatefulSet

```yaml
# k8s/mongodb/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: mern-app
type: Opaque
data:
  MONGO_ROOT_PASSWORD: bXlzZWNyZXRwYXNz    # base64("mysecretpass")
  MONGO_USERNAME: YWRtaW4=                  # base64("admin")

---
# k8s/mongodb/statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
  namespace: mern-app
spec:
  serviceName: mongodb
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:6.0
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: MONGO_USERNAME
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: MONGO_ROOT_PASSWORD
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        volumeMounts:
        - name: mongo-data
          mountPath: /data/db
        livenessProbe:
          exec:
            command:
            - mongosh
            - --eval
            - "db.adminCommand('ping')"
          initialDelaySeconds: 30
          periodSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: mongo-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi

---
# k8s/mongodb/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  namespace: mern-app
spec:
  clusterIP: None      # Headless service for StatefulSet
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
```

### Step 3: Backend (Node.js API)

```yaml
# k8s/backend/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
  namespace: mern-app
data:
  NODE_ENV: "production"
  PORT: "5000"
  MONGO_URI: "mongodb://admin:$(MONGO_PASS)@mongodb:27017/merndb?authSource=admin"

---
# k8s/backend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: mern-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: myorg/mern-backend:latest
        ports:
        - containerPort: 5000
        envFrom:
        - configMapRef:
            name: backend-config
        env:
        - name: MONGO_PASS
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: MONGO_ROOT_PASSWORD
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "300m"
        readinessProbe:
          httpGet:
            path: /api/health
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /api/health
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10

---
# k8s/backend/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: mern-app
spec:
  selector:
    app: backend
  ports:
  - port: 5000
    targetPort: 5000
```

### Step 4: Frontend (React)

```yaml
# k8s/frontend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: mern-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: myorg/mern-frontend:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
# k8s/frontend/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: mern-app
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

### Step 5: Ingress

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mern-ingress
  namespace: mern-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 5000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

### Deploying the Project

```bash
# Apply all manifests in order
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/mongodb/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/
kubectl apply -f k8s/ingress.yaml

# Check everything is running
kubectl get all -n mern-app

# Follow MongoDB startup
kubectl logs -f mongodb-0 -n mern-app

# Test backend health
kubectl port-forward svc/backend-service 5000:5000 -n mern-app
curl http://localhost:5000/api/health
```

---

## 19. Advanced Kubernetes

### Helm — Kubernetes Package Manager

Helm is the package manager for Kubernetes. It templatizes YAML files into reusable "charts".

```
Helm Chart
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes YAML templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
└── charts/             # Dependencies
```

**Basic Helm commands:**
```bash
# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx

# Install a chart
helm install my-nginx bitnami/nginx

# Install with custom values
helm install my-postgres bitnami/postgresql \
  --set auth.postgresPassword=secret \
  --set primary.persistence.size=20Gi

# List installed releases
helm list

# Upgrade a release
helm upgrade my-nginx bitnami/nginx --set replicaCount=3

# Rollback
helm rollback my-nginx 1

# Uninstall
helm uninstall my-nginx

# Create your own chart
helm create my-app
```

**Example template with values:**
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

```yaml
# values.yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

### StatefulSets

StatefulSets are for **stateful applications** (databases, message queues) that need:
- Stable, persistent network identities
- Ordered, graceful deployment and scaling
- Stable persistent storage

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7.0
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

Pods get stable names: `redis-0`, `redis-1`, `redis-2`
DNS: `redis-0.redis.default.svc.cluster.local`

### DaemonSets

A DaemonSet ensures that **one pod runs on every node** (or a subset). Perfect for:
- Log collectors (Fluentd, Filebeat)
- Monitoring agents (Prometheus Node Exporter)
- Network plugins (CNI)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          hostPort: 9100
        volumeMounts:
        - mountPath: /host/proc
          name: proc
          readOnly: true
        - mountPath: /host/sys
          name: sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

### Jobs and CronJobs

**Job** — run a task to completion:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: migration
        image: my-app:latest
        command: ["python", "manage.py", "migrate"]
```

**CronJob** — run on a schedule:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"    # Daily at 2am (cron syntax)
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: my-backup-tool:latest
            command: ["./backup.sh"]
```

### Custom Resource Definitions (CRD)

CRDs extend the Kubernetes API with your own resource types:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.myorg.io
spec:
  group: myorg.io
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: [postgres, mysql, mongodb]
              storage:
                type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

Now you can create objects of kind `Database`:
```yaml
apiVersion: myorg.io/v1
kind: Database
metadata:
  name: my-postgres
spec:
  engine: postgres
  storage: 20Gi
```

### Operators

An **Operator** is a controller that manages a custom resource. It codifies operational knowledge:

```
Human Operator                    Kubernetes Operator
"Deploy Postgres with             CRD: kind: PostgresCluster
 replication, backups..."    →    Controller watches CRDs
                                  and manages Postgres pods,
                                  services, backups automatically
```

Popular operators:
- **cert-manager** — manages TLS certificates
- **Prometheus Operator** — manages Prometheus deployments
- **Strimzi** — manages Kafka clusters
- **CloudNativePG** — manages PostgreSQL clusters

### Taints and Tolerations

Taints prevent pods from being scheduled on certain nodes:

```bash
# Taint a node (GPU nodes only for GPU workloads)
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule
```

Tolerations allow pods to be scheduled on tainted nodes:

```yaml
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  nodeSelector:
    kubernetes.io/hostname: gpu-node-1
```

---

## 20. Kubernetes Learning Roadmap

### Stage 1: Beginner (Weeks 1-4)

**Goal:** Understand core concepts and deploy simple apps.

**Topics:**
- Docker fundamentals (prerequisite)
- Kubernetes architecture
- Pods, Deployments, Services
- kubectl CLI basics
- Namespaces, Labels, Annotations
- ConfigMaps and Secrets
- Basic troubleshooting

**Hands-on:**
- Set up Minikube locally
- Deploy nginx with a Deployment
- Expose it with a Service
- Scale it up and down
- Perform a rolling update and rollback

**Resources:**
- Kubernetes official docs: kubernetes.io/docs
- Play with Kubernetes: labs.play-with-k8s.com
- Katacoda Kubernetes scenarios

---

### Stage 2: Intermediate (Weeks 5-10)

**Goal:** Understand networking, storage, and production patterns.

**Topics:**
- Ingress and Ingress Controllers
- PersistentVolumes and PVCs
- StatefulSets
- Resource quotas, limits, and LimitRanges
- Liveness and Readiness probes
- Horizontal Pod Autoscaler
- Basic RBAC
- NetworkPolicies
- Multi-container pod patterns

**Hands-on:**
- Deploy a stateful database (PostgreSQL/MongoDB)
- Set up Nginx Ingress with path-based routing
- Configure HPA for a deployment
- Deploy a multi-tier application (frontend + backend + DB)

---

### Stage 3: Advanced (Weeks 11-18)

**Goal:** Master production-grade Kubernetes.

**Topics:**
- Helm charts (using and creating)
- Helm templating
- Prometheus and Grafana monitoring stack
- EFK/ELK logging stack
- ArgoCD / GitOps
- Kubernetes security deep dive (RBAC, PodSecurity, OPA/Gatekeeper)
- CI/CD pipelines
- Cluster Autoscaler
- Custom Operators and CRDs
- Kubernetes API extension
- Multi-cluster deployments
- Service Mesh (Istio or Linkerd)

**Hands-on:**
- Build a Helm chart for your own application
- Set up full Prometheus + Grafana + AlertManager stack
- Implement GitOps with ArgoCD
- Deploy a service mesh with Istio

---

### Stage 4: Production DevOps Engineer (Weeks 19+)

**Goal:** Run Kubernetes in production reliably and securely.

**Topics:**
- Kubernetes cluster provisioning (kubeadm, EKS, GKE, AKS)
- Multi-node high availability setup
- Disaster recovery (etcd backup/restore)
- Kubernetes upgrade procedures
- Cost optimization (Goldilocks, VPA)
- Multi-tenancy patterns
- Compliance (CIS benchmarks, kube-bench)
- Advanced networking (BGP, multi-cluster)
- Chaos engineering (Chaos Monkey, LitmusChaos)

**Certifications to pursue:**
| Certification | Level | Focus |
|---|---|---|
| CKA (Certified Kubernetes Administrator) | Intermediate | Cluster administration |
| CKAD (Certified Kubernetes App Developer) | Intermediate | App deployment |
| CKS (Certified Kubernetes Security Specialist) | Advanced | Security hardening |

---

### The Full Roadmap

```
BEGINNER
   │
   ├── Docker basics
   ├── Pods, Deployments, Services
   ├── kubectl CLI
   └── Basic YAML
   │
   ▼
INTERMEDIATE
   │
   ├── Ingress, Storage, Networking
   ├── RBAC & Security basics
   ├── Autoscaling
   └── Multi-tier apps
   │
   ▼
ADVANCED
   │
   ├── Helm, Operators, CRDs
   ├── CI/CD, GitOps
   ├── Monitoring & Logging
   └── Service Mesh
   │
   ▼
PRODUCTION DEVOPS ENGINEER
   │
   ├── Cluster provisioning
   ├── HA, DR, backups
   ├── Cost optimization
   ├── Security compliance
   └── CKA/CKAD/CKS certification
```

---

## Quick Reference — Production Checklist

Before deploying to production, verify:

- [ ] Resource requests and limits set on all containers
- [ ] Liveness and readiness probes configured
- [ ] Pod Disruption Budgets created for critical workloads
- [ ] Replicas > 1 for all stateless services
- [ ] Anti-affinity rules to spread pods across nodes
- [ ] Secrets stored securely (not in Git)
- [ ] RBAC configured with least privilege
- [ ] Network policies in place
- [ ] Container running as non-root
- [ ] Images scanned for vulnerabilities
- [ ] Resource quotas on namespaces
- [ ] HPA configured for variable load services
- [ ] etcd backups configured
- [ ] Monitoring and alerting set up
- [ ] Log aggregation configured
- [ ] Graceful shutdown handling implemented

---

## Final Summary

Kubernetes is the foundation of modern cloud-native infrastructure. This guide has taken you from the basics of what Kubernetes is, through every core concept, all the way to production-grade patterns used by top engineering teams worldwide.

The best way to learn Kubernetes is by doing. Spin up Minikube today, deploy your first pod, break things, and fix them. The journey from beginner to production Kubernetes engineer is challenging but immensely rewarding.

**Keep building. Keep breaking. Keep learning.**

---

*Generated as part of the Kubernetes Mastery Roadmap series.*
*For more resources, visit: kubernetes.io/docs*
