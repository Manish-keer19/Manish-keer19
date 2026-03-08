# Kubernetes & kubectl Complete Guide

> A comprehensive reference for developers — from architecture fundamentals to production workflows.

---

## Table of Contents

1. [Kubernetes Architecture](#1-kubernetes-architecture)
2. [Deployment → ReplicaSet → Pod → Container](#2-deployment--replicaset--pod--container)
3. [Networking Architecture](#3-networking-architecture)
4. [kubectl Installation & Configuration](#4-kubectl-installation--configuration)
5. [Core kubectl Commands](#5-core-kubectl-commands)
6. [Pod Commands](#6-pod-commands)
7. [Service Commands](#7-service-commands)
8. [Deployment Commands](#8-deployment-commands)
9. [Scaling & Rollout Commands](#9-scaling--rollout-commands)
10. [Namespace Management](#10-namespace-management)
11. [Port Forwarding](#11-port-forwarding)
12. [Debugging & Troubleshooting](#12-debugging--troubleshooting)
13. [ConfigMaps & Secrets](#13-configmaps--secrets)
14. [Cluster Cleanup Commands](#14-cluster-cleanup-commands)
15. [MERN Stack on Kubernetes](#15-mern-stack-on-kubernetes)
16. [Developer Workflow: Docker → Kubernetes](#16-developer-workflow-docker--kubernetes)

---

## 1. Kubernetes Architecture

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          KUBERNETES CLUSTER                             │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        CONTROL PLANE                            │   │
│  │                                                                 │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │   │
│  │  │  API Server  │  │  Scheduler   │  │  Controller Manager  │  │   │
│  │  │  (kube-      │  │  (kube-      │  │  (kube-controller-   │  │   │
│  │  │  apiserver)  │  │  scheduler)  │  │   manager)           │  │   │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘  │   │
│  │           │                                                     │   │
│  │  ┌────────┴────────────────────────────────────────────────┐   │   │
│  │  │                      etcd                               │   │   │
│  │  │           (Distributed Key-Value Store)                 │   │   │
│  │  └────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌──────────────────────┐    ┌──────────────────────┐                  │
│  │      WORKER NODE 1   │    │      WORKER NODE 2   │                  │
│  │                      │    │                      │                  │
│  │  ┌────────────────┐  │    │  ┌────────────────┐  │                  │
│  │  │    kubelet     │  │    │  │    kubelet     │  │                  │
│  │  └────────────────┘  │    │  └────────────────┘  │                  │
│  │  ┌────────────────┐  │    │  ┌────────────────┐  │                  │
│  │  │   kube-proxy   │  │    │  │   kube-proxy   │  │                  │
│  │  └────────────────┘  │    │  └────────────────┘  │                  │
│  │  ┌────────────────┐  │    │  ┌────────────────┐  │                  │
│  │  │ Container      │  │    │  │ Container      │  │                  │
│  │  │ Runtime        │  │    │  │ Runtime        │  │                  │
│  │  │ (containerd)   │  │    │  │ (containerd)   │  │                  │
│  │  └────────────────┘  │    │  └────────────────┘  │                  │
│  │  [Pod][Pod][Pod]     │    │  [Pod][Pod][Pod]     │                  │
│  └──────────────────────┘    └──────────────────────┘                  │
└─────────────────────────────────────────────────────────────────────────┘
```

### Control Plane Components

| Component | Role |
|-----------|------|
| **API Server** | The front door to the cluster. All `kubectl` commands hit this REST API. Validates and stores objects in etcd. |
| **etcd** | Distributed key-value store. Single source of truth for all cluster state. |
| **Scheduler** | Watches for unscheduled pods and assigns them to worker nodes based on resource availability, taints, tolerations, and affinity rules. |
| **Controller Manager** | Runs control loops (controllers) that reconcile actual state with desired state (e.g., ReplicaSet controller, Deployment controller). |
| **Cloud Controller Manager** | Integrates with cloud provider APIs (AWS, GCP, Azure) for load balancers, storage, etc. |

### Worker Node Components

| Component | Role |
|-----------|------|
| **kubelet** | Agent on every node. Receives PodSpecs from the API server and ensures the described containers are running and healthy. |
| **kube-proxy** | Network proxy on every node. Maintains network rules that allow communication to Pods from inside or outside the cluster. |
| **Container Runtime** | Software that runs containers (containerd, CRI-O, Docker). Kubelet communicates with it via CRI (Container Runtime Interface). |

---

## 2. Deployment → ReplicaSet → Pod → Container

This is the ownership hierarchy for running workloads in Kubernetes.

```
┌──────────────────────────────────────────────────────────────────┐
│                         DEPLOYMENT                               │
│  name: my-app-deployment                                         │
│  replicas: 3                                                     │
│  strategy: RollingUpdate                                         │
│  template: <pod spec>                                            │
│                                                                  │
│  - Manages rollouts and rollbacks                                │
│  - Declares desired state (e.g., "always run 3 replicas")        │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    REPLICASET (v2)                         │  │
│  │  name: my-app-deployment-7d9f8b6c4                        │  │
│  │  replicas: 3 (desired) / 3 (ready)                        │  │
│  │                                                            │  │
│  │  - Created by Deployment automatically                     │  │
│  │  - Ensures N identical pods are always running             │  │
│  │  - Old ReplicaSets kept for rollback history               │  │
│  │                                                            │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │                      POD                            │  │  │
│  │  │  name: my-app-deployment-7d9f8b6c4-xk2vp            │  │  │
│  │  │  IP: 10.244.1.5                                     │  │  │
│  │  │  node: worker-node-1                                │  │  │
│  │  │                                                     │  │  │
│  │  │  - Smallest deployable unit                         │  │  │
│  │  │  - Shares network namespace (localhost)             │  │  │
│  │  │  - Shares storage volumes                           │  │  │
│  │  │  - Ephemeral: dies and is replaced                  │  │  │
│  │  │                                                     │  │  │
│  │  │  ┌──────────────┐  ┌──────────────────────────┐    │  │  │
│  │  │  │  CONTAINER   │  │   SIDECAR CONTAINER      │    │  │  │
│  │  │  │  my-app      │  │   log-aggregator         │    │  │  │
│  │  │  │  image:      │  │   image: fluent/fluentd  │    │  │  │
│  │  │  │  my-app:v2   │  │   port: 24224            │    │  │  │
│  │  │  │  port: 3000  │  │                          │    │  │  │
│  │  │  └──────────────┘  └──────────────────────────┘    │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  │                    POD   POD   (3 total)                    │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### Example: Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
  labels:
    app: my-app
spec:
  replicas: 3                          # ReplicaSet will maintain 3 Pods
  selector:
    matchLabels:
      app: my-app                      # Which Pods this Deployment manages
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                      # 1 extra pod allowed during update
      maxUnavailable: 0                # 0 pods down during update
  template:                            # Pod template (used by ReplicaSet)
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:v2
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
```

---

## 3. Networking Architecture

### Pod-to-Pod Communication

```
┌──────────────────────────────────────────────────────────────────────┐
│                        CLUSTER NETWORK                               │
│                                                                      │
│  Pod CIDR: 10.244.0.0/16   Service CIDR: 10.96.0.0/12               │
│                                                                      │
│  ┌───────────────────────┐         ┌───────────────────────┐        │
│  │     Worker Node 1     │         │     Worker Node 2     │        │
│  │   Subnet: 10.244.1/24 │         │   Subnet: 10.244.2/24 │        │
│  │                       │         │                       │        │
│  │  Pod A: 10.244.1.2    │◄───────►│  Pod B: 10.244.2.3    │        │
│  │  Pod C: 10.244.1.5    │         │  Pod D: 10.244.2.8    │        │
│  └───────────────────────┘         └───────────────────────┘        │
│             │                                   │                    │
│    ┌────────▼───────────────────────────────────▼───────────────┐   │
│    │                  CNI Plugin (Flannel / Calico / Weave)      │   │
│    │        Manages Pod IP assignment and cross-node routing     │   │
│    └────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
```

### Service Types

```
External Traffic
      │
      ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LoadBalancer Service (cloud-provided external IP)                  │
│  ClusterIP: 10.96.45.100 / External: 203.0.113.5:80                │
│                       │                                             │
│             ┌─────────▼──────────┐                                  │
│             │   NodePort Service  │  (also accessible at NodeIP:30080) │
│             │   ClusterIP: virtual│                                  │
│             └─────────┬──────────┘                                  │
│                       │  kube-proxy routes to pod IPs               │
│          ┌────────────┼────────────┐                                │
│          ▼            ▼            ▼                                │
│       Pod A        Pod B        Pod C                               │
│    10.244.1.2   10.244.1.3   10.244.2.4                             │
└─────────────────────────────────────────────────────────────────────┘

Service Types:
  ClusterIP    → Internal only (default). Stable virtual IP inside cluster.
  NodePort     → Exposes on each node's IP at a static port (30000-32767).
  LoadBalancer → Provisions cloud load balancer with external IP.
  ExternalName → Maps service to a DNS name (e.g., an external database).
```

### Ingress Architecture

```
Internet
    │
    ▼
┌───────────────────────────────────────────────────────┐
│                 Ingress Controller                     │
│              (nginx / traefik / istio)                │
│                                                       │
│  Rules:                                               │
│    api.myapp.com  → api-service:80                    │
│    myapp.com/     → frontend-service:80               │
│    myapp.com/ws   → websocket-service:8080            │
└───────────────────────────────────────────────────────┘
         │                    │                   │
         ▼                    ▼                   ▼
   [api-service]       [frontend-svc]      [ws-service]
         │                    │                   │
      [Pods]               [Pods]              [Pods]
```

---

## 4. kubectl Installation & Configuration

### Installation

```bash
# macOS (Homebrew)
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Windows (Chocolatey)
choco install kubernetes-cli

# Verify installation
kubectl version --client
```

### kubeconfig & Contexts

The kubeconfig file (`~/.kube/config`) stores cluster credentials and contexts.

```bash
# View entire kubeconfig
kubectl config view

# List all available contexts
kubectl config get-contexts

# Show current context
kubectl config current-context

# Switch to a different context
kubectl config use-context my-cluster-prod

# Set a context with a specific namespace
kubectl config set-context --current --namespace=my-namespace

# Add a new cluster to kubeconfig
kubectl config set-cluster my-cluster --server=https://1.2.3.4:6443

# Merge multiple kubeconfig files
KUBECONFIG=~/.kube/config:~/.kube/config2 kubectl config view --flatten > ~/.kube/merged
```

### Useful Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kns='kubectl config set-context --current --namespace'
alias kctx='kubectl config use-context'

# Auto-completion (bash)
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

# Auto-completion (zsh)
source <(kubectl completion zsh)
echo "[[ \$commands[kubectl] ]] && source <(kubectl completion zsh)" >> ~/.zshrc
```

---

## 5. Core kubectl Commands

### Resource Overview

```bash
# Get cluster info
kubectl cluster-info

# View all nodes
kubectl get nodes
kubectl get nodes -o wide           # Show IPs, OS, container runtime

# Describe a node
kubectl describe node worker-node-1

# Get all resources across all namespaces
kubectl get all --all-namespaces
kubectl get all -A                  # Shorthand

# Get all resources in current namespace
kubectl get all

# Watch resources in real-time
kubectl get pods -w
kubectl get pods --watch

# Output formats
kubectl get pods -o wide            # More columns
kubectl get pods -o yaml            # Full YAML spec
kubectl get pods -o json            # JSON output
kubectl get pods -o name            # Just resource names
kubectl get pod my-pod -o jsonpath='{.status.podIP}'

# Sort output
kubectl get pods --sort-by='.metadata.name'
kubectl get pods --sort-by='.status.startTime'

# Filter with labels
kubectl get pods -l app=my-app
kubectl get pods -l app=my-app,env=production
kubectl get pods -l 'app in (frontend,backend)'
kubectl get pods -l 'env notin (dev,staging)'

# API resource types
kubectl api-resources                          # All resource types
kubectl api-resources --namespaced=true        # Only namespaced resources
kubectl explain pod                            # Describe a resource type
kubectl explain pod.spec.containers            # Nested field explanation
```

### Apply, Create, Delete

```bash
# Apply a manifest (create or update)
kubectl apply -f deployment.yaml
kubectl apply -f ./k8s/                         # All files in directory
kubectl apply -f ./k8s/ --recursive             # Recurse into subdirs
kubectl apply -k ./kustomize/                   # Kustomize overlay

# Create resources (fails if exists)
kubectl create -f deployment.yaml

# Create from generators (no YAML needed)
kubectl create deployment nginx --image=nginx --replicas=3
kubectl create service clusterip my-svc --tcp=80:8080
kubectl create configmap my-config --from-literal=key=value
kubectl create secret generic my-secret --from-literal=password=s3cr3t

# Delete resources
kubectl delete -f deployment.yaml
kubectl delete pod my-pod
kubectl delete pod my-pod --grace-period=0 --force    # Force immediate delete
kubectl delete pods -l app=my-app                     # Delete by label
kubectl delete all -l app=my-app                      # Delete all labeled resources

# Dry run (preview without applying)
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --dry-run=server     # Server-side validation

# Generate YAML without applying
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deploy.yaml
```

---

## 6. Pod Commands

### Viewing Pods

```bash
# List pods
kubectl get pods
kubectl get pods -n kube-system                  # Specific namespace
kubectl get pods -A                              # All namespaces
kubectl get pods -o wide                         # With node & IP info

# Describe a pod (events, conditions, volumes)
kubectl describe pod my-pod
kubectl describe pod my-pod -n my-namespace

# Get pod YAML
kubectl get pod my-pod -o yaml

# Get pod logs
kubectl logs my-pod                              # Current logs
kubectl logs my-pod -f                           # Follow (stream) logs
kubectl logs my-pod --previous                   # Logs from crashed container
kubectl logs my-pod --tail=100                   # Last 100 lines
kubectl logs my-pod --since=1h                   # Last 1 hour
kubectl logs my-pod --since-time="2025-01-01T00:00:00Z"

# Logs from multi-container pods
kubectl logs my-pod -c my-container              # Specific container
kubectl logs my-pod --all-containers=true        # All containers

# Logs from all pods with a label
kubectl logs -l app=my-app --all-containers=true
```

### Exec & Shell Access

```bash
# Execute a command in a pod
kubectl exec my-pod -- ls /app
kubectl exec my-pod -- env
kubectl exec my-pod -- cat /etc/config/app.yaml

# Interactive shell
kubectl exec -it my-pod -- /bin/bash
kubectl exec -it my-pod -- /bin/sh                   # If bash not available
kubectl exec -it my-pod -c my-container -- /bin/bash # Specific container

# Run a command and get output
kubectl exec my-pod -- curl -s http://localhost:3000/health
```

### Running Temporary Pods

```bash
# Run a one-off pod (deleted after exit)
kubectl run tmp-shell --rm -it --image=busybox -- sh
kubectl run tmp-curl --rm -it --image=curlimages/curl -- sh
kubectl run tmp-debug --rm -it --image=ubuntu -- bash

# Test service connectivity from inside cluster
kubectl run tmp-curl --rm -it --image=curlimages/curl -- \
  curl http://my-service.my-namespace.svc.cluster.local

# Run a pod and keep it running
kubectl run my-pod --image=nginx --restart=Never
kubectl run my-pod --image=nginx --labels="app=test,env=dev"

# Ephemeral debug containers (K8s 1.23+)
kubectl debug -it my-pod --image=busybox --target=my-container
```

### Pod Lifecycle Management

```bash
# Get pod events
kubectl describe pod my-pod | grep -A 10 Events

# Copy files to/from pod
kubectl cp my-pod:/app/logs/app.log ./app.log
kubectl cp ./config.yaml my-pod:/app/config/config.yaml
kubectl cp my-pod:/app/data -c my-container ./local-data/  # With container name

# Attach to a running process
kubectl attach my-pod -c my-container -it
```

---

## 7. Service Commands

### Viewing Services

```bash
# List services
kubectl get services
kubectl get svc                                   # Shorthand
kubectl get svc -o wide
kubectl get endpoints                             # See pod IPs behind each service
kubectl get ep my-service                         # Endpoints for specific service

# Describe a service
kubectl describe service my-service
kubectl describe svc my-service
```

### Creating Services

```bash
# Expose a deployment as a service
kubectl expose deployment my-app --port=80 --target-port=3000
kubectl expose deployment my-app --port=80 --target-port=3000 --type=LoadBalancer
kubectl expose deployment my-app --port=80 --target-port=3000 --type=NodePort

# Service YAML (ClusterIP)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
EOF

# Service YAML (LoadBalancer with annotations for AWS)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  selector:
    app: my-app
  ports:
    - port: 443
      targetPort: 8443
  type: LoadBalancer
EOF
```

### DNS Resolution

```bash
# Service DNS format inside cluster:
# <service-name>.<namespace>.svc.cluster.local
# Examples:
#   my-service.default.svc.cluster.local
#   postgres.database.svc.cluster.local

# Test DNS from a pod
kubectl run dns-test --rm -it --image=busybox -- nslookup my-service
kubectl run dns-test --rm -it --image=busybox -- nslookup my-service.my-namespace
```

---

## 8. Deployment Commands

### Viewing Deployments

```bash
# List deployments
kubectl get deployments
kubectl get deploy                                # Shorthand
kubectl get deploy -o wide

# Describe deployment
kubectl describe deployment my-app
kubectl describe deploy my-app

# View ReplicaSets managed by deployment
kubectl get replicasets
kubectl get rs
kubectl get rs -l app=my-app
```

### Creating & Updating Deployments

```bash
# Create a deployment
kubectl create deployment my-app --image=my-app:v1 --replicas=3

# Update an image (triggers rolling update)
kubectl set image deployment/my-app my-app=my-app:v2
kubectl set image deployment/my-app my-app=my-app:v2 --record  # (deprecated but common)

# Update resources
kubectl set resources deployment my-app \
  --limits=cpu=500m,memory=512Mi \
  --requests=cpu=100m,memory=128Mi

# Patch a deployment (inline JSON)
kubectl patch deployment my-app -p '{"spec":{"replicas":5}}'
kubectl patch deployment my-app --type=json \
  -p='[{"op":"replace","path":"/spec/replicas","value":5}]'

# Edit deployment in-place (opens in $EDITOR)
kubectl edit deployment my-app

# Apply from file
kubectl apply -f deployment.yaml
```

---

## 9. Scaling & Rollout Commands

### Scaling

```bash
# Scale a deployment
kubectl scale deployment my-app --replicas=5
kubectl scale deployment my-app --replicas=0      # Scale to zero (pause workload)

# Scale multiple deployments
kubectl scale deployment frontend backend --replicas=3

# Autoscaling (HPA - Horizontal Pod Autoscaler)
kubectl autoscale deployment my-app --min=2 --max=10 --cpu-percent=70

# View HPA
kubectl get hpa
kubectl describe hpa my-app

# Delete HPA
kubectl delete hpa my-app
```

### Rollouts

```bash
# Check rollout status
kubectl rollout status deployment/my-app

# View rollout history
kubectl rollout history deployment/my-app
kubectl rollout history deployment/my-app --revision=2   # Details for revision 2

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to specific revision
kubectl rollout undo deployment/my-app --to-revision=2

# Pause a rollout
kubectl rollout pause deployment/my-app

# Resume a paused rollout
kubectl rollout resume deployment/my-app

# Restart a deployment (re-pulls images, bounces pods)
kubectl rollout restart deployment/my-app
kubectl rollout restart deployment/my-app -n production
```

### Full Rollout Workflow Example

```bash
# 1. Check current state
kubectl get deploy my-app
kubectl rollout history deployment/my-app

# 2. Update image to new version
kubectl set image deployment/my-app my-app=my-app:v3

# 3. Watch rollout in real-time
kubectl rollout status deployment/my-app -w

# 4. Verify new pods are running
kubectl get pods -l app=my-app

# 5. If something is wrong, rollback
kubectl rollout undo deployment/my-app

# 6. Verify rollback succeeded
kubectl rollout status deployment/my-app
kubectl get pods -l app=my-app
```

---

## 10. Namespace Management

Namespaces provide virtual clusters within a physical cluster.

```bash
# List namespaces
kubectl get namespaces
kubectl get ns                                        # Shorthand

# Create a namespace
kubectl create namespace my-namespace
kubectl create ns staging

# Create from YAML
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
EOF

# Delete a namespace (DELETES ALL RESOURCES INSIDE)
kubectl delete namespace staging

# Set default namespace for current context
kubectl config set-context --current --namespace=my-namespace

# Run commands in a specific namespace
kubectl get pods -n my-namespace
kubectl apply -f deployment.yaml -n my-namespace
kubectl logs my-pod -n my-namespace

# Get resources across ALL namespaces
kubectl get pods -A
kubectl get pods --all-namespaces

# Namespace-scoped RBAC example
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: my-namespace
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
EOF
```

---

## 11. Port Forwarding

Port forwarding lets you access cluster resources from your local machine without exposing them publicly.

```bash
# Forward local port 8080 → pod port 3000
kubectl port-forward pod/my-pod 8080:3000

# Forward to a deployment (any pod in the deployment)
kubectl port-forward deployment/my-app 8080:3000

# Forward to a service
kubectl port-forward service/my-service 8080:80
kubectl port-forward svc/my-service 8080:80

# Forward to a pod in a specific namespace
kubectl port-forward pod/my-pod 8080:3000 -n my-namespace

# Multiple ports
kubectl port-forward pod/my-pod 8080:3000 9090:9000

# Bind to all interfaces (accessible from network, not just localhost)
kubectl port-forward --address 0.0.0.0 pod/my-pod 8080:3000

# Run in background
kubectl port-forward pod/my-pod 8080:3000 &
# Kill later:
kill $(lsof -ti:8080)

# Common use cases:
kubectl port-forward svc/kubernetes-dashboard 8443:443 -n kubernetes-dashboard
kubectl port-forward svc/prometheus-server 9090:80 -n monitoring
kubectl port-forward svc/grafana 3000:80 -n monitoring
kubectl port-forward svc/postgres 5432:5432 -n database
```

---

## 12. Debugging & Troubleshooting

### Pod Debugging Workflow

```bash
# Step 1: See pod status
kubectl get pods
# STATUS meanings:
#   Pending      → Not scheduled yet (insufficient resources, no nodes match)
#   Running      → At least one container running
#   Completed    → All containers exited successfully
#   CrashLoopBackOff → Container keeps crashing (check logs!)
#   ImagePullBackOff → Can't pull image (wrong name, no credentials)
#   Terminating  → Being deleted
#   OOMKilled    → Out of memory (increase memory limits)

# Step 2: Describe the pod (look at Events section)
kubectl describe pod my-pod

# Step 3: Check logs
kubectl logs my-pod
kubectl logs my-pod --previous                    # If pod restarted

# Step 4: Shell in if running
kubectl exec -it my-pod -- /bin/sh

# Step 5: Debug with ephemeral container (K8s 1.23+)
kubectl debug -it my-pod --image=busybox --target=my-container
```

### Node Debugging

```bash
# Check node conditions
kubectl describe node my-node | grep -A 20 Conditions

# Check resource usage on nodes
kubectl top nodes                                 # (metrics-server required)
kubectl top pods
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory
kubectl top pods -A

# Node is NotReady — common checks:
kubectl describe node my-node | grep -A 5 "Conditions"
kubectl get events --field-selector involvedObject.name=my-node

# Cordon a node (prevent new pods from scheduling)
kubectl cordon my-node

# Drain a node (safely evict all pods for maintenance)
kubectl drain my-node --ignore-daemonsets --delete-emptydir-data

# Uncordon a node (allow pods again)
kubectl uncordon my-node
```

### Cluster Events & Logs

```bash
# Get cluster events (sorted by time)
kubectl get events --sort-by='.metadata.creationTimestamp'

# Filter events by type
kubectl get events --field-selector type=Warning
kubectl get events --field-selector reason=BackOff

# Get events for a specific resource
kubectl get events --field-selector involvedObject.name=my-pod

# Watch events in real-time
kubectl get events -w

# Get events in a namespace
kubectl get events -n my-namespace
```

### Network Debugging

```bash
# Test connectivity between pods
kubectl run test-curl --rm -it --image=curlimages/curl -- \
  curl -v http://my-service.default.svc.cluster.local

# DNS debugging
kubectl run dns-debug --rm -it --image=tutum/dnsutils -- \
  nslookup my-service.default.svc.cluster.local

# Check kube-dns/CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check service endpoints
kubectl get endpoints my-service                   # Empty = no pods match selector
kubectl describe endpoints my-service

# Netshoot (all-in-one network debug image)
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- bash
# Inside netshoot: nmap, tcpdump, netstat, curl, dig, ping, iperf all available
```

### Resource & Capacity Issues

```bash
# Check resource requests vs limits
kubectl describe pod my-pod | grep -A 10 Limits
kubectl describe node my-node | grep -A 20 "Allocated resources"

# Check if pods have been OOMKilled
kubectl get pods -o json | jq '.items[] | select(.status.containerStatuses[]?.lastState.terminated.reason=="OOMKilled") | .metadata.name'

# Check pending pods and why
kubectl describe pod my-pending-pod | grep -A 10 Events

# View PersistentVolumes and Claims
kubectl get pv
kubectl get pvc
kubectl describe pvc my-claim
```

---

## 13. ConfigMaps & Secrets

### ConfigMaps

```bash
# Create ConfigMap from literals
kubectl create configmap app-config \
  --from-literal=DB_HOST=postgres \
  --from-literal=DB_PORT=5432 \
  --from-literal=LOG_LEVEL=info

# Create from file
kubectl create configmap app-config --from-file=config.yaml
kubectl create configmap app-config --from-file=./config/     # All files in dir

# View ConfigMaps
kubectl get configmaps
kubectl get cm
kubectl describe cm app-config
kubectl get cm app-config -o yaml

# Use in Pod as environment variables
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: my-app
      image: my-app:v1
      envFrom:
        - configMapRef:
            name: app-config           # Inject all keys as env vars
      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_HOST             # Inject specific key
EOF

# Mount as volume (file in container)
cat <<EOF
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
EOF
```

### Secrets

```bash
# Create Secret (base64 encoded automatically)
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=s3cr3t!

# Create from file
kubectl create secret generic tls-secret \
  --from-file=tls.crt=./cert.crt \
  --from-file=tls.key=./cert.key

# Create TLS secret
kubectl create secret tls my-tls \
  --cert=tls.crt \
  --key=tls.key

# Create Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=my-user \
  --docker-password=my-pass \
  --docker-email=me@example.com

# View secrets (values are base64 encoded)
kubectl get secrets
kubectl describe secret db-secret
kubectl get secret db-secret -o yaml

# Decode a secret value
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode

# Use in Pod
cat <<EOF
  env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
EOF
```

---

## 14. Cluster Cleanup Commands

### Targeted Cleanup

```bash
# Delete specific resources
kubectl delete pod my-pod
kubectl delete deployment my-app
kubectl delete service my-service
kubectl delete configmap app-config
kubectl delete secret db-secret
kubectl delete ingress my-ingress

# Delete by label
kubectl delete pods -l app=my-app
kubectl delete all -l app=my-app                  # pods, deployments, services, etc.

# Delete from manifest
kubectl delete -f deployment.yaml
kubectl delete -f ./k8s/

# Delete all pods in a namespace (Deployment will recreate them)
kubectl delete pods --all -n my-namespace

# Force delete a stuck pod
kubectl delete pod my-pod --grace-period=0 --force

# Delete completed/failed jobs and their pods
kubectl delete jobs --field-selector status.successful=1
kubectl delete pod --field-selector status.phase=Failed
kubectl delete pod --field-selector status.phase=Succeeded
```

### Namespace Cleanup

```bash
# Delete an entire namespace (ALL resources inside are deleted)
kubectl delete namespace staging
kubectl delete ns dev testing staging               # Multiple at once

# Clean up all resources in a namespace (keep the namespace)
kubectl delete all --all -n my-namespace

# Delete everything in current namespace
kubectl delete all --all
```

### Cluster-wide Cleanup

```bash
# Remove evicted pods across all namespaces
kubectl get pods -A | grep Evicted | awk '{print $2, "-n", $1}' | xargs kubectl delete pod

# Remove all pods not running (Failed, Completed, Evicted)
kubectl get pods -A | grep -v Running | grep -v Pending | grep -v NAME | \
  awk '{print "kubectl delete pod " $2 " -n " $1}' | bash

# Clean up old ReplicaSets (no replicas running)
kubectl get rs -A | awk '$3==0 && $4==0 {print $2, "-n", $1}' | xargs kubectl delete rs

# Remove all completed jobs
kubectl delete jobs --field-selector status.successful=1 -A

# Clean up PVCs in lost/Released state
kubectl get pvc -A | grep -v Bound | grep -v NAME | \
  awk '{print "kubectl delete pvc " $2 " -n " $1}' | bash

# Full reset (nuclear option — removes all non-system resources)
kubectl delete all --all --all-namespaces
```

---

## 15. MERN Stack on Kubernetes

A production-ready MERN (MongoDB, Express, React, Node.js) architecture on Kubernetes.

### Architecture Overview

```
                        Internet
                            │
                            ▼
                    ┌───────────────┐
                    │    Ingress    │
                    │   (nginx)     │
                    │  myapp.com    │
                    └───────┬───────┘
                            │
               ┌────────────┴──────────────┐
               │                           │
               ▼                           ▼
    ┌─────────────────────┐     ┌─────────────────────┐
    │   React Frontend    │     │    Express API       │
    │   Deployment (3)    │     │    Deployment (3)    │
    │   Service:ClusterIP │     │   Service:ClusterIP  │
    │   Port: 80          │     │   Port: 3000         │
    └─────────────────────┘     └──────────┬──────────┘
                                           │
                    ┌──────────────────────┤
                    │                      │
                    ▼                      ▼
         ┌─────────────────┐    ┌─────────────────┐
         │    MongoDB       │    │   Redis Cache   │
         │  StatefulSet (1) │    │  Deployment (1) │
         │  Service:ClusterIP│   │  Service:ClusterIP│
         │  Port: 27017    │    │  Port: 6379     │
         │  PVC: 10Gi      │    └─────────────────┘
         └─────────────────┘
```

### Directory Structure

```
k8s/
├── namespaces/
│   └── namespace.yaml
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
│   └── pvc.yaml
├── redis/
│   ├── deployment.yaml
│   └── service.yaml
├── secrets/
│   └── secrets.yaml
└── ingress/
    └── ingress.yaml
```

### Namespace

```yaml
# k8s/namespaces/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mern-app
```

### MongoDB StatefulSet

```yaml
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
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: password
          volumeMounts:
            - name: mongo-data
              mountPath: /data/db
          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "1"
              memory: "2Gi"
  volumeClaimTemplates:
    - metadata:
        name: mongo-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  namespace: mern-app
spec:
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
  clusterIP: None                        # Headless service for StatefulSet
```

### Express Backend

```yaml
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
          image: my-registry/mern-backend:v1
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
            - name: PORT
              value: "3000"
            - name: MONGO_URI
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: connection-string
            - name: REDIS_URL
              value: "redis://redis-service:6379"
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: jwt-secret
          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          readinessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 15
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: mern-app
spec:
  selector:
    app: backend
  ports:
    - port: 3000
      targetPort: 3000
  type: ClusterIP
```

### React Frontend

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
          image: my-registry/mern-frontend:v1
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "50m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "256Mi"
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
---
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
  type: ClusterIP
```

### Ingress

```yaml
# k8s/ingress/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mern-ingress
  namespace: mern-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - myapp.com
      secretName: myapp-tls
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 3000
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### Secrets

```yaml
# k8s/secrets/secrets.yaml
# NOTE: Never commit real secrets! Use sealed-secrets, Vault, or external-secrets.
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: mern-app
type: Opaque
stringData:                           # stringData auto-encodes to base64
  username: admin
  password: changeme-use-a-real-password
  connection-string: "mongodb://admin:changeme@mongodb:27017/mydb?authSource=admin"
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: mern-app
type: Opaque
stringData:
  jwt-secret: changeme-use-a-real-jwt-secret
```

### HPA (Horizontal Pod Autoscaler)

```yaml
# k8s/backend/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: mern-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Deploy the Full MERN Stack

```bash
# Deploy everything
kubectl apply -f k8s/namespaces/
kubectl apply -f k8s/secrets/
kubectl apply -f k8s/mongodb/
kubectl apply -f k8s/redis/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/
kubectl apply -f k8s/ingress/

# Check all resources
kubectl get all -n mern-app

# Watch rollout
kubectl rollout status deployment/backend -n mern-app
kubectl rollout status deployment/frontend -n mern-app
```

---

## 16. Developer Workflow: Docker → Kubernetes

### Complete End-to-End Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                   DEVELOPER WORKFLOW                                │
│                                                                     │
│  1. Code  →  2. Dockerfile  →  3. docker build  →  4. docker push  │
│                                                                     │
│  5. Update image tag in YAML  →  6. kubectl apply  →  7. Verify    │
│                                                                     │
│  (Ideally automated via CI/CD pipeline)                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Step 1 — Write Your Dockerfile

```dockerfile
# Multi-stage build for Node.js app
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

### Step 2 — Build & Tag Image

```bash
# Build with a tag
docker build -t my-registry/my-app:v1.2.3 .
docker build -t my-registry/my-app:latest .

# Build for multiple platforms (for ARM + x86 clusters)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t my-registry/my-app:v1.2.3 \
  --push .

# Check image size
docker images my-registry/my-app
```

### Step 3 — Push to Registry

```bash
# Login to registry
docker login registry.example.com
docker login -u myuser -p mypass registry.example.com

# Push image
docker push my-registry/my-app:v1.2.3
docker push my-registry/my-app:latest

# Tag and push to ECR (AWS)
aws ecr get-login-password | docker login --username AWS \
  --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
docker tag my-app:v1 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1
```

### Step 4 — Update YAML & Apply

```bash
# Option A: Edit YAML and apply
vim k8s/backend/deployment.yaml   # Update image tag
kubectl apply -f k8s/backend/deployment.yaml

# Option B: Use kubectl set image (no YAML edit needed)
kubectl set image deployment/backend \
  backend=my-registry/my-app:v1.2.3 \
  -n mern-app

# Option C: Use envsubst for templating (CI/CD friendly)
export IMAGE_TAG=v1.2.3
envsubst '$IMAGE_TAG' < deployment.template.yaml | kubectl apply -f -
```

### Step 5 — Verify Deployment

```bash
# Watch rollout
kubectl rollout status deployment/backend -n mern-app

# Check pods are running
kubectl get pods -n mern-app -l app=backend

# Verify new image is deployed
kubectl describe pod -l app=backend -n mern-app | grep Image

# Quick smoke test via port-forward
kubectl port-forward svc/backend-service 3000:3000 -n mern-app &
curl http://localhost:3000/api/health
kill %1
```

### Step 6 — Monitor

```bash
# Watch pod resource usage
kubectl top pods -n mern-app

# Stream all backend logs
kubectl logs -f -l app=backend -n mern-app --all-containers=true

# Check events for any issues
kubectl get events -n mern-app --sort-by='.lastTimestamp'
```

### Step 7 — Rollback if Needed

```bash
# Check history
kubectl rollout history deployment/backend -n mern-app

# Rollback to previous
kubectl rollout undo deployment/backend -n mern-app

# Rollback to specific revision
kubectl rollout undo deployment/backend --to-revision=3 -n mern-app

# Confirm rollback
kubectl rollout status deployment/backend -n mern-app
```

### CI/CD Pipeline Snippet (GitHub Actions)

```yaml
# .github/workflows/deploy.yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set image tag
        run: echo "IMAGE_TAG=${GITHUB_SHA::8}" >> $GITHUB_ENV

      - name: Build and push Docker image
        run: |
          docker build -t ${{ secrets.REGISTRY }}/my-app:${{ env.IMAGE_TAG }} .
          docker push ${{ secrets.REGISTRY }}/my-app:${{ env.IMAGE_TAG }}

      - name: Configure kubectl
        uses: azure/setup-kubectl@v3

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/backend \
            backend=${{ secrets.REGISTRY }}/my-app:${{ env.IMAGE_TAG }} \
            -n mern-app
          kubectl rollout status deployment/backend -n mern-app
```

---

## Quick Reference Cheatsheet

```
PODS                               DEPLOYMENTS
──────────────────────────────     ────────────────────────────────
kubectl get pods                   kubectl get deployments
kubectl get pods -o wide           kubectl describe deploy <name>
kubectl describe pod <name>        kubectl apply -f deploy.yaml
kubectl logs <pod>                 kubectl set image deploy/<name> <c>=<img>
kubectl logs <pod> -f              kubectl rollout status deploy/<name>
kubectl logs <pod> --previous      kubectl rollout undo deploy/<name>
kubectl exec -it <pod> -- sh       kubectl rollout history deploy/<name>
kubectl delete pod <name>          kubectl scale deploy/<name> --replicas=5
kubectl cp <pod>:/path ./local     kubectl autoscale deploy/<name> --max=10

SERVICES                           NAMESPACES
──────────────────────────────     ────────────────────────────────
kubectl get svc                    kubectl get ns
kubectl describe svc <name>        kubectl create ns <name>
kubectl expose deploy/<name> \     kubectl delete ns <name>
  --port=80 --type=LoadBalancer    kubectl get all -n <name>
kubectl get endpoints              kubectl config set-context \
                                     --current --namespace=<name>

DEBUGGING                          PORT FORWARDING
──────────────────────────────     ────────────────────────────────
kubectl get events --sort-by=\     kubectl port-forward pod/<name> 8080:3000
  .metadata.creationTimestamp      kubectl port-forward svc/<name> 8080:80
kubectl top pods                   kubectl port-forward deploy/<name> 8080:3000
kubectl top nodes
kubectl describe node <name>
kubectl drain <node> --ignore-daemonsets
kubectl cordon / uncordon <node>
```

---

*Last updated: 2025 · Kubernetes v1.29+ · kubectl v1.29+*
