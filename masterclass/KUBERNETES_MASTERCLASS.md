# Kubernetes (K8s) Masterclass: The Orchestration Guide (Zero to Expert) ☸️

## Table of Contents
1.  [Introduction & Architecture](#1-introduction--architecture)
2.  [The Core Objects (The Building Blocks)](#2-the-core-objects-the-building-blocks)
3.  [Services & Networking](#3-services--networking)
4.  [Storage & Configuration](#4-storage--configuration)
5.  [Advanced Workloads](#5-advanced-workloads)
6.  [Essential Commands (Kubectl)](#6-essential-commands-kubectl)
7.  [Production Standards (Probes & Limits)](#7-production-standards-probes--limits)
8.  [Helm (Package Manager)](#8-helm-package-manager)

---

## 1. Introduction & Architecture

### The "Why"
Docker manages containers. Kubernetes manages **clusters** of servers running containers. It handles:
*   **Healing:** Restarting failed containers.
*   **Scaling:** Adding more replicas during traffic spikes.
*   **Updates:** Rolling out new versions without downtime.

### The Architecture (Control Plane vs Workers)

#### 1. The Control Plane (The Brain)
*   **API Server:** The entry point. All administrative tasks (kubectl commands) go here.
*   **etcd:** The database. A highly available key-value store that holds the entire state of the cluster.
*   **Scheduler:** Decides *where* to put a new Pod (based on CPU/RAM availability).
*   **Controller Manager:** The enforcer. It notices if state doesn't match desire (e.g., "I wanted 3 pods, but found 2") and fixes it.

#### 2. The Worker Nodes (The Muscle)
*   **Kubelet:** The agent running on every node. It talks to the API server and starts/stops containers.
*   **Kube-proxy:** Handles networking rules (IP tables) so services can be reached.
*   **Container Runtime:** The actual software running the container (Docker, containerd, CRI-O).

---

## 2. The Core Objects (The Building Blocks)

### Pods (The Atom)
The smallest deployable unit.
*   Usually contains 1 container (e.g., Node.js app).
*   Can contain "Sidecars" (helpers like loggers).
*   **Ephemeral:** If a node dies, the Pod dies. It is NOT resurrected. It is replaced by a *new* Pod.

### Deployments (The Manager)
You rarely create Pods directly. You create a **Deployment**.
*   Manages **ReplicaSets**.
*   Ensures a specific number of copies are running.
*   Handles **Rolling Updates** (Update v1 -> v2 one by one).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template: # The Pod Template
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: react-app
        image: my-react-app:v2
        ports:
        - containerPort: 80
```

---

## 3. Services & Networking

How do users (or other apps) talk to your Pods? Pod IPs change constantly. **Services** provide a stable IP.

### Service Types
1.  **ClusterIP (Default):** Internal only. Good for DBs or backend APIs.
2.  **NodePort:** Opens a specific port (30000-32767) on *every* Node IP. Good for testing.
3.  **LoadBalancer:** Asks your Cloud Provider (AWS/GCP) for a real Public IP/Load Balancer.
4.  **ExternalName:** Maps a service to a DNS name.

### Ingress (The Router)
A "Service" gives an IP. An **Ingress** gives a URL (`my-app.com`) and routing (`/api` goes to backend, `/` goes to frontend).
*   Requires an **Ingress Controller** (like Nginx) to be running in the cluster.

---

## 4. Storage & Configuration

### ConfigMaps & Secrets
Don't hardcode config.
*   **ConfigMap:** Non-sensitive data (URLs, feature flags). Injected as Environment Variables or Files.
*   **Secret:** Sensitive data (passwords, keys). Encoded (Base64) and encrypted at rest (ideally).

### Persistent Volumes (PV) & Claims (PVC)
Pods are ephemeral. Storage shouldn't be.
1.  **Administrator** creates a `PersistentVolume` (e.g., a 100GB AWS EBS drive).
2.  **Developer** creates a `PersistentVolumeClaim` ("I need 10GB").
3.  Kubernetes binds them together.

---

## 5. Advanced Workloads

1.  **StatefulSet:** For databases (Postgres, Mongo). Guarantees stable network IDs (`db-0`, `db-1`) and ordered deployment.
2.  **DaemonSet:** Runs exactly ONE copy of a Pod on EVERY node. (Perfect for Logs collectors like Fluentd or Monitoring agents).
3.  **Job / CronJob:** Runs a task to completion (Batch processing, Backups) then exits.

---

## 6. Essential Commands (Kubectl)

```bash
# Apply a change (The most used command)
kubectl apply -f my-file.yaml

# Get Status
kubectl get pods
kubectl get svc
kubectl get deployments
kubectl get all

# Debugging
kubectl describe pod <pod-name>  # Why is it pending/crashing?
kubectl logs <pod-name>          # App output
kubectl logs -f <pod-name>       # Follow logs
kubectl exec -it <pod-name> -- sh # Jump inside (SSH-like)

# Scaling
kubectl scale deployment/frontend --replicas=5

# Rollout status
kubectl rollout status deployment/frontend
kubectl rollout undo deployment/frontend # Oops, v2 broke. Go back to v1.
```

---

## 7. Production Standards (Probes & Limits)

If you don't use these, your cluster is unstable.

### Resource Requests & Limits
Tell K8s how much CPU/RAM you need.
*   **Request:** "I need at least this much to start." (Used for scheduling).
*   **Limit:** "Kill me if I use more than this." (Safety).

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m" # 1/4 of a core
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### Probes (Health Checks)
1.  **Liveness Probe:** "Is the app deadlock/frozen?" -> If No, K8s restarts it.
2.  **Readiness Probe:** "Is the app ready to take traffic?" (e.g., connected to DB) -> If No, K8s stops sending traffic (but doesn't restart it).
3.  **Startup Probe:** "Has the slow legacy app finished booting?"

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

---

## 8. Helm (Package Manager)

Writing 50 YAML files is hard. **Helm** is "NPM for Kubernetes".
*   **Chart:** A package of YAML templates.
*   **Values.yaml:** You just fill in the blanks (image version, replicas).

```bash
helm install my-release my-chart/
```

**Common usage:** "I want to install Prometheus". You don't write 100 YAMLs. You just `helm install prometheus`.
