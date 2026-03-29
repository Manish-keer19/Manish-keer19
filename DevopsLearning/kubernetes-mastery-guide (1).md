# Kubernetes Mastery Guide
## Beginner → Intermediate → Advanced → Production → Expert → Master

> **Author:** Senior Kubernetes Architect & DevOps Engineer  
> **Experience:** 15+ Years Large-Scale Production Kubernetes  
> **Purpose:** Complete Professional Learning Curriculum  

---

## Table of Contents

### Level 1 — Beginner
- [Chapter 1 — Kubernetes Introduction](#chapter-1--kubernetes-introduction)
- [Chapter 2 — Kubernetes Architecture](#chapter-2--kubernetes-architecture)
- [Chapter 3 — Kubernetes Installation](#chapter-3--kubernetes-installation)
- [Chapter 4 — kubectl Basics](#chapter-4--kubectl-basics)
- [Chapter 5 — Pods](#chapter-5--pods)

### Level 2 — Intermediate
- [Chapter 6 — Deployments](#chapter-6--deployments)
- [Chapter 7 — ReplicaSets](#chapter-7--replicasets)
- [Chapter 8 — Services](#chapter-8--services)
- [Chapter 9 — Labels & Selectors](#chapter-9--labels--selectors)
- [Chapter 10 — ConfigMaps](#chapter-10--configmaps)
- [Chapter 11 — Secrets](#chapter-11--secrets)
- [Chapter 12 — Namespaces](#chapter-12--namespaces)

### Level 3 — Advanced
- [Chapter 13 — Volumes](#chapter-13--volumes)
- [Chapter 14 — Persistent Volumes](#chapter-14--persistent-volumes)
- [Chapter 15 — Storage Classes](#chapter-15--storage-classes)
- [Chapter 16 — Ingress](#chapter-16--ingress)
- [Chapter 17 — Networking](#chapter-17--networking)
- [Chapter 18 — Resource Limits](#chapter-18--resource-limits)
- [Chapter 19 — Health Checks](#chapter-19--health-checks)

### Level 4 — Production
- [Chapter 20 — Rolling Updates](#chapter-20--rolling-updates)
- [Chapter 21 — Autoscaling](#chapter-21--autoscaling)
- [Chapter 22 — Jobs](#chapter-22--jobs)
- [Chapter 23 — CronJobs](#chapter-23--cronjobs)
- [Chapter 24 — RBAC](#chapter-24--rbac)
- [Chapter 25 — Security](#chapter-25--security)

### Level 5 — Expert
- [Chapter 26 — Helm](#chapter-26--helm)
- [Chapter 27 — Monitoring](#chapter-27--monitoring)
- [Chapter 28 — Logging](#chapter-28--logging)
- [Chapter 29 — Debugging](#chapter-29--debugging)
- [Chapter 30 — Production Architecture](#chapter-30--production-architecture)
- [Chapter 31 — Multi-Node Cluster](#chapter-31--multi-node-cluster)
- [Chapter 32 — Kubernetes Internals](#chapter-32--kubernetes-internals)

### Level 6 — Master
- [Chapter 33 — Networking Deep Dive](#chapter-33--networking-deep-dive)
- [Chapter 34 — Service Mesh](#chapter-34--service-mesh)
- [Chapter 35 — Performance Optimization](#chapter-35--performance-optimization)
- [Chapter 36 — CI/CD Kubernetes](#chapter-36--cicd-kubernetes)
- [Chapter 37 — GitOps](#chapter-37--gitops)
- [Chapter 38 — Multi-Cluster](#chapter-38--multi-cluster)
- [Chapter 39 — Kubernetes at Scale](#chapter-39--kubernetes-at-scale)

### Projects & Reference
- [Real World Projects](#real-world-projects)
- [Kubernetes Debugging Handbook](#kubernetes-debugging-handbook)
- [Production Folder Structure](#production-folder-structure)
- [Kubernetes Mastery Checklist](#kubernetes-mastery-checklist)

---

# LEVEL 1 — BEGINNER

---

## Chapter 1 — Kubernetes Introduction

### What You Will Learn
In this chapter you will understand what Kubernetes is, why it was created, what problems it solves, and where it sits in the modern software delivery world. You will also learn the mental model that every Kubernetes engineer carries in their head every single day.

### Why This Exists

Before Kubernetes, deploying applications meant:

- Manually SSH-ing into servers and running scripts
- Managing dependency conflicts between apps on the same machine
- No automatic recovery when a process crashed
- Manually scaling by buying new servers and configuring them
- Inconsistent environments between development and production
- Downtime during deployments

Docker solved the packaging problem. It let developers say "here is my app and everything it needs, wrapped in a container." But Docker alone doesn't tell you:

- Which machine should run this container?
- What happens if the machine goes down?
- How do you run 50 copies of the same container?
- How do containers talk to each other across machines?
- How do you update your app without downtime?

Kubernetes answers all of these questions. It is a **container orchestration platform** — a system that manages where, when, and how containers run across a fleet of machines.

Google built Kubernetes based on their internal system called **Borg**, which orchestrated billions of containers per week. They open-sourced it in 2014 and donated it to the Cloud Native Computing Foundation (CNCF) in 2016.

### How It Works Internally

Kubernetes works by maintaining a **desired state**. You tell Kubernetes what you want — "I want 3 copies of my web server running" — and Kubernetes makes it happen and keeps it that way forever.

This is called a **declarative model**:

```
You declare:  "I want 3 replicas of my app"
K8s observes: "There are currently 2 replicas"
K8s acts:     "Starting 1 more replica..."
K8s confirms: "Now there are 3 replicas ✓"
```

This loop runs continuously. Kubernetes never stops watching. If a container crashes, a node disappears, or network fails — Kubernetes detects the deviation from desired state and corrects it automatically.

### Architecture Explanation

```
┌─────────────────────────────────────────────────────────┐
│                   KUBERNETES CLUSTER                      │
│                                                           │
│  ┌─────────────────┐      ┌─────────────────────────┐   │
│  │   CONTROL PLANE  │      │        WORKER NODES      │   │
│  │   (Master Node)  │      │                         │   │
│  │                 │      │  ┌───────┐  ┌───────┐   │   │
│  │  API Server     │◄────►│  │ Node1 │  │ Node2 │   │   │
│  │  etcd           │      │  │       │  │       │   │   │
│  │  Scheduler      │      │  │ Pod   │  │ Pod   │   │   │
│  │  Controller Mgr │      │  │ Pod   │  │ Pod   │   │   │
│  │                 │      │  └───────┘  └───────┘   │   │
│  └─────────────────┘      └─────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Real World Example

Netflix runs thousands of microservices. Each microservice is a small application that does one job — authentication, recommendations, video streaming, billing, etc. Kubernetes orchestrates all of these services across thousands of machines. When Black Mirror releases a new season and millions of users suddenly log in simultaneously, Kubernetes automatically scales up the relevant services within seconds to handle the load, then scales them back down afterwards.

### Key Kubernetes Concepts — The Mental Model

| Concept | What It Is | Real World Analogy |
|---|---|---|
| Cluster | Group of machines managed by K8s | A fleet of ships |
| Node | A single machine in the cluster | One ship in the fleet |
| Pod | Smallest deployable unit (wraps containers) | A shipping container on a ship |
| Deployment | Manages desired number of Pod copies | A shipping manifest saying "deliver 5 boxes" |
| Service | Stable network endpoint for Pods | The dock where trucks pick up containers |
| Namespace | Virtual partition inside a cluster | A separate section of the warehouse |
| ConfigMap | Configuration data for Pods | A settings file |
| Secret | Sensitive configuration (passwords, tokens) | A locked vault |

### Commands

```bash
# Check kubectl is installed
kubectl version --client

# Check cluster connection
kubectl cluster-info

# List all nodes in cluster
kubectl get nodes

# See everything in the cluster
kubectl get all --all-namespaces
```

### Practical Exercises

**Exercise 1.1 — Verify Your Understanding**
After reading this chapter, answer these questions without looking at notes:
1. What problem does Kubernetes solve that Docker alone cannot?
2. What does "desired state" mean?
3. What is the difference between a cluster and a node?

**Exercise 1.2 — Research**
Look up the CNCF landscape at landscape.cncf.io and identify 3 tools that work alongside Kubernetes.

### Production Use Case

At a fintech company running 200+ microservices, Kubernetes manages every single workload. When a payment processing service crashes at 2 AM, Kubernetes restarts it within seconds — before any on-call engineer is even paged. The system self-heals. That is the core value of Kubernetes in production.

### Common Mistakes

**Mistake 1: Thinking Kubernetes is just Docker with extra steps**
Kubernetes is fundamentally different. Docker runs containers on one machine. Kubernetes orchestrates containers across hundreds of machines with automatic healing, scaling, and networking.

**Mistake 2: Using Kubernetes for every project**
Kubernetes has operational overhead. A simple personal project or small startup with one or two services doesn't need Kubernetes. Use it when you have scale, reliability, or team-size reasons.

**Mistake 3: Not understanding the declarative model**
Beginners try to imperatively manage Kubernetes like they manage servers. Always think: "What state do I want?" not "What commands do I need to run?"

### Best Practices

- Always learn the mental model before the commands. Understanding why makes the how obvious.
- Read the Kubernetes documentation. It is the best technical documentation in open source.
- Start with Minikube or kind locally. Never learn on production.
- Join the Kubernetes Slack community (#kubernetes-users channel).

### Debugging Section

**"I can't connect to my cluster"**
```bash
# Check your kubeconfig
kubectl config view

# Check current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>
```

---

## Chapter 2 — Kubernetes Architecture

### What You Will Learn
The internal components that make Kubernetes work, how the Control Plane communicates with Worker Nodes, what happens when you run `kubectl apply`, and the role of each component in keeping the cluster alive.

### Why This Exists

Without understanding architecture, you are flying blind. When things break in production — and they will — you need to know which component failed and why. Architecture knowledge transforms you from someone who runs commands into someone who understands what happens after each command.

### How It Works Internally

When you run `kubectl apply -f deployment.yaml`:

```
1. kubectl reads your YAML
2. kubectl sends HTTP request to API Server
3. API Server validates the YAML
4. API Server stores desired state in etcd
5. Controller Manager detects new desired state
6. Scheduler picks which node should run the Pod
7. kubelet on that node pulls the container image
8. kubelet starts the container
9. kubelet reports status back to API Server
10. API Server updates etcd with actual state
```

### Architecture Explanation

#### Control Plane Components

```
┌─────────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                             │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │  kube-        │  │    etcd      │  │  kube-controller-      │ │
│  │  apiserver    │  │              │  │  manager               │ │
│  │               │  │  Distributed │  │                        │ │
│  │  All clients  │  │  key-value   │  │  Node Controller       │ │
│  │  talk here    │  │  store       │  │  Deployment Controller │ │
│  │               │  │  Source of   │  │  ReplicaSet Controller │ │
│  │  REST API     │  │  truth       │  │  Service Controller    │ │
│  └──────┬───────┘  └──────────────┘  └────────────────────────┘ │
│         │                                                         │
│  ┌──────▼───────┐                                                │
│  │ kube-         │                                                │
│  │ scheduler     │                                                │
│  │               │                                                │
│  │ Picks which   │                                                │
│  │ node runs     │                                                │
│  │ each Pod      │                                                │
│  └──────────────┘                                                │
└─────────────────────────────────────────────────────────────────┘
```

#### Worker Node Components

```
┌─────────────────────────────────────────────────────────────────┐
│                         WORKER NODE                              │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │   kubelet    │  │  kube-proxy  │  │  Container Runtime   │  │
│  │              │  │              │  │                      │  │
│  │  Node agent  │  │  Network     │  │  containerd / CRI-O  │  │
│  │  Runs Pods   │  │  rules on    │  │  Actually runs       │  │
│  │  Reports     │  │  each node   │  │  the containers      │  │
│  │  health      │  │  (iptables)  │  │                      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                        Pods                              │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │    │
│  │  │  Container 1 │  │  Container 2 │  │  Container 3 │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Component Deep Dive

#### kube-apiserver

The API Server is the **front door** of Kubernetes. Every single operation — from `kubectl get pods` to your internal controllers — goes through the API Server. It:

- Validates all incoming requests
- Authenticates clients
- Authorizes actions based on RBAC rules
- Stores state in etcd
- Notifies other components of changes

The API Server is stateless and can be horizontally scaled for high availability.

#### etcd

etcd is a distributed key-value database. It stores **all** cluster state — what Pods exist, what Deployments are configured, what Secrets are stored, which Nodes are healthy. It is the single source of truth.

```
Everything in Kubernetes is stored in etcd as key-value pairs:

/registry/pods/default/my-app-abc123
/registry/deployments/default/my-app
/registry/services/default/my-service
/registry/secrets/default/my-secret
```

etcd uses the Raft consensus algorithm to ensure data consistency across multiple etcd instances. In production, you always run at least 3 etcd nodes (or 5 for larger clusters).

**Critical insight:** If etcd data is lost without a backup, the cluster's state is gone. Back up etcd. Always.

#### kube-scheduler

The Scheduler watches for new Pods that have no Node assigned. It then selects the best Node based on:

- Resource availability (CPU, memory)
- Node affinity/anti-affinity rules
- Pod affinity/anti-affinity rules  
- Taints and tolerations
- Resource requests and limits

```
Scheduling Decision:
Pod needs:  2 CPU, 4Gi memory
Node A has: 1 CPU free → SKIP
Node B has: 8 CPU free, 16Gi free → CANDIDATE
Node C has: 4 CPU free, 2Gi free → SKIP (not enough memory)
Result: Pod scheduled on Node B
```

#### kube-controller-manager

The Controller Manager runs a collection of controllers. Each controller is a control loop that watches the current state and takes action to reach desired state:

| Controller | What It Does |
|---|---|
| Node Controller | Notices when nodes go down, marks them NotReady |
| Deployment Controller | Ensures correct number of ReplicaSets |
| ReplicaSet Controller | Ensures correct number of Pods |
| Endpoints Controller | Links Services to Pods |
| Service Account Controller | Creates default accounts for namespaces |

#### kubelet

The kubelet runs on every worker node. It:

- Watches the API Server for Pods assigned to its node
- Tells the container runtime to start/stop containers
- Reports node and pod status back to the API Server
- Runs liveness and readiness probes

#### kube-proxy

kube-proxy maintains network rules on each node. When you send traffic to a Service IP, kube-proxy (via iptables or IPVS) routes it to the correct Pod.

#### Container Runtime

The actual software that runs containers. Kubernetes supports any runtime that implements the Container Runtime Interface (CRI):

- **containerd** — Most common, production default
- **CRI-O** — Lightweight, used in OpenShift
- **Docker Engine** — Used in development (via dockershim, now deprecated)

### Real World Example

In a production cluster at an e-commerce company:
- Control Plane runs on 3 dedicated master nodes for high availability
- 200 worker nodes across 3 availability zones
- etcd runs as a 5-node cluster with regular backups to S3
- The API Server handles 10,000+ requests per second at peak

### Commands

```bash
# See control plane components
kubectl get pods -n kube-system

# Describe a specific component
kubectl describe pod kube-apiserver-<node-name> -n kube-system

# Check component health
kubectl get componentstatuses

# View events across cluster
kubectl get events --sort-by='.metadata.creationTimestamp' -A

# Check node details
kubectl describe node <node-name>
```

### YAML Examples

```yaml
# You rarely write YAML for control plane components directly.
# But here is how to see the API Server manifest on a kubeadm cluster:

# SSH into master node:
# cat /etc/kubernetes/manifests/kube-apiserver.yaml

# It looks something like this:
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - name: kube-apiserver
    image: registry.k8s.io/kube-apiserver:v1.29.0
    command:
    - kube-apiserver
    - --advertise-address=10.0.0.1
    - --etcd-servers=https://127.0.0.1:2379
    - --secure-port=6443
    - --authorization-mode=Node,RBAC
```

### Common Mistakes

**Mistake 1: Thinking the API Server holds state**
The API Server is stateless. etcd holds the state. The API Server just processes requests and reads/writes to etcd.

**Mistake 2: Running a single etcd node in production**
One etcd node means one point of failure. Always run 3 or 5 etcd nodes in production.

**Mistake 3: Not backing up etcd**
If etcd loses data and you have no backup, your cluster configuration is gone. Automate etcd backups.

### Best Practices

- Run 3 control plane nodes for high availability in production
- Keep etcd on dedicated machines separate from worker nodes
- Never run user workloads on control plane nodes
- Monitor etcd disk I/O — it is the most sensitive component

### Hands-On Tasks

**Task 2.1:** Run `kubectl get pods -n kube-system` and identify all control plane components running in your cluster.

**Task 2.2:** Run `kubectl describe node <node-name>` and find the kubelet version, container runtime version, and available CPU/memory.

**Task 2.3:** Run `kubectl get events -A --sort-by='.metadata.creationTimestamp'` and read the last 10 events. Can you understand what each event means?

### Debugging Section

**"My cluster is not responding"**
```bash
# Check if API Server is up
curl -k https://<master-ip>:6443/healthz

# Check etcd health
kubectl exec -it etcd-<master-node> -n kube-system -- \
  etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health

# Check kubelet on a node (SSH into node)
systemctl status kubelet
journalctl -u kubelet -f
```

---

## Chapter 3 — Kubernetes Installation

### What You Will Learn
How to install Kubernetes for local development, how to install it on real servers using kubeadm, and what managed Kubernetes looks like on cloud providers.

### Why This Exists

You cannot learn Kubernetes without running it. This chapter gives you working Kubernetes environments from day one of learning to day one of production deployment.

### Installation Options

| Option | Best For | Complexity |
|---|---|---|
| Minikube | Local development, single node | Beginner |
| kind | Local CI/CD testing, multi-node sim | Beginner/Intermediate |
| k3s | Edge, IoT, lightweight production | Intermediate |
| kubeadm | Real servers, full production | Advanced |
| EKS (AWS) | Cloud production | Managed |
| GKE (Google) | Cloud production | Managed |
| AKS (Azure) | Cloud production | Managed |

### How It Works Internally

**Minikube** creates a virtual machine (or uses Docker) on your local machine and runs a full Kubernetes cluster inside it. It's one command to start, one command to stop.

**kubeadm** is the official Kubernetes bootstrapping tool. It initializes the control plane, generates certificates, and helps you join worker nodes.

**Managed Kubernetes (EKS/GKE/AKS)** — The cloud provider manages the control plane for you. You only manage worker nodes (or not at all with Fargate/Autopilot).

### Architecture Explanation

```
LOCAL DEVELOPMENT:
┌────────────────────────────────────┐
│           Your Laptop              │
│                                    │
│  ┌─────────────────────────────┐  │
│  │         Minikube            │  │
│  │                             │  │
│  │  Control Plane + Worker     │  │
│  │  (All in one VM or Docker)  │  │
│  └─────────────────────────────┘  │
└────────────────────────────────────┘

PRODUCTION (kubeadm):
┌────────────────────────────────────────────────────┐
│                   Data Center                       │
│                                                     │
│  Master1    Master2    Master3                      │
│  (Control Plane - HA)                               │
│                                                     │
│  Worker1  Worker2  Worker3 ... WorkerN              │
└────────────────────────────────────────────────────┘
```

### Option 1 — Minikube (Recommended for Beginners)

```bash
# Install Minikube on Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install kubectl (Kubernetes CLI)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Start Minikube
minikube start

# Start with specific Kubernetes version
minikube start --kubernetes-version=v1.29.0

# Start with more resources
minikube start --cpus=4 --memory=8192

# Check status
minikube status

# Access the Kubernetes dashboard
minikube dashboard

# Stop Minikube
minikube stop

# Delete the cluster
minikube delete
```

### Option 2 — kind (Kubernetes in Docker)

```bash
# Install kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create a simple cluster
kind create cluster

# Create a named cluster
kind create cluster --name my-cluster

# Create a multi-node cluster
cat > kind-config.yaml << EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
EOF
kind create cluster --config kind-config.yaml

# List clusters
kind get clusters

# Delete cluster
kind delete cluster --name my-cluster
```

### Option 3 — kubeadm (Production on Real Servers)

This installs Kubernetes on actual Linux servers (VMs, bare metal, or cloud VMs you manage).

#### Prerequisites (run on ALL nodes)

```bash
# Disable swap (Kubernetes requirement)
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Set kernel parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

# Install containerd (container runtime)
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y containerd.io

# Configure containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Install kubeadm, kubelet, kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

#### Initialize Control Plane (run ONLY on master node)

```bash
# Initialize cluster
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=<MASTER-IP>

# Set up kubectl for your user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install a Pod Network add-on (Flannel)
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# Save the join command output from kubeadm init!
# It looks like:
# kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

#### Join Worker Nodes (run on each worker)

```bash
# Run the join command from master init output
sudo kubeadm join <MASTER-IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>

# Back on master, verify nodes joined
kubectl get nodes
```

### Option 4 — Managed Kubernetes

```bash
# AWS EKS
aws eks create-cluster \
  --name my-cluster \
  --region us-east-1 \
  --kubernetes-version 1.29 \
  --role-arn arn:aws:iam::123456789:role/EKS-Cluster-Role \
  --resources-vpc-config subnetIds=subnet-xxx,subnet-yyy,securityGroupIds=sg-xxx

# Update kubeconfig for EKS
aws eks update-kubeconfig --name my-cluster --region us-east-1

# Google GKE
gcloud container clusters create my-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type n1-standard-4

# Get GKE credentials
gcloud container clusters get-credentials my-cluster --zone us-central1-a
```

### Common Mistakes

**Mistake 1: Not disabling swap for kubeadm**
kubeadm will refuse to run with swap enabled. Always disable it.

**Mistake 2: Using Docker as container runtime in new clusters**
Docker runtime support was removed in Kubernetes 1.24. Use containerd.

**Mistake 3: Forgetting to save the kubeadm join token**
The join token from `kubeadm init` expires in 24 hours. Save it or generate a new one:
```bash
kubeadm token create --print-join-command
```

### Best Practices

- For learning: Use Minikube or kind — they are disposable
- For staging: Use kubeadm on cloud VMs to mirror production
- For production: Use managed Kubernetes (EKS/GKE/AKS) — let the cloud handle the control plane
- Always pin Kubernetes versions — don't use `latest`

---

## Chapter 4 — kubectl Basics

### What You Will Learn
kubectl is your primary tool for interacting with Kubernetes. This chapter covers every common kubectl command, how to read output, how to use output formats, and how to use kubectl efficiently in production.

### Why This Exists

kubectl is to Kubernetes what git is to version control. You will use it hundreds of times per day. Mastering kubectl is not optional — it is the foundation of everything else.

### How It Works Internally

kubectl is a Go binary that reads your `~/.kube/config` file to find cluster connection details, then makes HTTPS REST API calls to the Kubernetes API Server. Every kubectl command is just an HTTP request:

```
kubectl get pods
  → GET https://api-server:6443/api/v1/namespaces/default/pods

kubectl apply -f deploy.yaml
  → POST https://api-server:6443/apis/apps/v1/namespaces/default/deployments

kubectl delete pod my-pod
  → DELETE https://api-server:6443/api/v1/namespaces/default/pods/my-pod
```

### kubectl Configuration

```bash
# View full kubeconfig
kubectl config view

# View current context (which cluster you're connected to)
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch to a different cluster
kubectl config use-context production-cluster

# Set default namespace (so you don't have to type -n every time)
kubectl config set-context --current --namespace=my-namespace

# View raw kubeconfig file
cat ~/.kube/config
```

### The Core kubectl Command Pattern

```
kubectl [command] [resource-type] [resource-name] [flags]

Examples:
kubectl get     pods                           # list all pods
kubectl get     pod       my-pod               # get specific pod
kubectl get     pod       my-pod  -o yaml      # get pod as YAML
kubectl describe pod       my-pod               # detailed info
kubectl delete  pod       my-pod               # delete pod
kubectl apply   -f        my-pod.yaml          # apply a YAML file
kubectl logs    my-pod                         # view pod logs
kubectl exec    my-pod    -- ls /              # run command in pod
```

### Get Commands — Reading Cluster State

```bash
# Get pods in default namespace
kubectl get pods

# Get pods in a specific namespace
kubectl get pods -n kube-system

# Get pods in ALL namespaces
kubectl get pods -A
kubectl get pods --all-namespaces

# Get more information (node name, IP addresses)
kubectl get pods -o wide

# Get pods and keep watching (refresh automatically)
kubectl get pods -w
kubectl get pods --watch

# Get all resource types at once
kubectl get all

# Get specific resource types
kubectl get pods,services,deployments

# Get nodes
kubectl get nodes

# Get namespaces
kubectl get namespaces

# Get services
kubectl get services
kubectl get svc         # short form

# Get deployments
kubectl get deployments
kubectl get deploy      # short form

# Common short forms:
# pods → po
# services → svc
# deployments → deploy
# replicasets → rs
# configmaps → cm
# secrets → secret
# namespaces → ns
# nodes → no
# persistentvolumes → pv
# persistentvolumeclaims → pvc
# ingresses → ing
```

### Output Formats

```bash
# Default table output
kubectl get pods

# Wide table (more columns)
kubectl get pods -o wide

# YAML format (see all fields)
kubectl get pod my-pod -o yaml

# JSON format
kubectl get pod my-pod -o json

# Custom columns
kubectl get pods -o custom-columns=\
  NAME:.metadata.name,\
  STATUS:.status.phase,\
  NODE:.spec.nodeName

# JSONPath (extract specific fields)
kubectl get pod my-pod -o jsonpath='{.status.podIP}'
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Sort output
kubectl get pods --sort-by='.metadata.creationTimestamp'
kubectl get pods --sort-by='.status.podIP'
```

### Describe — Detailed Resource Information

```bash
# Describe a pod (most important debugging command)
kubectl describe pod my-pod

# Describe a node
kubectl describe node worker-1

# Describe a deployment
kubectl describe deployment my-app

# Describe a service
kubectl describe service my-service
```

`kubectl describe` shows you:
- Resource metadata
- Current status
- Events (what happened recently)
- Conditions (why is something not ready?)

### Apply — Deploying Resources

```bash
# Apply a single file
kubectl apply -f pod.yaml

# Apply all files in a directory
kubectl apply -f ./manifests/

# Apply from a URL
kubectl apply -f https://raw.githubusercontent.com/example/repo/main/deploy.yaml

# Dry run (see what would happen without actually doing it)
kubectl apply -f deploy.yaml --dry-run=client

# Server-side dry run (validates against cluster)
kubectl apply -f deploy.yaml --dry-run=server

# Show diff between what's running and what you're about to apply
kubectl diff -f deploy.yaml
```

### Delete — Removing Resources

```bash
# Delete a pod
kubectl delete pod my-pod

# Delete using a file
kubectl delete -f pod.yaml

# Delete all pods in namespace
kubectl delete pods --all -n my-namespace

# Force delete (skip graceful shutdown)
kubectl delete pod my-pod --force --grace-period=0

# Delete a namespace (deletes everything inside it)
kubectl delete namespace my-namespace
```

### Logs — Viewing Container Output

```bash
# View logs from a pod
kubectl logs my-pod

# Follow logs in real time
kubectl logs -f my-pod

# View last N lines
kubectl logs my-pod --tail=100

# View logs from previous container instance (after crash)
kubectl logs my-pod --previous

# View logs from a specific container in a multi-container pod
kubectl logs my-pod -c my-container

# View logs from all containers in a pod
kubectl logs my-pod --all-containers

# View logs from pods matching a label
kubectl logs -l app=my-app

# View logs from the last 1 hour
kubectl logs my-pod --since=1h

# View logs since a specific time
kubectl logs my-pod --since-time="2024-01-15T14:00:00Z"
```

### Exec — Running Commands in Containers

```bash
# Open an interactive shell in a pod
kubectl exec -it my-pod -- /bin/bash
kubectl exec -it my-pod -- /bin/sh   # if bash not available

# Run a single command
kubectl exec my-pod -- ls /app
kubectl exec my-pod -- env
kubectl exec my-pod -- cat /etc/config.yaml

# Run in a specific container (multi-container pods)
kubectl exec -it my-pod -c my-container -- /bin/bash

# Copy files from pod to local
kubectl cp my-pod:/app/logs/error.log ./error.log

# Copy files from local to pod
kubectl cp ./config.yaml my-pod:/app/config.yaml
```

### Port Forwarding — Accessing Pods Locally

```bash
# Forward local port 8080 to pod port 80
kubectl port-forward pod/my-pod 8080:80

# Forward to a service
kubectl port-forward service/my-service 8080:80

# Forward to a deployment
kubectl port-forward deployment/my-deploy 8080:8080

# Now access in browser: http://localhost:8080
```

### Imperative Commands (Quick Creation)

```bash
# Create a pod immediately (no YAML needed)
kubectl run nginx --image=nginx

# Create a pod with a specific port
kubectl run nginx --image=nginx --port=80

# Create a deployment
kubectl create deployment nginx --image=nginx

# Create a deployment with replicas
kubectl create deployment nginx --image=nginx --replicas=3

# Expose a deployment as a service
kubectl expose deployment nginx --port=80 --type=NodePort

# Create a ConfigMap
kubectl create configmap my-config --from-literal=key1=value1

# Create a Secret
kubectl create secret generic my-secret --from-literal=password=mysecret

# Create a namespace
kubectl create namespace dev
```

### Labels and Filtering

```bash
# Show labels on pods
kubectl get pods --show-labels

# Filter by label
kubectl get pods -l app=my-app
kubectl get pods -l env=production,app=nginx

# Add a label
kubectl label pod my-pod environment=production

# Remove a label
kubectl label pod my-pod environment-

# Label a node
kubectl label node worker-1 disktype=ssd
```

### Resource Usage

```bash
# Show CPU and memory usage for nodes
kubectl top nodes

# Show CPU and memory usage for pods
kubectl top pods

# Show top pods in a namespace
kubectl top pods -n my-namespace

# Sort by CPU
kubectl top pods --sort-by=cpu

# Sort by memory
kubectl top pods --sort-by=memory
```

### Useful Aliases (Add to ~/.bashrc)

```bash
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kdel='kubectl delete'
alias kl='kubectl logs'
alias kex='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kns='kubectl config set-context --current --namespace'

# Example usage:
k get pods
kg pods -A
kl my-pod -f
kex my-pod -- bash
```

### YAML Examples

```yaml
# Generate YAML without applying (great for learning YAML syntax)
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Output:
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
```

### Common Mistakes

**Mistake 1: Using `kubectl delete` and `kubectl apply` instead of just `kubectl apply`**
`kubectl apply` is idempotent — run it again and again, it only changes what's different. Always use `apply`.

**Mistake 2: Not using namespaces**
Everything goes into `default` namespace unless you specify. Use namespaces from day one.

**Mistake 3: Not using `--dry-run` before making changes**
Before any `kubectl apply` on production, use `--dry-run=server` to validate.

### Hands-On Tasks

**Task 4.1:** Run `kubectl run test-pod --image=nginx` then find the pod's IP address using `kubectl get pod test-pod -o wide`.

**Task 4.2:** Shell into the pod with `kubectl exec -it test-pod -- bash`. Run `curl localhost` inside. Exit.

**Task 4.3:** View the pod's logs with `kubectl logs test-pod`. Delete the pod with `kubectl delete pod test-pod`.

---

## Chapter 5 — Pods

### What You Will Learn
What a Pod is, how it works internally, how to create and manage Pods, how containers in a Pod share resources, and why Pods are the fundamental unit of Kubernetes.

### Why This Exists

Everything in Kubernetes runs inside Pods. Deployments manage Pods. Services route traffic to Pods. All the advanced features ultimately create and manage Pods. You cannot skip this chapter.

### How It Works Internally

A Pod is a wrapper around one or more containers. The containers in a Pod:

- Share the same **network namespace** — they all have the same IP address and can communicate via localhost
- Share the same **IPC namespace** — they can use shared memory and signals
- Can share **volumes** — mounted into all containers in the Pod
- Are always co-located on the same Node — they live and die together

```
┌──────────────────────────────────────────────────────┐
│                       POD                             │
│   IP: 10.244.1.15                                    │
│                                                       │
│  ┌─────────────────┐    ┌─────────────────┐          │
│  │  Container 1    │    │  Container 2    │          │
│  │  (main app)     │    │  (sidecar)      │          │
│  │                 │    │                 │          │
│  │  Port 8080      │    │  Port 9090      │          │
│  └────────┬────────┘    └────────┬────────┘          │
│           │                     │                    │
│           └──────┬──────────────┘                    │
│                  │                                    │
│           Shared Network                             │
│           Shared Volumes                             │
└──────────────────────────────────────────────────────┘
```

### Architecture Explanation

#### The Pause Container (Infrastructure Container)

When a Pod starts, Kubernetes first creates a "pause" container (also called the infra container or sandbox container). This container holds the network namespace. All other containers in the Pod join this network namespace. This means:

- If a container crashes and restarts, the network namespace is preserved (no IP change)
- Containers can communicate via localhost without any special configuration

```
Pod lifecycle:
1. Pause container starts → creates network namespace → gets Pod IP
2. Init containers run (if any) → in sequence → must succeed
3. Main containers start → all share pause container's network
```

### YAML Examples

#### Simple Single-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  namespace: default
  labels:
    app: nginx
    environment: development
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
      name: http
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

#### Multi-Container Pod (Sidecar Pattern)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
  labels:
    app: my-app
spec:
  containers:
  # Main application container
  - name: main-app
    image: my-app:1.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app

  # Sidecar: Log shipping container
  - name: log-shipper
    image: fluent/fluentd:v1.16
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app
    env:
    - name: FLUENTD_CONF
      value: "fluentd.conf"

  volumes:
  - name: shared-logs
    emptyDir: {}
```

#### Pod with Init Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  # Init containers run before main containers
  # They must complete successfully
  initContainers:
  - name: wait-for-db
    image: busybox:1.35
    command: ['sh', '-c',
      'until nc -z postgres-service 5432; do echo "waiting for db"; sleep 2; done']

  - name: run-migrations
    image: my-app:1.0
    command: ['python', 'manage.py', 'migrate']
    env:
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: url

  containers:
  - name: my-app
    image: my-app:1.0
    ports:
    - containerPort: 8000
```

#### Pod with Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: env-demo
    image: nginx
    env:
    # Literal value
    - name: APP_ENV
      value: "production"

    # From ConfigMap
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: db-host

    # From Secret
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: db-password

    # From Pod metadata (Downward API)
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
```

#### Pod with Restart Policy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restart-demo
spec:
  # restartPolicy options:
  # Always   - always restart (default, good for long-running apps)
  # OnFailure - restart only if exits with non-zero (good for jobs)
  # Never    - never restart (good for one-time tasks)
  restartPolicy: Always

  containers:
  - name: my-app
    image: my-app:1.0
```

### Pod Lifecycle

```
Pending → Running → Succeeded/Failed

Pending:   Pod accepted, but containers not yet created
           (could be waiting for scheduling, image pull, etc.)

Running:   Pod bound to a node, containers created,
           at least one container is running

Succeeded: All containers have terminated successfully (exit 0)

Failed:    All containers have terminated,
           at least one exited with non-zero

Unknown:   State cannot be determined (usually node communication issue)
```

### Container States

```bash
# Each container within a pod has its own state:
# Waiting  - not yet running (pulling image, waiting for init containers)
# Running  - executing
# Terminated - finished (either completed or failed)

kubectl describe pod my-pod
# Look for:
# State: Running
# State: Waiting (Reason: ContainerCreating)
# State: Terminated (Reason: Error, Exit Code: 1)
```

### Commands

```bash
# Create a pod from YAML
kubectl apply -f pod.yaml

# Create a pod quickly
kubectl run my-nginx --image=nginx

# Get pod details
kubectl get pod my-nginx -o wide

# See pod YAML as stored in cluster
kubectl get pod my-nginx -o yaml

# Describe pod (events, conditions, container state)
kubectl describe pod my-nginx

# View pod logs
kubectl logs my-nginx

# Shell into pod
kubectl exec -it my-nginx -- bash

# Delete pod
kubectl delete pod my-nginx

# Delete pod immediately (no graceful shutdown)
kubectl delete pod my-nginx --force --grace-period=0

# Get pod IP
kubectl get pod my-nginx -o jsonpath='{.status.podIP}'

# Watch pod status changes
kubectl get pod my-nginx -w
```

### Pod Design Patterns

#### Pattern 1: Sidecar
A helper container that enhances the main container without being the main app. Examples: log shippers, monitoring agents, config reloaders.

#### Pattern 2: Ambassador
A proxy container that handles network connections on behalf of the main container. Example: Envoy proxy handling service mesh traffic.

#### Pattern 3: Adapter
A container that transforms the main container's output into a standard format. Example: converting proprietary metrics format to Prometheus format.

#### Pattern 4: Init Container
Runs before the main containers. Used for setup tasks — database migrations, waiting for dependencies, writing config files.

### Real World Example

A production Node.js API Pod might have:
- **Main container:** Node.js app on port 3000
- **Sidecar 1:** Fluentd log shipper reading application logs
- **Sidecar 2:** Envoy proxy (service mesh) handling mTLS and traffic management
- **Init container:** Waits for the PostgreSQL database to be ready before starting

### Common Mistakes

**Mistake 1: Running one Pod with no Deployment**
Bare Pods are not recreated if they die or the node fails. Always use Deployments for production workloads.

**Mistake 2: Putting all app services in one Pod**
Unless containers are genuinely tightly coupled, put each service in its own Pod so they can scale independently.

**Mistake 3: Not setting resource limits**
Without resource limits, one misbehaving container can consume all node resources and kill other Pods.

**Mistake 4: Using `latest` image tag**
`latest` is unpredictable in production. Always pin to a specific version: `nginx:1.25.3`.

### Best Practices

- Always set resource requests and limits
- Always use specific image tags, never `latest`
- Use init containers for dependency waiting instead of retry loops in your app
- Use multi-container Pods sparingly — only when containers are tightly coupled
- Add meaningful labels to every Pod
- Set `restartPolicy` appropriately

### Hands-On Tasks

**Task 5.1:** Create a Pod running nginx. Get its cluster IP. Port-forward to it and access it in your browser.

**Task 5.2:** Create a multi-container Pod with nginx and a busybox sidecar. In the busybox container, run `wget localhost` to confirm the two containers share the same network.

**Task 5.3:** Create a Pod with an init container that sleeps for 10 seconds before the main container starts. Watch it with `kubectl get pod -w` to see it go through stages.

**Task 5.4:** Deliberately create a Pod with a bad image name. Watch it fail. Use `kubectl describe pod` and `kubectl logs` to see the error.

### Debugging Section

```bash
# Pod is stuck in Pending?
kubectl describe pod <pod-name>
# Look at Events section at the bottom
# Common reasons:
# "Insufficient cpu" or "Insufficient memory" → node has no capacity
# "no nodes available to schedule pods" → no nodes match requirements
# "FailedScheduling" → scheduler could not place the pod

# Pod is in CrashLoopBackOff?
kubectl logs <pod-name>                  # current logs
kubectl logs <pod-name> --previous       # logs from before crash
kubectl describe pod <pod-name>          # check exit code and reason

# Pod stuck in ContainerCreating?
kubectl describe pod <pod-name>
# Could be: image pull failure, volume mount failure, init container still running

# Check image pull errors
kubectl describe pod <pod-name> | grep -A5 "Events"
# Look for: Failed to pull image, ImagePullBackOff
```

---

# LEVEL 2 — INTERMEDIATE

---

## Chapter 6 — Deployments

### What You Will Learn
How Deployments manage your application lifecycle, how rolling updates work, how to rollback, and why Deployments are the most important resource type you will use day to day.

### Why This Exists

Bare Pods have one massive problem: if a Pod dies, nothing recreates it. A Deployment wraps your Pods in a management layer that ensures your desired number of Pods always runs. It also gives you rolling updates, rollbacks, and scaling — all with zero downtime.

### How It Works Internally

```
Deployment → manages → ReplicaSet → manages → Pods

When you create a Deployment:
1. Deployment controller creates a ReplicaSet
2. ReplicaSet controller creates the specified number of Pods
3. If a Pod dies, ReplicaSet controller creates a replacement
4. When you update a Deployment, a NEW ReplicaSet is created
5. New RS scales up, old RS scales down (rolling update)
6. Old RS kept at 0 replicas (for rollback)
```

### Architecture Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT                                │
│  name: my-app    replicas: 3    strategy: RollingUpdate         │
│                                                                   │
│  ┌───────────────────────────────┐                              │
│  │       ReplicaSet v1           │  (current)                   │
│  │       replicas: 3             │                              │
│  │                               │                              │
│  │   ┌───────┐ ┌───────┐ ┌──────┐│                             │
│  │   │ Pod-1 │ │ Pod-2 │ │Pod-3 ││                             │
│  │   └───────┘ └───────┘ └──────┘│                             │
│  └───────────────────────────────┘                              │
│                                                                   │
│  ┌───────────────────────────────┐                              │
│  │       ReplicaSet v0           │  (old, kept for rollback)    │
│  │       replicas: 0             │                              │
│  └───────────────────────────────┘                              │
└─────────────────────────────────────────────────────────────────┘
```

### YAML Examples

#### Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: default
  labels:
    app: my-app
spec:
  # How many Pod copies to run
  replicas: 3

  # Which Pods this Deployment manages
  selector:
    matchLabels:
      app: my-app

  # Update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max extra pods during update
      maxUnavailable: 0  # Zero downtime (never remove a pod before new one ready)

  # Pod template - defines what each Pod looks like
  template:
    metadata:
      labels:
        app: my-app       # Must match selector above
        version: "1.0"
    spec:
      containers:
      - name: my-app
        image: my-app:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

#### Production Deployment with All Options

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-api
  namespace: production
  labels:
    app: web-api
    team: backend
    tier: api
  annotations:
    deployment.kubernetes.io/revision: "1"
    description: "Web API service for customer-facing endpoints"
spec:
  replicas: 5
  revisionHistoryLimit: 10  # Keep last 10 ReplicaSets for rollback

  selector:
    matchLabels:
      app: web-api

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Allow 2 extra pods during rollout
      maxUnavailable: 0  # Never have fewer than 5 pods

  minReadySeconds: 10    # Wait 10s after pod ready before considering update successful

  template:
    metadata:
      labels:
        app: web-api
        version: "2.1.0"
    spec:
      # Spread pods across availability zones
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: web-api

      # Don't run two pods of same app on same node
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-api
            topologyKey: kubernetes.io/hostname

      containers:
      - name: web-api
        image: registry.company.com/web-api:2.1.0
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 8080
        - name: metrics
          containerPort: 9090

        env:
        - name: ENVIRONMENT
          value: "production"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password

        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "1000m"

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 5
          failureThreshold: 3

        livenessProbe:
          httpGet:
            path: /live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3

        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]

      terminationGracePeriodSeconds: 60

      imagePullSecrets:
      - name: registry-credentials
```

### Commands

```bash
# Create deployment
kubectl apply -f deployment.yaml

# Create deployment quickly
kubectl create deployment nginx --image=nginx --replicas=3

# List deployments
kubectl get deployments
kubectl get deploy

# See rollout status (live update progress)
kubectl rollout status deployment/my-app

# Detailed deployment info
kubectl describe deployment my-app

# Scale deployment
kubectl scale deployment my-app --replicas=5

# Update image (triggers rolling update)
kubectl set image deployment/my-app my-app=my-app:2.0

# View rollout history
kubectl rollout history deployment/my-app

# View specific revision
kubectl rollout history deployment/my-app --revision=2

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to specific version
kubectl rollout undo deployment/my-app --to-revision=2

# Pause rollout (stop mid-update)
kubectl rollout pause deployment/my-app

# Resume rollout
kubectl rollout resume deployment/my-app

# Restart all pods (rolling restart)
kubectl rollout restart deployment/my-app
```

### Rolling Update Deep Dive

```
Start: 3 pods running v1 (app:v1 app:v1 app:v1)

maxSurge: 1, maxUnavailable: 0

Step 1: Start 1 new pod v2    (app:v1 app:v1 app:v1 app:v2)  ← 4 pods
Step 2: v2 becomes ready
Step 3: Remove 1 old pod v1   (app:v1 app:v1 app:v2)          ← 3 pods
Step 4: Start 1 new pod v2    (app:v1 app:v1 app:v2 app:v2)  ← 4 pods
Step 5: v2 becomes ready
Step 6: Remove 1 old pod v1   (app:v1 app:v2 app:v2)          ← 3 pods
Step 7: Start 1 new pod v2    (app:v1 app:v2 app:v2 app:v2)  ← 4 pods
Step 8: v2 becomes ready
Step 9: Remove 1 old pod v1   (app:v2 app:v2 app:v2)          ← 3 pods
Done! Zero downtime.
```

### Rollback Strategy

When something goes wrong in production:

```bash
# Something is wrong! Check what happened
kubectl rollout status deployment/my-app
kubectl get pods   # Are pods crashing?
kubectl logs deployment/my-app --tail=50

# Instant rollback to previous version
kubectl rollout undo deployment/my-app

# Verify rollback
kubectl rollout status deployment/my-app
kubectl get pods

# Check which image is now running
kubectl get deployment my-app -o jsonpath='{.spec.template.spec.containers[0].image}'
```

### Common Mistakes

**Mistake 1: No readiness probe**
Without a readiness probe, Kubernetes sends traffic to new pods before they're ready. Always add a readiness probe.

**Mistake 2: `maxUnavailable: 1` in production**
This means during a rolling update, you could have one fewer pod serving traffic. In production with tight SLAs, use `maxUnavailable: 0`.

**Mistake 3: Not setting `revisionHistoryLimit`**
Default is 10. Old ReplicaSets accumulate. Set a reasonable limit.

**Mistake 4: Not testing rollbacks**
Test your rollback process in staging before you need it in production at 3 AM.

### Best Practices

- Always use Deployments instead of bare Pods
- Always add readiness and liveness probes
- Set `maxUnavailable: 0` for zero-downtime deployments
- Add `preStop` lifecycle hook with a sleep for graceful shutdown
- Use `revisionHistoryLimit: 10` to keep rollback options
- Always add resource requests and limits

### Hands-On Tasks

**Task 6.1:** Create a Deployment with 3 replicas of nginx. Scale it to 5 replicas. Watch the new pods come up.

**Task 6.2:** Update the image to `nginx:1.25` and watch the rolling update with `kubectl rollout status deployment/nginx -w`.

**Task 6.3:** Update to a non-existent image like `nginx:broken-image`. Watch it fail. Then rollback with `kubectl rollout undo`.

---

## Chapter 7 — ReplicaSets

### What You Will Learn
What ReplicaSets are, how they maintain Pod count, and why you almost never create them directly.

### Why This Exists

A ReplicaSet is the mechanism that actually maintains the desired number of Pod replicas. Understanding it helps you understand Deployments and troubleshoot pod count issues.

### How It Works Internally

The ReplicaSet controller constantly compares desired replicas vs actual running pods. If a Pod dies, the controller creates a new one. If there are too many, it deletes extras.

```
Control Loop:
OBSERVE: Current running Pods = 2 (one crashed)
COMPARE: Desired Pods = 3
ACT:     Create 1 new Pod
OBSERVE: Current running Pods = 3
COMPARE: Desired Pods = 3
ACT:     Nothing (desired == actual)
```

### YAML Examples

```yaml
# You rarely create ReplicaSets directly — Deployments create them.
# But here is the YAML structure:

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-replicaset
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Commands

```bash
# List ReplicaSets
kubectl get replicasets
kubectl get rs

# See ReplicaSet details
kubectl describe rs my-app-replicaset

# See which RS belongs to which Deployment
kubectl get rs -o wide

# Get the owner reference of a ReplicaSet
kubectl get rs my-app-rs -o jsonpath='{.metadata.ownerReferences}'
```

### Key Insight: Ownership Chain

```
Deployment owns → ReplicaSet owns → Pods

If you delete a ReplicaSet that belongs to a Deployment, the 
Deployment controller will immediately create a new one!

If you scale a ReplicaSet directly (not through the Deployment),
the Deployment controller will override your change.

Always manage Pods through Deployments, never through ReplicaSets directly.
```

---

## Chapter 8 — Services

### What You Will Learn
How Pods communicate with each other and the outside world, the four types of Services, how DNS works in Kubernetes, and how to expose applications properly.

### Why This Exists

Pods are ephemeral. They crash and restart with new IP addresses. If Service A needs to talk to Service B, it can't hardcode an IP that changes. Services provide a **stable** network endpoint backed by a group of Pods. The Service IP never changes, even as Pods come and go.

### How It Works Internally

```
Client → Service (stable IP: 10.96.45.123) → Pod A, Pod B, or Pod C

When a Pod calls the Service IP:
1. kube-proxy intercepts the request (via iptables/IPVS rules)
2. Selects one of the healthy backend Pods (load balancing)
3. Forwards request to that Pod's IP:Port
4. Pod responds
5. Response flows back through kube-proxy

The Endpoints object tracks which Pods are currently backing the Service:
kubectl get endpoints my-service
```

### Architecture Explanation

```
External User
     │
     ▼
┌─────────────────┐
│  LoadBalancer   │  ← Cloud provider LB (AWS ELB, GCP LB)
│  203.0.113.10   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   NodePort      │  ← All nodes expose port 30080
│   :30080        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   ClusterIP     │  ← Internal cluster IP
│   10.96.45.123  │  ← Only accessible inside cluster
└────────┬────────┘
         │  (load balances)
    ─────┼─────
    │    │    │
    ▼    ▼    ▼
  Pod1  Pod2  Pod3
```

### The Four Service Types

| Type | Accessible From | Use Case |
|---|---|---|
| ClusterIP | Inside cluster only | Service-to-service communication |
| NodePort | External via node IP | Development, simple external access |
| LoadBalancer | External via cloud LB | Production external access |
| ExternalName | Inside cluster | Map to external DNS name |

### YAML Examples

#### ClusterIP Service (Default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: default
spec:
  type: ClusterIP  # Default type
  selector:
    app: backend   # Routes to Pods with this label
  ports:
  - name: http
    protocol: TCP
    port: 80        # Port the Service listens on
    targetPort: 8080 # Port the Pod is actually listening on
```

#### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
    nodePort: 30080  # External port (30000-32767 range)
    # Access: http://<any-node-ip>:30080
```

#### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: public-api
  annotations:
    # AWS-specific: use internal load balancer
    service.beta.kubernetes.io/aws-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
```

#### Headless Service (for StatefulSets)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None  # Makes it headless - no single Service IP
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
# Now DNS: postgres-0.postgres-headless.default.svc.cluster.local
# Each pod gets its own DNS entry
```

### DNS in Kubernetes

Every Service automatically gets a DNS name:

```
Pattern: <service-name>.<namespace>.svc.cluster.local

Examples:
backend-service.default.svc.cluster.local
database.production.svc.cluster.local
redis.cache.svc.cluster.local

Short form (within same namespace):
backend-service
database
redis

Cross-namespace:
backend-service.production.svc.cluster.local
```

From inside a Pod:

```bash
# Test DNS resolution
kubectl exec -it my-pod -- nslookup backend-service
kubectl exec -it my-pod -- curl http://backend-service/api/health
```

### Commands

```bash
# Create service
kubectl apply -f service.yaml

# List services
kubectl get services
kubectl get svc

# Get service details
kubectl describe service my-service

# See which Pods the service routes to
kubectl get endpoints my-service

# Create a ClusterIP service quickly
kubectl expose deployment my-app --port=80 --target-port=8080

# Create a NodePort service
kubectl expose deployment my-app --type=NodePort --port=80

# Create a LoadBalancer service
kubectl expose deployment my-app --type=LoadBalancer --port=80

# Port forward to a service for local testing
kubectl port-forward service/my-service 8080:80
```

### Real World Example

An e-commerce app has these services:
- `frontend` → LoadBalancer (user-facing, gets public IP)
- `api-gateway` → ClusterIP (receives traffic from frontend internally)
- `product-service` → ClusterIP (only called by api-gateway)
- `order-service` → ClusterIP (only called by api-gateway)
- `postgres` → ClusterIP with headless for direct pod access
- `redis` → ClusterIP (cache, internal only)

### Common Mistakes

**Mistake 1: Label selector mismatch**
Service selectors must exactly match Pod labels. If your Service has `app: myapp` but your Pod has `app: my-app`, the service routes to no pods (empty endpoints).

```bash
# Diagnose this
kubectl get endpoints my-service  
# If no endpoints (or empty list), check label mismatch
kubectl get pods --show-labels
```

**Mistake 2: Using NodePort in production**
NodePort exposes a port on every node. Use LoadBalancer or Ingress for production external access.

**Mistake 3: Forgetting `targetPort`**
`port` is what the Service listens on. `targetPort` is what the container listens on. They can be different.

### Best Practices

- Always use ClusterIP for internal services — never expose directly unless needed
- Use descriptive service names that match your app names
- Use named ports (`name: http`) for clarity
- Use Ingress instead of LoadBalancer for HTTP/HTTPS traffic (one LB for all services)

---

## Chapter 9 — Labels & Selectors

### What You Will Learn
How labels organize Kubernetes resources, how selectors filter resources, and how labels power Services, Deployments, and almost everything in Kubernetes.

### Why This Exists

Labels are key-value pairs attached to every Kubernetes resource. They are how Kubernetes objects find and relate to each other. Services find Pods via labels. Deployments manage ReplicaSets via labels. NetworkPolicies apply to Pods via labels. Without labels, Kubernetes cannot function.

### How It Works Internally

```
Label: key=value pair on a resource
         "app=nginx"
         "environment=production"
         "tier=frontend"
         "version=1.2.3"

Selector: a filter that matches resources with specific labels
         matchLabels: {app: nginx}         # exact match
         matchExpressions:
           key: environment
           operator: In
           values: [staging, production]   # match either value
```

### YAML Examples

```yaml
# Labeling resources
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    # Recommended label conventions:
    app.kubernetes.io/name: my-app
    app.kubernetes.io/version: "1.2.3"
    app.kubernetes.io/component: backend
    app.kubernetes.io/part-of: my-platform
    app.kubernetes.io/managed-by: helm
    # Custom labels:
    environment: production
    team: backend
    tier: api
    region: us-east-1
spec:
  containers:
  - name: my-app
    image: my-app:1.2.3
---
# Service using selector to find pods
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: my-app
    environment: production
  ports:
  - port: 80
    targetPort: 8080
```

### Commands

```bash
# Show all labels on pods
kubectl get pods --show-labels

# Filter pods by label
kubectl get pods -l app=my-app
kubectl get pods -l environment=production
kubectl get pods -l app=my-app,environment=production  # AND

# Advanced selectors
kubectl get pods -l 'environment in (production, staging)'
kubectl get pods -l 'environment notin (development)'
kubectl get pods -l '!debug'  # doesn't have 'debug' label

# Add a label
kubectl label pod my-pod color=blue

# Update a label
kubectl label pod my-pod color=green --overwrite

# Remove a label
kubectl label pod my-pod color-

# Label a node
kubectl label node worker-1 disktype=ssd gpu=true

# Filter nodes by label
kubectl get nodes -l disktype=ssd
```

### Common Label Conventions

```yaml
# Kubernetes recommended labels (use these in production):
labels:
  app.kubernetes.io/name: mysql        # Name of the application
  app.kubernetes.io/instance: prod-db  # Unique name of instance
  app.kubernetes.io/version: "8.0"     # App version
  app.kubernetes.io/component: database # Component within the architecture
  app.kubernetes.io/part-of: my-app    # Higher level app this belongs to
  app.kubernetes.io/managed-by: helm   # Tool used to manage the app
```

---

## Chapter 10 — ConfigMaps

### What You Will Learn
How to separate application configuration from application code using ConfigMaps, and how to inject configuration into Pods as environment variables or files.

### Why This Exists

Hardcoding configuration in your container image means rebuilding the image every time configuration changes. ConfigMaps let you store configuration outside the container and inject it at runtime, making your containers portable across environments.

### How It Works Internally

ConfigMap is a Kubernetes object that stores non-sensitive key-value data. The data is stored in etcd. At Pod creation time, the kubelet reads the ConfigMap and either:
- Sets it as environment variables in the container
- Mounts it as files in the container's filesystem

When a ConfigMap mounted as a volume is updated, Kubernetes automatically updates the files in running pods (within ~1 minute). Environment variables do NOT update automatically — a pod restart is required.

### YAML Examples

#### Creating ConfigMaps

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  # Simple key-value pairs
  APP_PORT: "8080"
  APP_ENV: "production"
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"

  # Multi-line data (like config files)
  nginx.conf: |
    server {
      listen 80;
      server_name example.com;
      location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
      }
    }

  database.yml: |
    production:
      adapter: postgresql
      host: postgres-service
      port: 5432
      database: myapp_production
```

#### Using ConfigMap as Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0

    # Method 1: Individual keys from ConfigMap
    env:
    - name: APP_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_PORT
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL

    # Method 2: All keys from ConfigMap at once
    envFrom:
    - configMapRef:
        name: app-config
    # This injects ALL keys from app-config as environment variables
```

#### Using ConfigMap as Volume (Files)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d  # Mount point inside container
      readOnly: true

  volumes:
  - name: config-volume
    configMap:
      name: app-config
      # Optionally select specific keys
      items:
      - key: nginx.conf
        path: default.conf  # File name inside the mount path
```

### Commands

```bash
# Create ConfigMap from literal values
kubectl create configmap my-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info

# Create ConfigMap from a file
kubectl create configmap nginx-config --from-file=nginx.conf

# Create ConfigMap from a directory of files
kubectl create configmap app-configs --from-file=./configs/

# List ConfigMaps
kubectl get configmaps
kubectl get cm

# View ConfigMap data
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Edit a ConfigMap
kubectl edit configmap app-config

# Delete a ConfigMap
kubectl delete configmap app-config
```

### Common Mistakes

**Mistake 1: Storing secrets in ConfigMaps**
ConfigMaps are not encrypted. Never put passwords, API keys, or other secrets in ConfigMaps. Use Secrets for sensitive data.

**Mistake 2: Expecting environment variables to update automatically**
Environment variables from ConfigMaps are set at pod start time. If you update a ConfigMap, pods using `env`/`envFrom` will not see changes until they restart.

**Mistake 3: Very large ConfigMaps**
ConfigMaps are stored in etcd with a 1MB size limit. For large files, consider other storage mechanisms.

### Best Practices

- Use ConfigMaps for non-sensitive configuration only
- Group related config into one ConfigMap rather than many small ones
- Use volume mounts for configuration files, environment variables for simple key-value pairs
- Consider using Kustomize or Helm to manage ConfigMaps across environments

---

## Chapter 11 — Secrets

### What You Will Learn
How Kubernetes stores sensitive information like passwords, API keys, and TLS certificates, and how to use them safely in your applications.

### Why This Exists

Applications need sensitive data — database passwords, API tokens, TLS certificates, SSH keys. Hardcoding these in images or passing them as environment variables in plain text is a security risk. Secrets provide a dedicated place to store sensitive data with access controls.

### How It Works Internally

Secrets are stored in etcd. In a default installation, etcd stores them base64-encoded (NOT encrypted). In a hardened installation, etcd encrypts them at rest using AES encryption. Access to Secrets is controlled by RBAC.

```
Secret Data Flow:
1. You create a Secret (kubectl apply or API call)
2. Data stored in etcd (base64-encoded or encrypted)
3. kubelet fetches Secret from API Server when Pod needs it
4. kubelet delivers it to the container as:
   - Environment variable (value decoded from base64)
   - File in a volume (decoded content written to file)
5. Secret is stored in a tmpfs (RAM) on the node
   → disappears when pod stops
   → never written to disk on the node
```

### YAML Examples

#### Creating Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: default
type: Opaque  # Generic secret type
data:
  # Values must be base64 encoded!
  # echo -n "mypassword" | base64  →  bXlwYXNzd29yZA==
  username: YWRtaW4=          # "admin" in base64
  password: bXlwYXNzd29yZA==  # "mypassword" in base64
---
# Or use stringData - Kubernetes encodes it automatically
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  username: admin
  password: mypassword  # Written as plain text, stored as base64
```

#### TLS Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

#### Using Secrets in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0

    # Method 1: Single key as environment variable
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password

    # Method 2: All secret keys as environment variables
    envFrom:
    - secretRef:
        name: db-credentials

    # Method 3: Mount as file (more secure - not visible in env)
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true

  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
      defaultMode: 0400  # Read-only for owner only
```

### Commands

```bash
# Create secret from literal
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=mysecretpassword

# Create TLS secret from cert files
kubectl create secret tls my-tls \
  --cert=tls.crt \
  --key=tls.key

# Create Docker registry secret
kubectl create secret docker-registry registry-creds \
  --docker-server=registry.company.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=me@company.com

# List secrets
kubectl get secrets

# View secret (data will be base64 encoded)
kubectl describe secret db-secret

# Decode a secret value
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode

# Edit a secret
kubectl edit secret db-secret
```

### Security Hardening for Secrets

```yaml
# 1. Enable encryption at rest in kube-apiserver
# /etc/kubernetes/manifests/kube-apiserver.yaml:
# --encryption-provider-config=/etc/kubernetes/encryption-config.yaml

# encryption-config.yaml:
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-encoded-32-byte-key>
      - identity: {}  # Allow existing unencrypted secrets
```

### Common Mistakes

**Mistake 1: Committing Secrets to Git**
NEVER commit Secret YAML files to version control. Use sealed-secrets, Vault, or external-secrets-operator instead.

**Mistake 2: Assuming base64 is encryption**
base64 is encoding, not encryption. `echo bXlwYXNzd29yZA== | base64 --decode` reveals the secret immediately.

**Mistake 3: Using environment variables for very sensitive data**
Environment variables can be exposed via `kubectl exec`, `/proc/<pid>/environ`, and logging. Mount secrets as files for better security.

### Best Practices

- Enable etcd encryption at rest
- Use external secret management (HashiCorp Vault, AWS Secrets Manager)
- Use RBAC to restrict who can read Secrets
- Never log environment variables in your application
- Rotate secrets regularly
- Mount as files rather than environment variables for sensitive credentials

---

## Chapter 12 — Namespaces

### What You Will Learn
How to use Namespaces to organize and isolate workloads within a cluster, how to set resource quotas per namespace, and how to manage multi-team cluster access.

### Why This Exists

A Kubernetes cluster is shared infrastructure. Without namespaces, every team's resources would exist in the same space, creating naming conflicts and making it hard to control access. Namespaces create virtual partitions within a cluster.

### How It Works Internally

Namespaces partition cluster resources. Most Kubernetes resources (Pods, Deployments, Services, ConfigMaps, Secrets) are namespace-scoped. Some resources (Nodes, PersistentVolumes, StorageClasses) are cluster-scoped and exist outside any namespace.

```
Cluster
├── Namespace: default         ← Used when no namespace specified
├── Namespace: kube-system     ← Kubernetes system components
├── Namespace: kube-public     ← Public cluster info
├── Namespace: kube-node-lease ← Node heartbeats
├── Namespace: production      ← Your production apps
├── Namespace: staging         ← Your staging apps
├── Namespace: development     ← Developer workloads
└── Namespace: monitoring      ← Prometheus, Grafana, etc.
```

### YAML Examples

#### Creating Namespaces

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
    team: platform
---
apiVersion: v1
kind: Namespace
metadata:
  name: staging
  labels:
    environment: staging
```

#### Resource Quota (Limit what a namespace can use)

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    # Pod limits
    pods: "50"
    # CPU limits
    requests.cpu: "20"
    limits.cpu: "40"
    # Memory limits
    requests.memory: 40Gi
    limits.memory: 80Gi
    # Storage limits
    persistentvolumeclaims: "20"
    requests.storage: 500Gi
    # Object count limits
    services: "20"
    secrets: "50"
    configmaps: "50"
```

#### LimitRange (Default limits for Pods in a namespace)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - type: Container
    default:          # Default limits if not specified
      cpu: 500m
      memory: 256Mi
    defaultRequest:   # Default requests if not specified
      cpu: 100m
      memory: 128Mi
    max:              # Maximum allowed
      cpu: "2"
      memory: 2Gi
    min:              # Minimum required
      cpu: 50m
      memory: 64Mi
```

### Commands

```bash
# Create namespace
kubectl create namespace my-namespace
kubectl apply -f namespace.yaml

# List namespaces
kubectl get namespaces
kubectl get ns

# Set default namespace for current session
kubectl config set-context --current --namespace=production

# List all pods across ALL namespaces
kubectl get pods -A
kubectl get pods --all-namespaces

# List pods in specific namespace
kubectl get pods -n production

# Apply resource to specific namespace
kubectl apply -f deployment.yaml -n production

# Delete all resources in namespace
kubectl delete all --all -n development

# Delete a namespace (deletes EVERYTHING inside)
kubectl delete namespace development
```

### Namespace Best Practices

```
Recommended namespace structure for a production cluster:

cluster/
├── kube-system          (Kubernetes internal)
├── monitoring           (Prometheus, Grafana, Alertmanager)
├── logging              (ELK stack, Loki)
├── ingress-nginx        (Ingress controller)
├── cert-manager         (TLS certificate management)
├── production           (Production workloads)
├── staging              (Staging workloads)
└── development          (Developer experimentation)
```

### Common Mistakes

**Mistake 1: Putting everything in the default namespace**
This creates a mess in production clusters. Always use meaningful namespaces from day one.

**Mistake 2: Treating namespaces as security boundaries**
Namespaces are NOT hard security boundaries. Use RBAC, NetworkPolicies, and Pod Security Standards for real isolation.

**Mistake 3: Forgetting to specify namespace in kubectl commands**
Always be explicit with `-n namespace` or set the default context namespace.

---

# LEVEL 3 — ADVANCED

---

## Chapter 13 — Volumes

### What You Will Learn
How containers share data, how to persist data beyond container restarts, and the different types of volumes available in Kubernetes.

### Why This Exists

Container filesystems are ephemeral — when a container restarts, its filesystem is wiped. Volumes solve this by providing storage that outlives containers. They also allow multiple containers in a Pod to share data.

### How It Works Internally

A Volume is a directory accessible to containers in a Pod. Its lifetime is tied to the Pod (not individual containers). When a container restarts, the Volume's data persists within the same Pod. When the Pod itself is deleted, ephemeral volumes are also deleted.

```
Pod
├── Container A (writes to /data)
├── Container B (reads from /data)
└── Volume "shared-data" mounted at /data in both containers
    → Both containers see same filesystem
    → Data persists across container restarts
    → Data deleted when Pod is deleted
```

### Volume Types

| Type | Lifetime | Use Case |
|---|---|---|
| emptyDir | Pod lifetime | Temporary shared storage between containers |
| hostPath | Node lifetime | Access host filesystem (dangerous!) |
| configMap | External | Mount ConfigMap as files |
| secret | External | Mount Secret as files |
| persistentVolumeClaim | External | Durable storage (databases) |
| nfs | External | Shared NFS filesystem |
| awsElasticBlockStore | External | AWS EBS volumes |

### YAML Examples

#### emptyDir — Temporary Shared Storage

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-data-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'while true; do date >> /data/timestamps.txt; sleep 5; done']
    volumeMounts:
    - name: shared-data
      mountPath: /data

  - name: reader
    image: busybox
    command: ['sh', '-c', 'while true; do cat /data/timestamps.txt; sleep 10; done']
    volumeMounts:
    - name: shared-data
      mountPath: /data  # Same mount path - same data

  volumes:
  - name: shared-data
    emptyDir: {}  # Created empty when Pod starts

  # For memory-backed temp storage (faster, uses RAM):
  # emptyDir:
  #   medium: Memory
  #   sizeLimit: 256Mi
```

#### hostPath — Mount Node Filesystem

```yaml
# WARNING: Use with extreme caution in production
# hostPath gives Pod access to host node files
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: my-app
    volumeMounts:
    - name: docker-socket
      mountPath: /var/run/docker.sock

  volumes:
  - name: docker-socket
    hostPath:
      path: /var/run/docker.sock
      type: Socket  # Type: File, Directory, Socket, CharDevice, BlockDevice
```

### Common Mistakes

**Mistake 1: Using hostPath for application data**
hostPath is tied to a specific node. If the Pod moves to another node, the data is gone. Use PersistentVolumeClaims instead.

**Mistake 2: Not understanding emptyDir lifetime**
emptyDir is deleted when the Pod dies — it's only temporary. For durable storage, use PVC.

---

## Chapter 14 — Persistent Volumes

### What You Will Learn
How to provision durable storage that survives Pod deletions, how PersistentVolumes and PersistentVolumeClaims work together, and how to use external storage for stateful applications.

### Why This Exists

Databases and stateful applications need storage that persists when Pods restart or are rescheduled on different nodes. PersistentVolumes (PV) and PersistentVolumeClaims (PVC) decouple storage provisioning from Pod creation.

### How It Works Internally

```
Admin/Cloud creates PersistentVolume (actual disk)
    ↓
Developer creates PersistentVolumeClaim (request for storage)
    ↓
Kubernetes BINDS the PVC to a matching PV
    ↓
Pod references the PVC
    ↓
Container gets the disk mounted at specified path

Access Modes:
ReadWriteOnce (RWO)    - One node can read/write (most disks)
ReadOnlyMany (ROX)     - Multiple nodes can read
ReadWriteMany (RWX)    - Multiple nodes can read/write (NFS, etc.)

Reclaim Policy (what happens when PVC is deleted):
Retain  - PV kept, data preserved, admin must clean up manually
Delete  - PV and backing storage deleted automatically
Recycle - Data wiped, PV made available again (deprecated)
```

### YAML Examples

#### PersistentVolume (Admin creates this)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
  labels:
    type: local
    app: postgres
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  # Where the actual storage is:
  hostPath:
    path: /data/postgres  # For local dev/testing only
  # For AWS EBS:
  # awsElasticBlockStore:
  #   volumeID: vol-0123456789abcdef0
  #   fsType: ext4
```

#### PersistentVolumeClaim (Developer creates this)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: production
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  # Optional: select a specific PV
  selector:
    matchLabels:
      app: postgres
```

#### Pod Using PVC

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
      valueFrom:
        secretKeyRef:
          name: postgres-secret
          key: password
    volumeMounts:
    - name: postgres-storage
      mountPath: /var/lib/postgresql/data
      subPath: postgres  # Use subdirectory to avoid mount issues

  volumes:
  - name: postgres-storage
    persistentVolumeClaim:
      claimName: postgres-pvc  # References the PVC
```

### Commands

```bash
# List PersistentVolumes
kubectl get pv

# List PersistentVolumeClaims
kubectl get pvc
kubectl get pvc -n my-namespace

# Describe PVC (check if it's Bound or Pending)
kubectl describe pvc my-pvc

# Check PV-PVC binding
kubectl get pv -o custom-columns=\
  NAME:.metadata.name,\
  STATUS:.status.phase,\
  CLAIM:.spec.claimRef.name
```

---

## Chapter 15 — Storage Classes

### What You Will Learn
How StorageClasses enable dynamic provisioning of storage, eliminating manual PV creation.

### Why This Exists

Without StorageClasses, admins must manually create PersistentVolumes before developers can claim storage. StorageClasses allow PVs to be created automatically (dynamically) when a PVC is created.

### YAML Examples

```yaml
# StorageClass for AWS GP3 EBS
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-gp3
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"  # Default StorageClass
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer  # Don't provision until Pod needs it
reclaimPolicy: Delete
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
---
# PVC using StorageClass (PV created automatically)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: auto-pvc
spec:
  storageClassName: aws-gp3  # Use this StorageClass
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
# When this PVC is applied:
# 1. StorageClass "aws-gp3" is triggered
# 2. EBS volume is automatically created in AWS
# 3. PV is created and bound to this PVC
# 4. No manual intervention needed!
```

### Commands

```bash
# List StorageClasses
kubectl get storageclass
kubectl get sc

# See which is default
kubectl get sc | grep default

# Describe a StorageClass
kubectl describe sc aws-gp3
```

---

## Chapter 16 — Ingress

### What You Will Learn
How to route external HTTP/HTTPS traffic to internal Services, how to configure TLS termination, and how Ingress replaces the need for multiple LoadBalancer services.

### Why This Exists

Without Ingress, every Service that needs external access gets its own LoadBalancer — its own cloud load balancer with its own IP and cost. Ingress puts a single load balancer in front of all your services and routes based on hostname and path. One LB, many services.

### How It Works Internally

```
Internet
    │
    ▼
Load Balancer (single IP, e.g. 203.0.113.10)
    │
    ▼
Ingress Controller (nginx, traefik, etc.)
    │  Reads Ingress rules
    │
    ├── api.example.com  →  api-service:80
    ├── app.example.com  →  frontend-service:80
    └── app.example.com/api → api-service:8080
```

### YAML Examples

#### Ingress Controller Installation (nginx)

```bash
# Install nginx ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

# Wait for it to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# Get the external IP
kubectl get service ingress-nginx-controller -n ingress-nginx
```

#### Basic Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx

  rules:
  # Route based on hostname
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80

  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      # Route /api path to different service on same host
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

#### Ingress with TLS (HTTPS)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx

  tls:
  - hosts:
    - api.example.com
    - app.example.com
    secretName: example-tls  # cert-manager will create this

  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### Common Ingress Annotations (nginx)

```yaml
annotations:
  # Rate limiting
  nginx.ingress.kubernetes.io/limit-rps: "10"

  # Request size
  nginx.ingress.kubernetes.io/proxy-body-size: "10m"

  # Timeouts
  nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
  nginx.ingress.kubernetes.io/proxy-read-timeout: "60"

  # CORS
  nginx.ingress.kubernetes.io/enable-cors: "true"
  nginx.ingress.kubernetes.io/cors-allow-origin: "https://example.com"

  # Basic auth
  nginx.ingress.kubernetes.io/auth-type: basic
  nginx.ingress.kubernetes.io/auth-secret: basic-auth
  nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"

  # Redirect
  nginx.ingress.kubernetes.io/permanent-redirect: "https://newsite.com"
```

---

## Chapter 17 — Networking

### What You Will Learn
How Kubernetes networking works under the hood, how Pods communicate across nodes, and the Container Network Interface (CNI) plugins.

### Why This Exists

Kubernetes networking seems complex because it solves a hard problem: making thousands of containers across hundreds of machines communicate as if they're all on the same flat network — without NAT, without manual IP management.

### Kubernetes Networking Model (The 4 Rules)

```
Rule 1: Every Pod gets its own unique IP address
Rule 2: All Pods can communicate with all other Pods without NAT
Rule 3: Agents on a node (kubelet, etc.) can communicate with all Pods on that node
Rule 4: Services have their own IP range (ClusterIP range), separate from Pod IPs
```

### How It Works Internally

```
Node A (10.0.1.1)                Node B (10.0.1.2)
Pod IP: 10.244.1.2               Pod IP: 10.244.2.2
Pod IP: 10.244.1.3               Pod IP: 10.244.2.3

Traffic from 10.244.1.2 to 10.244.2.3:
1. Pod on Node A sends packet to 10.244.2.3
2. CNI plugin (Flannel/Calico/Cilium) intercepts
3. Packet encapsulated (VXLAN) or routed to Node B
4. Node B's CNI receives packet
5. Packet delivered to Pod on Node B

No NAT! Source IP preserved.
```

### CNI Plugins Comparison

| CNI Plugin | Network Policy | Performance | Use Case |
|---|---|---|---|
| Flannel | No | Good | Simple setups, beginners |
| Calico | Yes | Very Good | Most production clusters |
| Cilium | Yes (eBPF) | Excellent | High-performance, observability |
| Weave | Yes | Good | Simple multi-cloud |
| Canal | Yes | Good | Flannel+Calico hybrid |

### Network Policies

```yaml
# Deny all ingress traffic to pods in production namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}  # Applies to all pods
  policyTypes:
  - Ingress
---
# Allow only specific traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: postgres  # This policy applies to postgres pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api  # Only allow traffic from api pods
    ports:
    - protocol: TCP
      port: 5432
```

---

## Chapter 18 — Resource Limits

### What You Will Learn
How to set CPU and memory requests and limits, how Kubernetes uses these for scheduling, and how to prevent one application from consuming all cluster resources.

### Why This Exists

Without resource limits, one misbehaving application (memory leak, infinite loop) can consume all CPU and memory on a node, crashing all other applications on that node. Resource requests and limits give every application its fair share.

### How It Works Internally

```
Requests: What the container is GUARANTEED to get
          Used by scheduler to find a node with enough capacity
          Node must have at least this much free

Limits:   The MAXIMUM the container can use
          Container killed (OOMKilled) if exceeds memory limit
          Throttled (not killed) if exceeds CPU limit

Example:
Node capacity:      16 CPU, 32Gi memory
Container requests: 2 CPU, 4Gi
Container limits:   4 CPU, 8Gi

Scheduler finds node with at least 2 CPU and 4Gi free
Container can burst to 4 CPU/8Gi if node has spare capacity
Container killed if it tries to use more than 8Gi memory
```

### CPU Units

```
1 CPU = 1 vCPU = 1 core = 1000m (millicores)

Examples:
100m  = 0.1 CPU = 10% of one core
500m  = 0.5 CPU = half a core
1000m = 1 CPU   = one full core
"2"   = 2 CPU   = two cores
```

### Memory Units

```
Memory is specified in bytes or with SI suffixes:
Ki = Kibibytes (1024 bytes)
Mi = Mebibytes (1024 Ki)
Gi = Gibibytes (1024 Mi)

Examples:
64Mi   = 64 Mebibytes
256Mi  = 256 Mebibytes
1Gi    = 1 Gibibyte
2.5Gi  = 2.5 Gibibytes
```

### YAML Examples

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: my-app:1.0
    resources:
      # REQUESTS: Guaranteed minimum
      requests:
        memory: "128Mi"   # 128 mebibytes guaranteed
        cpu: "250m"       # 0.25 CPU guaranteed

      # LIMITS: Hard maximum
      limits:
        memory: "512Mi"   # Never more than 512 mebibytes
        cpu: "1000m"      # Never more than 1 CPU
```

### Quality of Service Classes

Kubernetes assigns QoS classes based on resource settings:

```
Guaranteed (best):
  requests == limits for ALL containers in Pod
  Never evicted unless node is completely out of resources
  requests.cpu = limits.cpu AND requests.memory = limits.memory

Burstable (middle):
  At least one container has requests/limits set
  Not all equal
  Evicted after BestEffort when node is pressured

BestEffort (worst):
  No requests or limits set
  First to be evicted when node is under pressure
  Use only for batch/non-critical workloads
```

```bash
# Check QoS class of a pod
kubectl get pod my-pod -o jsonpath='{.status.qosClass}'
```

### Commands

```bash
# See current resource usage
kubectl top nodes
kubectl top pods
kubectl top pods -n production --sort-by=memory

# Describe a pod to see resource settings
kubectl describe pod my-pod | grep -A5 "Limits\|Requests"
```

### Common Mistakes

**Mistake 1: No resource requests or limits**
Your pod is BestEffort and will be killed first under node pressure.

**Mistake 2: Setting limits much higher than requests**
A pod that requests 100m CPU but has a 10 CPU limit can burst and starve other pods.

**Mistake 3: OOMKilled and not knowing why**
```bash
kubectl describe pod my-pod | grep -A5 "OOMKilled\|Last State"
# Increase memory limit or fix memory leak
```

---

## Chapter 19 — Health Checks

### What You Will Learn
How Kubernetes monitors your applications, how to configure liveness and readiness probes, and how probes enable zero-downtime deployments and automatic recovery.

### Why This Exists

Kubernetes needs to know: Is this container alive? Is it ready to receive traffic? Without probes, Kubernetes sends traffic to containers that aren't ready and can't detect crashed processes that are stuck.

### How It Works Internally

```
Liveness Probe:  "Is the container alive?"
  FAIL → Container is killed and restarted
  PASS → Container continues running

Readiness Probe: "Is the container ready to receive traffic?"
  FAIL → Pod removed from Service endpoints (no traffic sent)
  PASS → Pod added to Service endpoints (traffic sent)

Startup Probe:   "Has the container finished starting?"
  FAIL (until timeout) → Container killed
  PASS → Liveness/Readiness probes take over
  Use for slow-starting apps
```

### Probe Types

```
HTTP GET:   Makes HTTP request, success if 200-399
TCP Socket: Checks if port is open
Exec:       Runs command inside container, success if exit 0
gRPC:       gRPC health check protocol
```

### YAML Examples

#### Complete Health Check Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: web-app:1.0

        # Startup probe - for slow-starting applications
        # Checks every 10s, fails after 30 attempts = 5 min max startup
        startupProbe:
          httpGet:
            path: /health/startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 10

        # Readiness probe - controls if pod receives traffic
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 10  # Wait 10s before first check
          periodSeconds: 5         # Check every 5 seconds
          timeoutSeconds: 3        # Fail if no response in 3s
          successThreshold: 1      # 1 success = ready
          failureThreshold: 3      # 3 failures = not ready

        # Liveness probe - restart container if unhealthy
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30  # Wait 30s (let app start)
          periodSeconds: 10        # Check every 10 seconds
          timeoutSeconds: 5        # Fail if no response in 5s
          successThreshold: 1      # 1 success = healthy
          failureThreshold: 3      # 3 failures = restart container
```

#### TCP Socket Probe (for non-HTTP services)

```yaml
# For databases, Redis, etc.
readinessProbe:
  tcpSocket:
    port: 5432
  initialDelaySeconds: 15
  periodSeconds: 10
```

#### Exec Probe (Run a command)

```yaml
livenessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - "redis-cli ping | grep PONG"
  initialDelaySeconds: 10
  periodSeconds: 5
```

### What to Expose on Health Endpoints

```javascript
// Example Node.js health endpoints

// /health/ready - Are we ready to accept traffic?
app.get('/health/ready', async (req, res) => {
  try {
    // Check database connection
    await db.query('SELECT 1');
    // Check cache connection
    await redis.ping();
    res.status(200).json({ status: 'ready' });
  } catch (error) {
    res.status(503).json({ status: 'not ready', error: error.message });
  }
});

// /health/live - Are we alive (not deadlocked)?
app.get('/health/live', (req, res) => {
  // Simple check - if we can respond, we're alive
  res.status(200).json({ status: 'alive' });
});
```

### Common Mistakes

**Mistake 1: Same endpoint for liveness and readiness**
Liveness should be simpler (is the process alive?). Readiness should check dependencies. If DB goes down, readiness fails (stop traffic) but liveness stays up (don't restart).

**Mistake 2: Too aggressive liveness probe**
A liveness probe that's too quick or strict will restart your container during normal high load spikes. Use conservative settings.

**Mistake 3: No startup probe for slow apps**
JVM apps, apps with DB migrations, etc. need startup probes. Without them, liveness probe fires before app is ready and kills it.

---

# LEVEL 4 — PRODUCTION

---

## Chapter 20 — Rolling Updates

### What You Will Learn
How to update applications with zero downtime, how to control update speed, and how to safely deploy changes in production.

### Why This Exists

In production, you update applications constantly — new features, bug fixes, security patches. Rolling updates let you deploy new versions while the old version continues serving traffic. No maintenance windows, no downtime.

### YAML Examples

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # At most 8 pods exist during update
      maxUnavailable: 0  # Always keep 6 serving traffic

  # To use Recreate instead (downtime accepted):
  # strategy:
  #   type: Recreate  # Kill all old pods, then start new pods
```

### Safe Deployment Checklist

```bash
# 1. Diff what will change
kubectl diff -f deployment.yaml

# 2. Apply with record (deprecated but still used)
kubectl apply -f deployment.yaml

# 3. Watch rollout
kubectl rollout status deployment/my-app

# 4. Verify pod health
kubectl get pods -l app=my-app

# 5. Quick smoke test
kubectl port-forward deployment/my-app 8080:8080 &
curl http://localhost:8080/health

# 6. Check logs
kubectl logs deployment/my-app --tail=50

# 7. Monitor for 5-10 minutes, then confirm
# 8. If something's wrong - instant rollback:
kubectl rollout undo deployment/my-app
```

---

## Chapter 21 — Autoscaling

### What You Will Learn
How to automatically scale your application based on CPU, memory, or custom metrics, and how to scale nodes automatically.

### Why This Exists

Manual scaling is slow and expensive. Autoscaling means your application handles traffic spikes automatically and reduces resource usage during quiet periods — saving money while maintaining performance.

### Types of Autoscaling

```
HPA  (Horizontal Pod Autoscaler) → Add/remove PODS based on metrics
VPA  (Vertical Pod Autoscaler)   → Increase/decrease POD resource limits
CA   (Cluster Autoscaler)        → Add/remove NODES based on pod demand
KEDA (Event-Driven Autoscaling)  → Scale based on external events (queue depth, etc.)
```

### YAML Examples

#### HPA — Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  minReplicas: 3   # Never go below 3
  maxReplicas: 20  # Never exceed 20

  metrics:
  # Scale on CPU usage
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Scale up when avg CPU > 70%

  # Scale on Memory usage
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

  # Scale on custom metric (requests per second)
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60   # Wait 60s before scaling up again
      policies:
      - type: Pods
        value: 4          # Add at most 4 pods at a time
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5min before scaling down
      policies:
      - type: Percent
        value: 25         # Remove at most 25% at a time
        periodSeconds: 60
```

#### VPA — Vertical Pod Autoscaler

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Auto, Off, or Initial
  resourcePolicy:
    containerPolicies:
    - containerName: my-app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: "4"
        memory: 4Gi
```

### Commands

```bash
# Create HPA
kubectl apply -f hpa.yaml

# Or create quickly
kubectl autoscale deployment my-app --cpu-percent=70 --min=3 --max=20

# View HPA status
kubectl get hpa
kubectl describe hpa my-app-hpa

# Watch HPA in real time
kubectl get hpa -w

# Generate load to test autoscaling
kubectl run load-generator --image=busybox --restart=Never -- \
  /bin/sh -c "while true; do wget -q -O- http://my-service/; done"
```

---

## Chapter 22 — Jobs

### What You Will Learn
How to run one-off tasks and batch workloads in Kubernetes using Jobs.

### Why This Exists

Not everything runs forever. Database migrations, data processing, report generation, batch imports — these run to completion and stop. Jobs are the Kubernetes resource for finite workloads.

### YAML Examples

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  completions: 1          # How many times to run successfully
  parallelism: 1          # How many pods to run at once
  backoffLimit: 3         # Retry up to 3 times on failure
  activeDeadlineSeconds: 300  # Kill job after 5 minutes

  template:
    spec:
      restartPolicy: OnFailure  # Required: OnFailure or Never
      containers:
      - name: migration
        image: my-app:1.0
        command: ['python', 'manage.py', 'migrate']
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
---
# Parallel Job (process multiple items simultaneously)
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-processor
spec:
  completions: 10   # Process 10 items total
  parallelism: 3    # 3 at a time
  backoffLimit: 2
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: worker
        image: data-processor:1.0
        command: ['python', 'process.py']
```

### Commands

```bash
# Create a job
kubectl apply -f job.yaml

# List jobs
kubectl get jobs

# Watch job progress
kubectl get jobs -w

# See pods created by job
kubectl get pods -l job-name=my-job

# View job logs
kubectl logs job/my-job

# Delete completed jobs
kubectl delete job my-job
```

---

## Chapter 23 — CronJobs

### What You Will Learn
How to schedule recurring tasks in Kubernetes using CronJobs.

### Why This Exists

Backups, reports, cleanup tasks, data syncs — these need to run on a schedule. CronJobs are Kubernetes Jobs that run automatically on a cron schedule.

### YAML Examples

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
  namespace: production
spec:
  # Cron schedule: minute hour day month day-of-week
  schedule: "0 2 * * *"  # Every day at 2:00 AM

  # How many successful jobs to keep
  successfulJobsHistoryLimit: 3

  # How many failed jobs to keep
  failedJobsHistoryLimit: 1

  # Don't run if previous job hasn't finished
  concurrencyPolicy: Forbid  # Allow, Forbid, or Replace

  # Start within 60s of scheduled time or skip
  startingDeadlineSeconds: 60

  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: postgres:15
            command:
            - /bin/sh
            - -c
            - |
              TIMESTAMP=$(date +%Y%m%d_%H%M%S)
              pg_dump $DATABASE_URL | gzip > /backup/db_$TIMESTAMP.sql.gz
              aws s3 cp /backup/db_$TIMESTAMP.sql.gz s3://my-backups/
            env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: url
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: aws-credentials
                  key: access-key
```

### Cron Schedule Quick Reference

```
┌──────── minute (0-59)
│ ┌────── hour (0-23)
│ │ ┌──── day of month (1-31)
│ │ │ ┌── month (1-12)
│ │ │ │ ┌ day of week (0-7, 0 and 7 are Sunday)
│ │ │ │ │
* * * * *

"0 2 * * *"    = Every day at 2 AM
"*/5 * * * *"  = Every 5 minutes
"0 */6 * * *"  = Every 6 hours
"0 0 * * 0"   = Every Sunday at midnight
"0 9 * * 1-5" = Weekdays at 9 AM
```

---

## Chapter 24 — RBAC

### What You Will Learn
How to control who can do what in your Kubernetes cluster using Role-Based Access Control.

### Why This Exists

In a shared cluster, you don't want every developer able to delete production Pods, view Secrets, or modify cluster-wide settings. RBAC lets you grant the minimum permissions needed — the principle of least privilege.

### How It Works Internally

```
Subject (who)      → RoleBinding/ClusterRoleBinding → Role/ClusterRole (what)

Subject types:
- User (human, authenticated via certificate or OIDC)
- Group (set of users)
- ServiceAccount (application/pod identity)

Role types:
- Role: permissions within a namespace
- ClusterRole: permissions across all namespaces (cluster-wide)

Binding types:
- RoleBinding: binds Role or ClusterRole within a namespace
- ClusterRoleBinding: binds ClusterRole across all namespaces
```

### YAML Examples

#### Role and RoleBinding

```yaml
# Role: What is allowed
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]           # "" = core API group
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list"]
---
# RoleBinding: Who gets the Role
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
# Bind to a user
- kind: User
  name: alice@company.com
  apiGroup: rbac.authorization.k8s.io
# Bind to a group
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
# Bind to a ServiceAccount
- kind: ServiceAccount
  name: my-app-sa
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### ClusterRole for Admins

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-readonly
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: readonly-cluster-admin
subjects:
- kind: Group
  name: sre-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-readonly
  apiGroup: rbac.authorization.k8s.io
```

#### ServiceAccount for Applications

```yaml
# Create a ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: production
---
# Give it minimal permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-app-role
  namespace: production
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
  resourceNames: ["my-app-config"]  # Only this specific ConfigMap
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: production
roleRef:
  kind: Role
  name: my-app-role
  apiGroup: rbac.authorization.k8s.io
---
# Use the ServiceAccount in a Pod
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  serviceAccountName: my-app-sa  # Use this ServiceAccount
  containers:
  - name: app
    image: my-app:1.0
```

### Commands

```bash
# Check what you can do
kubectl auth can-i get pods
kubectl auth can-i delete deployments -n production
kubectl auth can-i '*' '*'  # Can I do everything?

# Check what someone else can do
kubectl auth can-i get pods --as alice@company.com
kubectl auth can-i get pods --as system:serviceaccount:production:my-app-sa

# List all roles
kubectl get roles -A
kubectl get clusterroles

# List all bindings
kubectl get rolebindings -A
kubectl get clusterrolebindings

# Describe a role
kubectl describe role pod-reader -n production

# Get all RBAC for a user
kubectl get rolebindings -A -o json | \
  jq '.items[] | select(.subjects[]?.name == "alice")'
```

---

## Chapter 25 — Security

### What You Will Learn
How to harden Kubernetes workloads, Pod Security Standards, network policies, and production security best practices.

### Why This Exists

A default Kubernetes Pod can do many dangerous things — run as root, mount host filesystems, escalate privileges, access the Kubernetes API. Production clusters must be hardened.

### Pod Security Standards

```yaml
# Assign Pod Security Standard to namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # Enforce: reject pods that don't meet standard
    pod-security.kubernetes.io/enforce: restricted
    # Warn: allow but warn about violations
    pod-security.kubernetes.io/warn: restricted
    # Audit: log violations
    pod-security.kubernetes.io/audit: restricted
```

### Security Context

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  # Pod-level security context
  securityContext:
    runAsNonRoot: true       # Must run as non-root user
    runAsUser: 1000          # Run as user ID 1000
    runAsGroup: 3000         # Run as group ID 3000
    fsGroup: 2000            # Files created in volumes owned by group 2000
    seccompProfile:
      type: RuntimeDefault   # Use container runtime's default seccomp profile

  containers:
  - name: app
    image: my-app:1.0
    # Container-level security context (overrides pod-level)
    securityContext:
      allowPrivilegeEscalation: false  # Cannot gain more privileges
      readOnlyRootFilesystem: true     # Filesystem is read-only
      capabilities:
        drop:
        - ALL              # Drop ALL Linux capabilities
        add:
        - NET_BIND_SERVICE # Add back only what you need
```

### Security Best Practices

```yaml
# Production secure deployment template
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      # Don't auto-mount service account token unless needed
      automountServiceAccountToken: false

      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault

      containers:
      - name: app
        image: registry.company.com/app:1.0.0  # Use private registry
        imagePullPolicy: Always  # Always pull to get latest security patches

        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: [ALL]

        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"

        # Writable directories when readOnlyRootFilesystem: true
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: cache
          mountPath: /app/cache

      volumes:
      - name: tmp
        emptyDir: {}
      - name: cache
        emptyDir: {}
```

---

# LEVEL 5 — EXPERT

---

## Chapter 26 — Helm

### What You Will Learn
How to use Helm as a Kubernetes package manager, how to write Helm charts, and how to manage application releases.

### Why This Exists

Deploying a complex application to Kubernetes means managing many YAML files — Deployments, Services, ConfigMaps, Secrets, Ingress, RBAC, etc. Helm packages all of these into a single deployable unit called a Chart. It also handles version management, configuration templating, and upgrades.

### How It Works Internally

```
Helm Chart (your package)
├── Chart.yaml     (metadata: name, version, description)
├── values.yaml    (default configuration values)
├── templates/     (Kubernetes YAML templates)
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl (template helpers)
└── charts/        (sub-charts/dependencies)

Helm Release = Chart installed with specific values into a namespace
```

### Installation

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version

# Add official stable repo
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# Search for charts
helm search repo nginx
helm search hub wordpress  # Search Artifact Hub

# Update repos
helm repo update
```

### Using Existing Charts

```bash
# Install nginx ingress controller
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace

# Install with custom values
helm install my-postgres bitnami/postgresql \
  --namespace production \
  --set auth.postgresPassword=secretpassword \
  --set primary.persistence.size=20Gi

# Install from values file
helm install my-app ./my-chart -f values-production.yaml

# Upgrade a release
helm upgrade my-postgres bitnami/postgresql \
  --namespace production \
  --set primary.persistence.size=50Gi  # Increase storage

# Rollback a release
helm rollback my-app 1  # Roll back to revision 1

# Uninstall
helm uninstall my-app -n production

# List releases
helm list -A

# View release history
helm history my-app

# Get values used for a release
helm get values my-app

# Render templates without installing (debug)
helm template my-app ./my-chart -f values.yaml
```

### Creating a Helm Chart

```bash
# Create a new chart skeleton
helm create my-app

# Chart structure:
# my-app/
# ├── Chart.yaml
# ├── values.yaml
# ├── charts/
# └── templates/
#     ├── deployment.yaml
#     ├── service.yaml
#     ├── ingress.yaml
#     ├── serviceaccount.yaml
#     ├── hpa.yaml
#     ├── NOTES.txt
#     └── _helpers.tpl
```

### Chart.yaml

```yaml
apiVersion: v2
name: my-app
description: A Helm chart for My Application
type: application
version: 1.2.0       # Chart version
appVersion: "2.1.0"  # Application version

dependencies:
- name: postgresql
  version: "12.x.x"
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
```

### values.yaml

```yaml
# Default values (overrideable per environment)
replicaCount: 2

image:
  repository: my-registry/my-app
  tag: "2.1.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: false
  host: ""
  tls: false

resources:
  requests:
    memory: 128Mi
    cpu: 100m
  limits:
    memory: 256Mi
    cpu: 500m

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

postgresql:
  enabled: true
  auth:
    postgresPassword: ""  # Override this in production!
    database: myapp
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: {{ .Values.service.targetPort }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- if .Values.ingress.enabled }}
        env:
        - name: BASE_URL
          value: {{ .Values.ingress.host | quote }}
        {{- end }}
```

### Environment-Specific Values

```yaml
# values-production.yaml
replicaCount: 5

image:
  tag: "2.1.0"

ingress:
  enabled: true
  host: api.example.com
  tls: true

resources:
  requests:
    memory: 512Mi
    cpu: 500m
  limits:
    memory: 1Gi
    cpu: "2"

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 30

postgresql:
  auth:
    postgresPassword: "use-external-secret-here"
```

```bash
# Deploy to production with production values
helm upgrade --install my-app ./my-chart \
  -f values.yaml \
  -f values-production.yaml \
  --namespace production \
  --create-namespace \
  --atomic \        # Rollback automatically if deployment fails
  --timeout 5m
```

---

## Chapter 27 — Monitoring

### What You Will Learn
How to set up monitoring for Kubernetes with Prometheus and Grafana, what metrics to collect, and how to alert on problems.

### Why This Exists

You cannot manage what you cannot measure. In production, you need to know: Is my application healthy? Is it slow? Are resources running out? Is the cluster behaving normally? Monitoring answers all of these.

### Architecture

```
Application → Prometheus (scrapes metrics) → Grafana (visualizes)
                                          → Alertmanager (sends alerts)

kube-state-metrics   → Kubernetes object state (pod counts, deployment status)
node-exporter        → Node hardware metrics (CPU, memory, disk, network)
cAdvisor             → Container resource usage
```

### Installation via kube-prometheus-stack

```bash
# Add Prometheus community Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack (includes Prometheus, Grafana, Alertmanager)
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=admin123 \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi

# Access Grafana
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
# Open http://localhost:3000, login admin/admin123
```

### Exposing Application Metrics

```yaml
# Add Prometheus annotations to your Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/path: "/metrics"
        prometheus.io/port: "9090"
    spec:
      containers:
      - name: app
        image: my-app:1.0
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 9090
          name: metrics
```

### ServiceMonitor (Prometheus Operator)

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
  namespace: monitoring
  labels:
    release: monitoring  # Must match Prometheus selector
spec:
  selector:
    matchLabels:
      app: my-app
  namespaceSelector:
    matchNames:
    - production
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### PrometheusRule (Alerting Rules)

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-alerts
  namespace: monitoring
  labels:
    release: monitoring
spec:
  groups:
  - name: my-app
    rules:
    # Alert if pod is down for 5 minutes
    - alert: PodDown
      expr: up{job="my-app"} == 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod {{ $labels.pod }} is down"
        description: "Pod has been down for more than 5 minutes"

    # Alert if high error rate
    - alert: HighErrorRate
      expr: |
        rate(http_requests_total{status=~"5.."}[5m])
        / rate(http_requests_total[5m]) > 0.05
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High error rate on {{ $labels.job }}"

    # Alert if memory usage > 90%
    - alert: HighMemoryUsage
      expr: |
        container_memory_usage_bytes{container!=""}
        / container_spec_memory_limit_bytes{container!=""} > 0.9
      for: 5m
      labels:
        severity: warning
```

### Essential Grafana Dashboards

Import these by ID in Grafana:
- **1860** — Node Exporter Full (node metrics)
- **3119** — Kubernetes cluster monitoring
- **6417** — Kubernetes pod and cluster monitoring
- **315** — Kubernetes cluster monitoring (Prometheus)

---

## Chapter 28 — Logging

### What You Will Learn
How to collect, store, and query logs from all your Kubernetes applications in a centralized logging system.

### Why This Exists

When 50 pods spread across 20 nodes are all generating logs, you cannot SSH into each node and run `kubectl logs` to debug issues. You need centralized logging where all logs flow to one place and you can search across all of them.

### Architecture: ELK Stack vs Loki

```
Option 1 — ELK Stack:
Pods → Filebeat/Fluentd (collector) → Logstash (transform) → Elasticsearch → Kibana

Option 2 — Loki (Lighter, Recommended):
Pods → Promtail (collector) → Loki (storage) → Grafana (query/visualize)
```

### Loki + Promtail Installation

```bash
# Add Grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts

# Install Loki stack
helm install loki grafana/loki-stack \
  --namespace logging \
  --create-namespace \
  --set promtail.enabled=true \
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=50Gi

# If Grafana already installed, add Loki as data source
# In Grafana UI: Configuration → Data Sources → Add → Loki
# URL: http://loki.logging:3100
```

### Structured Logging Best Practices

```javascript
// Your application should output structured JSON logs
// BAD:
console.log("User 12345 logged in at 2024-01-15 14:30:00");

// GOOD:
console.log(JSON.stringify({
  timestamp: new Date().toISOString(),
  level: "info",
  message: "User logged in",
  userId: 12345,
  requestId: req.headers['x-request-id'],
  duration_ms: 45
}));

// This makes logs searchable in Loki/Kibana by any field
```

### Querying Logs with LogQL (Loki)

```
# All logs from a pod
{pod="my-app-abc123"}

# All logs from pods with label app=my-app
{app="my-app"}

# Filter to error level
{app="my-app"} |= "ERROR"
{app="my-app"} | json | level="error"

# Count errors per minute
count_over_time({app="my-app"} |= "ERROR" [1m])

# Parse JSON and filter
{namespace="production"} | json | statusCode=500
```

---

## Chapter 29 — Debugging

### What You Will Learn
A systematic approach to debugging any Kubernetes problem, from pod crashes to networking issues to performance problems.

### The Kubernetes Debugging Framework

```
When something is broken, work through this hierarchy:

1. CLUSTER LEVEL  → Are nodes healthy? Is control plane up?
2. NAMESPACE      → Are resources in the right namespace?
3. POD            → Is the pod running? What's the state?
4. CONTAINER      → What's in the logs? Exit code?
5. NETWORKING     → Can pods communicate? Service endpoints?
6. STORAGE        → Are volumes mounted? PVC bound?
7. CONFIG         → Are ConfigMaps/Secrets correct?
8. RESOURCES      → Are there enough CPU/memory?
```

### kubectl logs — Deep Dive

```bash
# Basic logs
kubectl logs my-pod

# Follow logs in real time (like tail -f)
kubectl logs -f my-pod

# Last 100 lines
kubectl logs my-pod --tail=100

# Logs from crashed previous instance
kubectl logs my-pod --previous

# Specific container in multi-container pod
kubectl logs my-pod -c sidecar-container

# All containers at once
kubectl logs my-pod --all-containers

# Logs from all pods matching a label
kubectl logs -l app=my-app --all-containers

# Logs from last hour
kubectl logs my-pod --since=1h

# Timestamp each line
kubectl logs my-pod --timestamps

# Stream logs from all pods in a deployment (not built-in, use stern)
stern my-app -n production
```

### kubectl describe — Reading Events

```bash
# The most important debugging command
kubectl describe pod my-pod

# Output sections:
# Name, Namespace, Labels, Annotations → metadata
# Status: Running/Pending/Failed
# IP, Node → where is it running
# Containers: state, image, restarts, resource limits
# Conditions: Initialized, Ready, ContainersReady, PodScheduled
# Volumes: mounted volumes
# Events: WHAT HAPPENED (most important section)

# Events tell you:
# "FailedScheduling" → can't find a node (insufficient resources?)
# "Pulling image" → image pull in progress
# "Failed to pull image" → image not found or auth failure
# "Started" → container started
# "Killing" → container being killed (liveness probe?)
# "BackOff" → container crashing and restarting
```

### kubectl exec — Debugging Inside Containers

```bash
# Shell into a running container
kubectl exec -it my-pod -- bash
kubectl exec -it my-pod -- sh  # if bash not available

# Check environment variables
kubectl exec my-pod -- env

# Check network connectivity
kubectl exec my-pod -- curl http://other-service
kubectl exec my-pod -- nslookup other-service

# Check disk usage
kubectl exec my-pod -- df -h

# Check processes
kubectl exec my-pod -- ps aux

# Check listening ports
kubectl exec my-pod -- ss -tlnp

# Check file contents
kubectl exec my-pod -- cat /etc/config/app.yaml

# Run a debug container (when main container has no shell)
kubectl debug -it my-pod --image=busybox --target=my-container
```

### Debugging Scenarios

#### Pod Stuck in Pending

```bash
kubectl describe pod my-pod
# Look at Events:
# "0/3 nodes are available: 3 Insufficient memory"
# → Nodes don't have enough memory

# Check node capacity
kubectl describe nodes | grep -A5 "Allocated resources"

# Solution: Increase node count or reduce pod memory requests
```

#### CrashLoopBackOff

```bash
# Get logs from crashed container
kubectl logs my-pod --previous

# Get exit code
kubectl describe pod my-pod | grep "Exit Code"
# Exit Code 1: Application error
# Exit Code 137: OOMKilled (out of memory)
# Exit Code 143: SIGTERM (graceful stop)

# Solutions:
# If OOMKilled: increase memory limit
# If app error: check application logs for exception
# If config error: check ConfigMap/Secret values
```

#### Service Not Routing Traffic

```bash
# Check if service has endpoints
kubectl get endpoints my-service
# If empty, the selector doesn't match any pods

# Check labels on pods
kubectl get pods --show-labels

# Check service selector
kubectl describe service my-service | grep Selector

# Verify selector matches pod labels
kubectl get pods -l app=my-app  # Should return pods
```

#### ImagePullBackOff

```bash
kubectl describe pod my-pod
# Events will show:
# "Failed to pull image: ... not found" → wrong image name/tag
# "... authentication required" → need imagePullSecret

# Fix wrong image
kubectl set image deployment/my-app my-app=correct-image:correct-tag

# Fix auth - create docker registry secret
kubectl create secret docker-registry registry-creds \
  --docker-server=my-registry.com \
  --docker-username=myuser \
  --docker-password=mypassword

# Add to pod spec:
# imagePullSecrets:
# - name: registry-creds
```

---

## Chapter 30 — Production Architecture

### What You Will Learn
How to design a production-grade Kubernetes cluster with high availability, disaster recovery, and operational excellence.

### Production Architecture Pattern

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     PRODUCTION KUBERNETES CLUSTER                        │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                      CONTROL PLANE (HA)                          │    │
│  │                                                                   │    │
│  │  Master-1 (AZ-A)   Master-2 (AZ-B)   Master-3 (AZ-C)           │    │
│  │  API Server         API Server         API Server                │    │
│  │  etcd               etcd               etcd                      │    │
│  │  Controller Mgr     Controller Mgr     Controller Mgr            │    │
│  │  Scheduler          Scheduler          Scheduler                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │
│  │   WORKER AZ-A   │  │   WORKER AZ-B   │  │   WORKER AZ-C   │         │
│  │                 │  │                 │  │                 │         │
│  │  Node1  Node2   │  │  Node3  Node4   │  │  Node5  Node6   │         │
│  │  [Pods] [Pods]  │  │  [Pods] [Pods]  │  │  [Pods] [Pods]  │         │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘         │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    PLATFORM SERVICES                              │    │
│  │  Ingress Nginx  │  Cert Manager  │  External-DNS  │  Vault       │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    OBSERVABILITY                                  │    │
│  │  Prometheus  │  Grafana  │  Loki  │  Tempo  │  Alertmanager     │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### Production Checklist

```
CLUSTER SETUP:
✓ 3+ control plane nodes across availability zones
✓ etcd backed up hourly to object storage (S3/GCS)
✓ Worker nodes across 3+ availability zones
✓ Node autoscaling (Cluster Autoscaler) configured
✓ Private API endpoint with VPN access only
✓ Kubernetes version within 2 minor versions of latest

NETWORKING:
✓ CNI plugin installed (Calico or Cilium)
✓ NetworkPolicies: default-deny, explicit allows
✓ Ingress controller with TLS termination
✓ Certificate manager (cert-manager) for automatic TLS
✓ Private registry for container images

SECURITY:
✓ RBAC enabled and configured
✓ Pod Security Standards enforced (restricted)
✓ Secrets encrypted at rest in etcd
✓ No privileged containers in production
✓ Image scanning in CI/CD pipeline
✓ Audit logging enabled

OBSERVABILITY:
✓ Prometheus + Grafana for metrics
✓ Centralized logging (Loki or ELK)
✓ Distributed tracing (Jaeger or Tempo)
✓ Alerting rules for critical conditions
✓ PagerDuty/OpsGenie integration

OPERATIONS:
✓ GitOps deployment (ArgoCD or Flux)
✓ Resource quotas per namespace
✓ Pod disruption budgets configured
✓ Backup strategy for stateful applications
✓ Disaster recovery runbook documented and tested
✓ On-call rotation and escalation policies
```

---

## Chapter 31 — Multi-Node Cluster

### What You Will Learn
How to configure nodes for specific workloads using taints, tolerations, and node affinity.

### Taints and Tolerations

```
Taint: Applied to a NODE — "Don't schedule pods here unless they tolerate this"
Toleration: Applied to a POD — "I can tolerate this taint, schedule me here"
```

```yaml
# Taint a node (kubectl)
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule
kubectl taint nodes gpu-node-1 gpu=true:NoExecute  # Also evict existing pods

# Toleration in Pod spec
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

# Node Affinity (prefer or require specific nodes)
spec:
  affinity:
    nodeAffinity:
      # REQUIRED: Must run on nodes with disktype=ssd
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd

      # PREFERRED: Try to run on high-memory nodes
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: memory-tier
            operator: In
            values:
            - high
```

### Pod Disruption Budgets

```yaml
# Ensure minimum pods available during node drains/updates
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2  # Or use maxUnavailable: 1
  selector:
    matchLabels:
      app: my-app
```

---

## Chapter 32 — Kubernetes Internals

### What You Will Learn
How Kubernetes controllers work, the watch/reconcile pattern, and how to think like a Kubernetes engineer.

### The Control Loop Pattern

Every Kubernetes controller follows the same pattern:

```
1. WATCH  → Subscribe to changes in desired state
2. OBSERVE → Get current actual state
3. DIFF   → Compare desired vs actual
4. ACT    → Make changes to reach desired state
5. REPEAT → Go back to step 1

This is the reconciliation loop.
It runs forever, continuously.
It is eventually consistent.
It is self-healing.
```

### The Kubernetes API

```bash
# See all API groups and versions
kubectl api-versions

# See all resource types
kubectl api-resources

# See API for a specific resource
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.resources

# Make raw API calls
kubectl get --raw /api/v1/namespaces
kubectl get --raw /apis/apps/v1/deployments
```

### Custom Resource Definitions (CRD)

```yaml
# Define a custom resource type
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.example.com
spec:
  group: example.com
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
              replicas:
                type: integer
              image:
                type: string
  scope: Namespaced
  names:
    plural: myapps
    singular: myapp
    kind: MyApp
---
# Now use your custom resource
apiVersion: example.com/v1
kind: MyApp
metadata:
  name: my-custom-app
spec:
  replicas: 3
  image: my-image:1.0
```

---

# LEVEL 6 — MASTER

---

## Chapter 33 — Networking Deep Dive

### What You Will Learn
iptables rules, IPVS, kube-proxy internals, DNS internals, and how to debug complex networking issues.

### iptables and Service Traffic

```bash
# See all iptables rules related to Kubernetes
iptables -t nat -L KUBE-SERVICES

# See rules for a specific service
iptables -t nat -L | grep my-service

# Check kube-proxy mode
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode

# Check DNS resolution inside cluster
kubectl run debug-pod --image=busybox --rm -it -- \
  nslookup kubernetes.default.svc.cluster.local

# Check DNS server
kubectl get configmap coredns -n kube-system -o yaml
```

### CoreDNS Configuration

```yaml
# CoreDNS ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

### Network Troubleshooting Toolkit

```bash
# Deploy a debugging pod with network tools
kubectl run netshoot --image=nicolaka/netshoot -it --rm -- bash

# Inside the pod:
nslookup my-service.namespace.svc.cluster.local
dig my-service.namespace.svc.cluster.local
curl -v http://my-service:8080/health
traceroute my-service
tcpdump -i eth0 -n port 8080
ss -tlnp
```

---

## Chapter 34 — Service Mesh

### What You Will Learn
What a service mesh is, how Istio or Linkerd works, and what problems they solve.

### Why This Exists

In a microservices architecture with 50+ services, you need: mutual TLS between services, advanced traffic routing (canary, A/B testing), observability (distributed tracing, metrics), circuit breaking, and retries. Implementing all of this in every service is impossible. A service mesh does it transparently.

### Architecture

```
WITHOUT service mesh:
Service A → [HTTP, no auth] → Service B

WITH service mesh (Istio):
Service A → Envoy Sidecar → [mTLS, encrypted] → Envoy Sidecar → Service B
              ↓                                         ↓
           Observability                           Observability
           Traffic Control                         Traffic Control
```

### Istio Installation

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.x.x
export PATH=$PWD/bin:$PATH

# Install Istio on cluster
istioctl install --set profile=production -y

# Enable sidecar injection for namespace
kubectl label namespace production istio-injection=enabled

# Now every new Pod gets an Envoy sidecar automatically

# Install observability addons
kubectl apply -f samples/addons  # Kiali, Jaeger, Prometheus, Grafana

# Access Kiali dashboard (service mesh topology)
istioctl dashboard kiali
```

### Istio Traffic Management

```yaml
# Virtual Service: Advanced routing
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app-vs
spec:
  hosts:
  - my-app
  http:
  # Canary: 90% to v1, 10% to v2
  - route:
    - destination:
        host: my-app
        subset: v1
      weight: 90
    - destination:
        host: my-app
        subset: v2
      weight: 10
---
# Destination Rule: Define subsets
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-app-dr
spec:
  host: my-app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

---

## Chapter 35 — Performance Optimization

### What You Will Learn
How to optimize Kubernetes cluster performance, application performance, and reduce costs.

### Optimization Areas

#### 1. Scheduler Performance

```yaml
# Use Pod Topology Spread Constraints for even distribution
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: my-app
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app: my-app
```

#### 2. Resource Right-Sizing

```bash
# Use VPA in "Off" mode to get recommendations without auto-applying
kubectl apply -f vpa-off.yaml

# Check VPA recommendations
kubectl describe vpa my-app-vpa
# Look for:
# Container Recommendations:
#   Lower Bound: 50m cpu, 100Mi memory
#   Target:      200m cpu, 300Mi memory
#   Upper Bound: 500m cpu, 800Mi memory

# Apply the recommended values to your deployment
```

#### 3. Startup Time Optimization

```bash
# Use init containers to pre-warm caches
# Use readiness gates for dependencies
# Pre-pull images on nodes:
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: image-preloader
spec:
  selector:
    matchLabels:
      name: image-preloader
  template:
    metadata:
      labels:
        name: image-preloader
    spec:
      initContainers:
      - name: pull-image
        image: my-app:2.0  # Pre-pull this image on all nodes
        command: ['true']
      containers:
      - name: pause
        image: gcr.io/google_containers/pause:3.1
EOF
```

#### 4. etcd Performance

```bash
# Monitor etcd latency (should be <10ms)
kubectl exec -it etcd-master -n kube-system -- \
  etcdctl endpoint status --write-out=table

# Defragment etcd (run during low traffic)
kubectl exec -it etcd-master -n kube-system -- \
  etcdctl defrag --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Compact etcd revision history
kubectl exec -it etcd-master -n kube-system -- \
  etcdctl compact $(etcdctl endpoint status --write-out=json | \
  jq '.[0].Status.header.revision')
```

#### 5. Cost Optimization

```yaml
# Use Spot/Preemptible instances for non-critical workloads
# with node affinity + toleration

# Mark spot node pool
kubectl taint nodes spot-node-1 spot=true:NoSchedule
kubectl label nodes spot-node-1 node-type=spot

# Pod that prefers spot but falls back to on-demand
spec:
  tolerations:
  - key: spot
    operator: Equal
    value: "true"
    effect: NoSchedule
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: node-type
            operator: In
            values:
            - spot
```

---

## Chapter 36 — CI/CD Kubernetes

### What You Will Learn
How to build a CI/CD pipeline that deploys to Kubernetes automatically and safely.

### Pipeline Architecture

```
Developer pushes code
    ↓
CI (GitHub Actions / GitLab CI / Jenkins)
    ├── Run tests
    ├── Build Docker image
    ├── Push image to registry
    └── Update Kubernetes manifests
          ↓
CD (ArgoCD / Flux)
    ├── Detect manifest changes in Git
    ├── Validate changes
    ├── Deploy to staging
    ├── Run smoke tests
    └── Deploy to production (with approval gate)
```

### GitHub Actions Example

```yaml
# .github/workflows/deploy.yaml
name: Build and Deploy

on:
  push:
    branches: [main]

env:
  IMAGE_NAME: my-app
  REGISTRY: ghcr.io/myorg

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Run tests
      run: |
        npm install
        npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
    steps:
    - uses: actions/checkout@v3

    - name: Login to registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=sha,prefix={{branch}}-,format=short

    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - uses: actions/checkout@v3

    - name: Update staging manifest
      run: |
        # Update image tag in deployment YAML
        sed -i "s|image: .*my-app:.*|image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.build.outputs.image-tag }}|" \
          k8s/staging/deployment.yaml

    - name: Apply to staging
      uses: azure/k8s-deploy@v4
      with:
        namespace: staging
        manifests: k8s/staging/
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.build.outputs.image-tag }}

  deploy-production:
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production  # Requires manual approval
    steps:
    - uses: actions/checkout@v3

    - name: Deploy to production
      uses: azure/k8s-deploy@v4
      with:
        namespace: production
        manifests: k8s/production/
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.build.outputs.image-tag }}
        strategy: canary
        percentage: 20  # Deploy to 20% of pods first
```

---

## Chapter 37 — GitOps

### What You Will Learn
How to implement GitOps with ArgoCD, making Git the single source of truth for all cluster state.

### Why This Exists

GitOps is the practice of using Git as the single source of truth for declarative infrastructure. Every change to the cluster goes through a Pull Request. History is in Git. Rollback is a git revert. Drift is detected and corrected automatically.

### ArgoCD Installation and Setup

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available deployment -l "app.kubernetes.io/name=argocd-server" \
  -n argocd --timeout=120s

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Port forward ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Install ArgoCD CLI
curl -sSL -o argocd-linux-amd64 \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

# Login
argocd login localhost:8080 --username admin --password <password> --insecure
```

### ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-production
  namespace: argocd
spec:
  project: default

  # Git repository containing Kubernetes manifests
  source:
    repoURL: https://github.com/myorg/my-app-config
    targetRevision: main
    path: k8s/production  # Directory with K8s YAML files

  # Where to deploy
  destination:
    server: https://kubernetes.default.svc
    namespace: production

  # Sync policy
  syncPolicy:
    automated:
      prune: true     # Delete resources removed from Git
      selfHeal: true  # Revert manual changes to cluster
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    retry:
      limit: 3
      backoff:
        duration: 5s
        maxDuration: 3m
```

### GitOps Repository Structure

```
my-app-config/ (Git Repository)
├── k8s/
│   ├── base/           (common manifests)
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── patches/
│   │       └── replicas.yaml  (replicas: 1)
│   └── production/
│       ├── kustomization.yaml
│       └── patches/
│           └── replicas.yaml  (replicas: 5)
├── infrastructure/
│   ├── ingress-nginx/
│   ├── cert-manager/
│   └── monitoring/
└── apps/
    ├── staging.yaml    (ArgoCD Application for staging)
    └── production.yaml (ArgoCD Application for production)
```

---

## Chapter 38 — Multi-Cluster

### What You Will Learn
How to manage multiple Kubernetes clusters, federation patterns, and disaster recovery across clusters.

### Why This Exists

Large organizations run multiple clusters for: different environments (prod/staging/dev), different regions (US/EU/APAC), disaster recovery, compliance (data sovereignty), and workload isolation.

### Multi-Cluster Patterns

```
Pattern 1: Hub-Spoke
  Hub Cluster (management) → manages → Spoke Clusters (workloads)
  Tools: Fleet (Rancher), ArgoCD multi-cluster

Pattern 2: Active-Active
  Cluster A (US-East)   ←→   Cluster B (US-West)
  Both serve traffic         Failover ready
  Tools: Submariner (cross-cluster networking)

Pattern 3: Active-Passive
  Primary Cluster → replicates to → DR Cluster
  DR cluster only activated on failure
  Tools: Velero (backup/restore)
```

### ArgoCD Multi-Cluster Setup

```bash
# Add a remote cluster to ArgoCD
argocd cluster add production-us-east \
  --kubeconfig /path/to/prod-kubeconfig \
  --name us-east-production

# Create an ApplicationSet (deploy to multiple clusters)
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-app-all-clusters
  namespace: argocd
spec:
  generators:
  - clusters: {}  # Generate one Application per cluster
  template:
    metadata:
      name: '{{name}}-my-app'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/config
        targetRevision: main
        path: 'k8s/{{metadata.labels.environment}}'
      destination:
        server: '{{server}}'
        namespace: my-app
      syncPolicy:
        automated:
          selfHeal: true
```

---

## Chapter 39 — Kubernetes at Scale

### What You Will Learn
How to operate Kubernetes clusters with hundreds of nodes and thousands of workloads.

### Scale Challenges and Solutions

#### 1. API Server at Scale

```yaml
# kube-apiserver flags for large clusters
--max-requests-inflight=400     # Concurrent non-mutating requests
--max-mutating-requests-inflight=200  # Concurrent mutating requests
--watch-cache-sizes=node#100,pod#1000  # Increase watch cache

# API Priority and Fairness (APF) - prioritize important requests
apiVersion: flowcontrol.apiserver.k8s.io/v1beta3
kind: PriorityLevelConfiguration
metadata:
  name: critical-system
spec:
  type: Limited
  limited:
    assuredConcurrencyShares: 100  # High priority gets more shares
    limitResponse:
      type: Queue
      queuing:
        queues: 64
        handSize: 6
        queueLengthLimit: 50
```

#### 2. etcd at Scale

```bash
# etcd performance targets for large clusters:
# < 10ms p99 disk write latency
# < 100ms p99 network peer latency
# Use dedicated SSD disks for etcd
# Use dedicated nodes for etcd (separate from control plane)

# Monitor etcd
kubectl exec -it etcd-master -n kube-system -- \
  etcdctl endpoint status --write-out=table
```

#### 3. Large Cluster Best Practices

| Cluster Size | Recommendation |
|---|---|
| <50 nodes | Single master, managed K8s fine |
| 50-500 nodes | 3 masters, tuned API server |
| 500-1000 nodes | 5 masters, dedicated etcd, tune scheduler |
| 1000-5000 nodes | Multiple control planes, API sharding |
| 5000+ nodes | Multiple clusters, federation |

#### 4. Namespace Organization at Scale

```bash
# In large organizations, use hierarchical namespaces
# tool: HNC (Hierarchical Namespace Controller)

# Create parent namespace
kubectl create namespace team-payments

# Create child namespaces
kubectl hns create production --namespace team-payments
kubectl hns create staging --namespace team-payments

# Policies (RBAC, NetworkPolicies) propagate to children automatically
```

---

# REAL WORLD PROJECTS

---

## Project 1 — Deploy Node.js Application

### Overview
Deploy a simple Node.js REST API with PostgreSQL database, Redis cache, and Ingress for external access.

### Project Structure

```
project-1/
├── app/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── k8s/
    ├── namespace.yaml
    ├── postgres-secret.yaml
    ├── postgres-deployment.yaml
    ├── postgres-service.yaml
    ├── redis-deployment.yaml
    ├── redis-service.yaml
    ├── api-configmap.yaml
    ├── api-deployment.yaml
    ├── api-service.yaml
    └── api-ingress.yaml
```

### Complete YAML

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: node-app
---
# postgres-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
  namespace: node-app
type: Opaque
stringData:
  POSTGRES_USER: appuser
  POSTGRES_PASSWORD: supersecretpassword
  POSTGRES_DB: myapp
---
# postgres-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        envFrom:
        - secretRef:
            name: postgres-credentials
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
          subPath: postgres
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        readinessProbe:
          exec:
            command: ['pg_isready', '-U', 'appuser', '-d', 'myapp']
          initialDelaySeconds: 10
          periodSeconds: 5
      volumes:
      - name: postgres-data
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: node-app
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 10Gi
---
# postgres-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: node-app
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
---
# redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: node-app
spec:
  replicas: 1
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
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:
          exec:
            command: ['redis-cli', 'ping']
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: node-app
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
---
# api-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
  namespace: node-app
data:
  DATABASE_HOST: postgres
  DATABASE_PORT: "5432"
  DATABASE_NAME: myapp
  REDIS_URL: redis://redis:6379
  PORT: "3000"
  NODE_ENV: production
---
# api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: node-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      initContainers:
      - name: wait-for-postgres
        image: busybox:1.35
        command: ['sh', '-c',
          'until nc -z postgres 5432; do echo "waiting for postgres"; sleep 2; done']
      containers:
      - name: api
        image: myrepo/node-api:1.0.0
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: api-config
        env:
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: postgres-credentials
              key: POSTGRES_USER
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-credentials
              key: POSTGRES_PASSWORD
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 15
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: node-app
spec:
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 3000
---
# api-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: node-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
```

### Deployment Steps

```bash
# Deploy everything
kubectl apply -f k8s/

# Watch everything come up
kubectl get all -n node-app -w

# Check logs
kubectl logs deployment/api -n node-app

# Test the API
kubectl port-forward service/api 3000:80 -n node-app
curl http://localhost:3000/health
```

---

## Project 2 — Full Stack Application

Deploy a React frontend + Node.js API + PostgreSQL + Redis with HPA and monitoring.

```yaml
# Frontend deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: fullstack-app
spec:
  replicas: 2
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
        image: myrepo/react-app:1.0.0
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
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: fullstack-app
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
---
# Ingress for routing frontend vs API
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fullstack-ingress
  namespace: fullstack-app
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
---
# HPA for API
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: fullstack-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Project 3 — Microservices Architecture

Deploy a full microservices app with API Gateway, multiple services, and message queue.

```yaml
# API Gateway
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: microservices
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: myrepo/api-gateway:1.0
        env:
        - name: USER_SERVICE_URL
          value: http://user-service:8080
        - name: ORDER_SERVICE_URL
          value: http://order-service:8080
        - name: PRODUCT_SERVICE_URL
          value: http://product-service:8080
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
  namespace: microservices
spec:
  selector:
    app: api-gateway
  ports:
  - port: 80
    targetPort: 3000
---
# User Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: microservices
spec:
  replicas: 2
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: myrepo/user-service:1.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservices
spec:
  selector:
    app: user-service
  ports:
  - port: 8080
    targetPort: 8080
# (Repeat pattern for order-service, product-service, notification-service...)
```

---

## Project 4 — Production Kubernetes Setup

Complete production-ready cluster setup guide.

```bash
#!/bin/bash
# Production Kubernetes Setup Script

# 1. Install ingress-nginx
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.replicaCount=3 \
  --set controller.nodeSelector."node-role"=ingress \
  --set controller.resources.requests.memory=256Mi \
  --set controller.resources.limits.memory=512Mi

# 2. Install cert-manager
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true \
  --set replicaCount=3

# 3. Install monitoring stack
helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=100Gi \
  --set grafana.persistence.enabled=true \
  --set grafana.persistence.size=10Gi \
  --set alertmanager.alertmanagerSpec.storage.volumeClaimTemplate.spec.resources.requests.storage=10Gi

# 4. Install logging stack (Loki)
helm upgrade --install loki grafana/loki-stack \
  --namespace logging \
  --create-namespace \
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=50Gi \
  --set promtail.enabled=true

# 5. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 6. Set up external-dns
helm upgrade --install external-dns bitnami/external-dns \
  --namespace kube-system \
  --set provider=aws \
  --set aws.region=us-east-1

# 7. Set up cluster autoscaler (AWS example)
helm upgrade --install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=my-cluster \
  --set awsRegion=us-east-1

echo "Production cluster setup complete!"
```

---

# KUBERNETES DEBUGGING HANDBOOK

---

## Systematic Debugging Guide

### Level 1 — Pod Issues

```bash
# ============================================================
# STEP 1: What is the pod status?
# ============================================================
kubectl get pod <pod-name> -n <namespace>

# STATUS: Pending
# → Scheduling problem
kubectl describe pod <pod-name> | tail -30  # Check Events
# Common causes: Insufficient resources, no matching nodes, PVC not bound

# STATUS: ContainerCreating
# → Container not started yet
kubectl describe pod <pod-name> | grep -A10 "Events:"
# Common causes: Image pull failure, volume mount failure

# STATUS: CrashLoopBackOff
# → Container starts but crashes
kubectl logs <pod-name> --previous  # Logs from before crash
kubectl describe pod <pod-name> | grep "Exit Code\|Reason\|Last State"

# STATUS: OOMKilled
# → Out of memory
# Solution: Increase memory limit or fix memory leak

# STATUS: Error
# → Container exited with non-zero code
kubectl logs <pod-name>  # Check application error

# STATUS: ImagePullBackOff
# → Cannot pull container image
kubectl describe pod <pod-name> | grep -A5 "Failed to pull\|Error"
# Fix: check image name, tag, registry credentials

# ============================================================
# STEP 2: Check events
# ============================================================
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
kubectl describe pod <pod-name> | grep -A50 "Events:"

# ============================================================
# STEP 3: Check logs
# ============================================================
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous  # If crashed
kubectl logs <pod-name> -n <namespace> -f          # Follow
kubectl logs <pod-name> -n <namespace> --tail=200  # Last 200 lines
```

### Level 2 — Networking Issues

```bash
# ============================================================
# Service not reachable?
# ============================================================

# 1. Does service exist?
kubectl get service <service-name> -n <namespace>

# 2. Does service have endpoints?
kubectl get endpoints <service-name> -n <namespace>
# If no endpoints: label selector mismatch!

# 3. Check service selector vs pod labels
kubectl get service <service-name> -o yaml | grep -A5 "selector:"
kubectl get pods --show-labels | grep <label-value>

# 4. Test from inside the cluster
kubectl run test-pod --image=busybox --rm -it -- \
  wget -O- http://<service-name>.<namespace>.svc.cluster.local

# 5. DNS resolution test
kubectl run test-pod --image=busybox --rm -it -- \
  nslookup <service-name>.<namespace>.svc.cluster.local

# 6. Check NetworkPolicy (might be blocking traffic)
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <policy-name> -n <namespace>
```

### Level 3 — Storage Issues

```bash
# ============================================================
# PVC not bound?
# ============================================================
kubectl get pvc -n <namespace>
kubectl describe pvc <pvc-name> -n <namespace>

# If Pending:
# - No PV available with matching size/accessMode/storageClass
# - StorageClass provisioner not working

kubectl get storageclass
kubectl get pv  # Are any PVs available?

# Check provisioner logs (for dynamic provisioning)
kubectl logs -l app=ebs-csi-controller -n kube-system

# ============================================================
# Volume mount failing?
# ============================================================
kubectl describe pod <pod-name> | grep -A5 "Mount failed\|Volume"
# Common: wrong secret/configmap name, PVC not bound
```

### Level 4 — Resource Issues

```bash
# ============================================================
# Node under pressure?
# ============================================================
kubectl describe node <node-name> | grep -A10 "Conditions:\|Allocated resources:"

# High CPU/memory? Which pods are consuming most?
kubectl top pods -A --sort-by=memory
kubectl top pods -A --sort-by=cpu

# Node disk pressure
kubectl describe node <node-name> | grep DiskPressure
# Solution: Clean up unused images, increase disk size

# ============================================================
# Pod evicted?
# ============================================================
kubectl get pods -A | grep Evicted
kubectl describe pod <evicted-pod>
# Common: Node ran out of memory, pod evicted based on QoS class
# BestEffort pods evicted first, then Burstable, then Guaranteed
```

---

# PRODUCTION FOLDER STRUCTURE

---

## Kubernetes Repository Layout

```
/k8s-infrastructure/         ← Infrastructure-as-Code Repository
│
├── /clusters/               ← Cluster-specific configurations
│   ├── /production/
│   │   ├── cluster-config.yaml
│   │   └── kustomization.yaml
│   └── /staging/
│       ├── cluster-config.yaml
│       └── kustomization.yaml
│
├── /platform/               ← Platform services (run by SRE/Platform team)
│   ├── /ingress-nginx/
│   │   ├── helmrelease.yaml
│   │   └── values-production.yaml
│   ├── /cert-manager/
│   │   ├── helmrelease.yaml
│   │   └── cluster-issuer.yaml
│   ├── /monitoring/
│   │   ├── helmrelease.yaml
│   │   ├── values-production.yaml
│   │   └── /alerts/
│   │       ├── node-alerts.yaml
│   │       └── app-alerts.yaml
│   ├── /logging/
│   │   ├── helmrelease.yaml
│   │   └── values-production.yaml
│   └── /argocd/
│       ├── install.yaml
│       └── /apps/
│           ├── production-apps.yaml
│           └── staging-apps.yaml
│
├── /apps/                   ← Application manifests (per team/app)
│   ├── /my-app/
│   │   ├── /base/           ← Common manifests
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── ingress.yaml
│   │   │   ├── hpa.yaml
│   │   │   └── kustomization.yaml
│   │   └── /overlays/       ← Environment overrides
│   │       ├── /production/
│   │       │   ├── kustomization.yaml
│   │       │   ├── replicas-patch.yaml
│   │       │   └── resources-patch.yaml
│   │       └── /staging/
│   │           ├── kustomization.yaml
│   │           └── replicas-patch.yaml
│   └── /payments-service/
│       ├── /base/
│       └── /overlays/
│
└── /rbac/                   ← Cluster-wide RBAC
    ├── /roles/
    │   ├── developer-role.yaml
    │   └── sre-role.yaml
    └── /bindings/
        ├── team-alpha-binding.yaml
        └── team-beta-binding.yaml
```

## Application Helm Chart Structure

```
/my-app-helm-chart/
├── Chart.yaml               ← Chart metadata
├── values.yaml              ← Default values
├── values-staging.yaml      ← Staging overrides
├── values-production.yaml   ← Production overrides
├── /templates/
│   ├── _helpers.tpl         ← Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml             ← PodDisruptionBudget
│   ├── serviceaccount.yaml
│   ├── rbac.yaml
│   └── NOTES.txt            ← Post-install instructions
└── /charts/                 ← Chart dependencies
    └── postgresql/          ← Sub-chart
```

## Kustomize-Based Overlay Pattern

```yaml
# apps/my-app/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
- ingress.yaml
- hpa.yaml
commonLabels:
  app: my-app
  managed-by: kustomize
---
# apps/my-app/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- ../../base
namespace: production
patches:
- path: replicas-patch.yaml
- path: resources-patch.yaml
images:
- name: my-app
  newTag: "1.5.2"  # Pin to specific version in production
---
# apps/my-app/overlays/production/replicas-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5
```

```bash
# Apply with Kustomize
kubectl apply -k apps/my-app/overlays/production/

# Preview what would be applied
kubectl kustomize apps/my-app/overlays/production/
```

---

# KUBERNETES MASTERY CHECKLIST

---

## Level 1 — Beginner
- ✔ Understand what Kubernetes is and why it exists
- ✔ Know the difference between control plane and worker nodes
- ✔ Can install Minikube and start a local cluster
- ✔ Can navigate kubectl (get, describe, apply, delete, logs, exec)
- ✔ Can create and manage Pods
- ✔ Understand Pod lifecycle states

## Level 2 — Intermediate
- ✔ Deployments — create, update, rollback
- ✔ ReplicaSets — understand their role
- ✔ Services — ClusterIP, NodePort, LoadBalancer
- ✔ DNS in Kubernetes — how services are discovered
- ✔ Labels and Selectors — how resources relate
- ✔ ConfigMaps — injecting configuration
- ✔ Secrets — storing sensitive data securely
- ✔ Namespaces — organizing workloads

## Level 3 — Advanced
- ✔ Volumes and emptyDir for temporary storage
- ✔ PersistentVolumes and PersistentVolumeClaims
- ✔ StorageClasses for dynamic provisioning
- ✔ Ingress for HTTP routing and TLS
- ✔ Networking model and CNI plugins
- ✔ Resource Requests and Limits
- ✔ QoS classes (Guaranteed, Burstable, BestEffort)
- ✔ Liveness, Readiness, and Startup probes

## Level 4 — Production
- ✔ Rolling updates with zero downtime
- ✔ HPA — Horizontal Pod Autoscaler
- ✔ Jobs and CronJobs
- ✔ RBAC — Roles, ClusterRoles, Bindings
- ✔ Pod Security Standards
- ✔ SecurityContext for hardened containers
- ✔ Network Policies for traffic control
- ✔ PodDisruptionBudgets

## Level 5 — Expert
- ✔ Helm — installing, writing, managing charts
- ✔ Prometheus + Grafana monitoring stack
- ✔ Centralized logging with Loki
- ✔ Systematic debugging methodology
- ✔ Production architecture design
- ✔ Taints, Tolerations, Node Affinity
- ✔ Kubernetes API and CRDs
- ✔ etcd backup and restore

## Level 6 — Master
- ✔ iptables/IPVS networking internals
- ✔ CoreDNS configuration
- ✔ Service Mesh with Istio
- ✔ Performance optimization at scale
- ✔ CI/CD pipeline with GitHub Actions
- ✔ GitOps with ArgoCD
- ✔ Multi-cluster management
- ✔ Kubernetes at 1000+ nodes
- ✔ Custom controllers and operators
- ✔ API Priority and Fairness

---

## Quick Reference Card

```
MOST USED COMMANDS (memorize these):

kubectl get pods -A                         # All pods everywhere
kubectl get pods -n <ns> -w                 # Watch pods
kubectl describe pod <name> -n <ns>         # Debug a pod
kubectl logs <pod> -f --tail=100            # Follow logs
kubectl exec -it <pod> -- bash              # Shell into pod
kubectl apply -f <file>                     # Apply manifest
kubectl delete -f <file>                    # Delete from manifest
kubectl rollout status deploy/<name>        # Rollout progress
kubectl rollout undo deploy/<name>          # Rollback
kubectl scale deploy/<name> --replicas=5    # Scale
kubectl top pods -A --sort-by=memory        # Resource usage
kubectl get events -A --sort-by='.lastTimestamp' # All events
kubectl port-forward svc/<name> 8080:80    # Local access
kubectl auth can-i get pods                 # Check permissions
kubectl diff -f <file>                      # Preview changes
```

---

## Learning Path Summary

```
Week 1-2:  Level 1 — Chapters 1-5 (Fundamentals)
Week 3-4:  Level 2 — Chapters 6-12 (Core Resources)
Week 5-6:  Level 3 — Chapters 13-19 (Storage & Networking)
Week 7-8:  Level 4 — Chapters 20-25 (Production Readiness)
Week 9-10: Level 5 — Chapters 26-32 (Expert Operations)
Week 11+:  Level 6 — Chapters 33-39 (Master Level)

Practice:  Projects 1-4 (Build real systems)
Certify:   CKA (Certified Kubernetes Administrator)
           CKAD (Certified Kubernetes Application Developer)
           CKS (Certified Kubernetes Security Specialist)
```

---

*This guide was created as a complete Kubernetes learning resource — from your first `kubectl get pods` to running Kubernetes at scale in production. The journey from beginner to master takes time and practice. Build things. Break things. Fix things. That is how you learn Kubernetes.*

---

**End of Kubernetes Mastery Guide**
