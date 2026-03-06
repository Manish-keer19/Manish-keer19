# The Ultimate Kubernetes Mastery Guide: Beginner to Production

_A comprehensive 15-module handbook crafted by a Senior DevOps Engineer with 15+ years of experience managing large-scale, production-grade clusters at top tech companies._

## Table of Contents

1. [Module 1: Kubernetes Fundamentals](#module-1-kubernetes-fundamentals)
2. [Module 2: Kubernetes Setup](#module-2-kubernetes-setup)
3. [Module 3: Core Kubernetes Objects](#module-3-core-kubernetes-objects)
4. [Module 4: Networking](#module-4-networking)
5. [Module 5: Storage](#module-5-storage)
   _(Further modules are continued below)_

---

## Module 1: Kubernetes Fundamentals

### What is Kubernetes

Kubernetes (often abbreviated as "K8s") is an open-source container orchestration platform initially developed by Google. In the production world, running a single container is easy. Running 10,000 containers across 500 servers securely, with automatic failover, load balancing, and scaling, is hard. Kubernetes solves this problem. It is the operating system for the cloud native world.

### Why Kubernetes was created

Google ran billions of containers a week using their internal systems "Borg" and "Omega". They open-sourced the underlying concepts as Kubernetes in 2014. Before Kubernetes, DevOps teams relied on complex custom bash scripts, configuration management tools (Chef, Puppet), and manual interventions. K8s was created to:

- Abstract away the underlying hardware infrastructure.
- Provide self-healing applications (if a node dies, move the container).
- Automate rollouts and rollbacks without downtime.
- Scale out easily when web traffic spikes.

### Containers vs Virtual Machines

- **Virtual Machines (VMs):** Emulate entire hardware environments. They run a full Guest OS (Linux/Windows) on top of a hypervisor. This makes them heavy, slow to start, and resource-intensive.
- **Containers:** Share the host operating system's kernel. They are essentially process-level isolation features (using Linux Namespaces and Cgroups). They start in milliseconds, consume almost zero overhead, and are highly portable.

### Kubernetes Architecture Overview

Kubernetes uses a traditional Client-Server / Master-Worker architecture. However, we refer to the Master as the **Control Plane** and the Workers as **Worker Nodes**.

```text
            +-------------------+
            |      kubectl      | (You, configuring the cluster)
            +---------+---------+
                      |
                      v
             +--------+--------+
             |    API Server   | (The Brain/Gateway)
             +--------+--------+
                      |
    +-----------------+-----------------+
    |                                   |
    v                                   v
+---------------+                   +---------------+
|   Scheduler   |                   | ControllerMgr |
+-------+-------+                   +-------+-------+
|                                   |
v                                   v
+-------------------+
|       etcd        | (Key-Value Datastore)
+---------+---------+
|
-----------------------------------------
|                    |                  |
v                    v                  v
+-----------+        +-----------+      +-----------+
| Worker 1  |        | Worker 2  |      | Worker 3  |
| kubelet   |        | kubelet   |      | kubelet   |
| kubeproxy |        | kubeproxy |      | kubeproxy |
+-----------+        +-----------+      +-----------+
```

### Control Plane Components

The Control Plane is responsible for global decisions (like scheduling) and detecting/responding to cluster events.

1. **kube-apiserver:** The front-end of the control plane. It exposes the Kubernetes API. Everything, including other control plane components, talks to the API server.
2. **etcd:** A consistent and highly-available key-value store. It strictly holds the cluster state and configuration. If you lose `etcd` without a backup, your cluster is gone.
3. **kube-scheduler:** Watches for newly created Pods with no assigned node, and selects a node for them to run on based on resource requirements, affinity rules, and constraints.
4. **kube-controller-manager:** Runs controller processes (like Node Controller, Job Controller, EndpointSlice controller). It ensures the actual state matches the desired state.
5. **cloud-controller-manager:** Links your cluster into a cloud provider's API (e.g., AWS, GCP, Azure).

### Worker Node Components

Nodes are the actual servers where your application workloads run.

1. **kubelet:** An agent running on each node. It communicates with the API server and ensures that containers are running in a Pod by talking to the container runtime.
2. **kube-proxy:** A network proxy that runs on each node, implementing part of the Kubernetes Service concept. It maintains network rules via `iptables` or `IPVS` to route traffic to pods.
3. **Container Runtime:** The software responsible for running containers (e.g., containerd, CRI-O). Kubernetes deprecated Docker as a runtime, shifting to compliant CRI (Container Runtime Interface) technologies.

---

## Module 2: Kubernetes Setup

In the production world, we provision clusters using Infrastructure as Code (e.g., Terraform on EKS/GKE). For learning, we need local development clusters.

### What is Minikube?

Minikube is a local Kubernetes environment for developers. It runs a small Kubernetes cluster inside a VM or container on your laptop.

### Install Kubernetes with Minikube

Minikube runs a single-node cluster inside a VM, Docker container, or bare-metal environment on your local machine.

1. Install Docker Desktop.
2. Install Minikube:
   - Mac: `brew install minikube`
   - Linux: `curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube`
   - Windows: Use `winget install minikube`
3. Start the cluster:
   ```bash
   minikube start --driver=docker
   ```

### Install Kubernetes with kubeadm (Production-like Local)

`kubeadm` is the standard tool for bootstrapping clusters.

```bash
# On Master Node
kubeadm init --pod-network-cidr=10.244.0.0/16

# Configure standard kubectl access
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install a Pod Network plugin (Flannel)
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Install kubectl

The `kubectl` command-line tool is your steering wheel for Kubernetes.

```bash
# standard curl installation
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Cluster verification

Check if your setup is functional:

```bash
# Check node status
kubectl get nodes -o wide

# Check cluster info
kubectl cluster-info

# List all system pods
kubectl get pods -n kube-system
```

### First Pod Deployment

Let's deploy a simple NGINX web server.

```bash
# Imperative command
kubectl run my-first-nginx --image=nginx:latest --port=80

# Check status
kubectl get pods
```

---

## Module 3: Core Kubernetes Objects

Rather than using imperative commands, serious deployments use declarative YAML manifests.

### Pods

The smallest deployable computing unit. A pod encapsulates one or more containers (e.g., your app container + a sidecar proxy).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: frontend
spec:
  containers:
    - name: nginx
      image: nginx:1.23
      ports:
        - containerPort: 80
```

Deploy via: `kubectl apply -f pod.yaml`

### ReplicaSets

Ensures a specified number of pod replicas are running at any given time for high availability.
_(Note: Never create a ReplicaSet directly; always use Deployments)._

### Deployments

Deployments manage ReplicaSets and provide declarative updates to Pods (e.g., rolling updates and rollbacks).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: app
          image: myapp:v1.0
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

**Scaling rule:** `kubectl scale deployment web-app --replicas=5`

### Services

Pods are ephemeral. They die and get new IP addresses. A Service provides a single, stable IP address and load balances traffic across a set of pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### Namespaces

Logical boundaries within a physical cluster. Good for multi-tenant environments (e.g., `dev`, `staging`, `prod`).

```bash
kubectl create namespace staging
kubectl get pods -n staging
```

### Labels and Selectors

Labels are key/value pairs attached to objects. Selectors are how you query objects based on labels. Service uses selectors to figure out which Pods to route traffic to.
_(Pro Tip: In production, standardize label schema: `app.kubernetes.io/name`, `app.kubernetes.io/version`, etc.)_

### ConfigMaps and Secrets

- **ConfigMap:** Store non-confidential configuration data (environment variables, settings config files).
- **Secret:** Store sensitive data securely (passwords, tokens, SSH keys). Secrets are base64 encoded by default (Make sure `etcd` encryption is turned on in production!).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  password: cGFzc3dvcmQxMjM= # "password123" in base64
```

---

## Module 4: Networking

Kubernetes networking assumes that every Pod has a unique IP and can communicate with all other Pods without NAT.

### Cluster Networking & Pod Communication

CNI (Container Network Interface) plugins (like Calico, Cilium, Flannel) implement this requirement. When Pod A (10.1.0.5) talks to Pod B (10.2.0.6), the CNI handles the packet routing seamlessly whether they are on the same worker node or across the world.

### Service Types

1. **ClusterIP (Default):** Exposes the service on an internal-only cluster IP.
2. **NodePort:** Opens a static port (between 30000-32767) on EVERY worker node. Rarely used in modern production apart from legacy setups.
3. **LoadBalancer:** Provisions a cloud load balancer (AWS ALB, GCP TCP Loadbalancer) that points to your NodePorts/Pods.
4. **ExternalName:** Maps a service to a DNS name.

### Ingress

While LoadBalancers are layer 4, Ingress is layer 7 (HTTP/HTTPS). An Ingress Controller (like NGINX Ingress Controller or Traefik) sits inside the cluster, handling SSL termination, path-based routing, and hostname routing.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: www.myproductionapp.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
```

### Network Policies

By default, all pods can talk to all pods. Network Policies act as firewall rules for pods.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

**Best Practice:** Implement zero-trust networking by starting with a default-deny policy in every namespace.

---

## Module 5: Storage

Containers are ephemeral. If a container crashes, its local data is lost. We need persistent storage.

### Volumes

Kubernetes Volumes outlive containers, but they are tied to a Pod's lifecycle. Example: `emptyDir` (cleared when pod dies), `hostPath` (mounts a file/directory from the host node).

### Persistent Volumes (PV) & PVCs

- **PV (Persistent Volume):** A piece of storage provisioned in the cluster (e.g., an AWS EBS volume or NFS drive).
- **PVC (Persistent Volume Claim):** A request for storage by a user/developer.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### Storage Classes

Instead of admins manually creating PVs, StorageClasses allow for _dynamic volume provisioning_. When a user creates a PVC asking for "fast-storage", the cloud provider automatically spins up a real SSD and attaches it.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
```

### StatefulSets

Deployments are stateless and uniform. StatefulSets are used for stateful applications like Databases (Kafka, PostgreSQL, MongoDB, Elasticsearch).

- Pods are given a sticky, unique identity (e.g., `mysql-0`, `mysql-1`).
- Pods are deployed symmetrically in sequential order.
- Each Pod automatically gets its own PVC derived from a VolumeClaimTemplate.

_End of Part 1. (Continues with Module 6)_

## Module 6: Scaling and High Availability

Production environments require systems that adapt to traffic patterns automatically without human intervention.

### Horizontal Pod Autoscaler (HPA)

The HPA scales the number of Pod replicas up or down based on observed CPU utilization or custom metrics (e.g., requests per second).

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

_Note: HPA requires the Metrics Server to be installed in the cluster._

### Cluster Autoscaler / Karpenter

While HPA adds more Pods, what happens when your physical nodes run out of CPU/Memory capacity? The pods sit in a `Pending` state.
The **Cluster Autoscaler** interacts with your cloud provider to spin up brand-new EC2/GCE instances directly and adds them to the cluster.
Modern alternative: **Karpenter** (AWS) provisions purely customized nodes instantly, reducing launch times from minutes to seconds.

### Self Healing

Kubernetes inherently self-heals by restarting crashed containers. We manage application health using Probes.

1. **Liveness Probe:** Checks if the app is dead. If fails, Kubelet restarts the container.
2. **Readiness Probe:** Checks if app is ready to take traffic. If fails, it's removed from Service endpoints temporarily.
3. **Startup Probe:** Validates that legacy, slow applications have finished starting up.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

### Rolling Updates and Rollbacks

If you update a deployment's image, K8s performs a Rolling Update—it spins up a new pod, waits for it to become ready, and then kills an old pod. This ensures zero downtime.

```bash
# Trigger an update
kubectl set image deployment/frontend www=nginx:1.24.0

# Watch rollout status
kubectl rollout status deployment/frontend

# Oops, the new version has bugs! Rollback instantly:
kubectl rollout undo deployment/frontend
```

---

## Module 7: Security

Kubernetes defaults prioritize ease-of-use over security. In production, we must harden it.

### RBAC (Role-Based Access Control)

Avoid giving everyone cluster-admin rights! RBAC uses four core concepts:

1. **Role:** Permissions within a specific namespace (e.g., can "list" and "get" pods).
2. **ClusterRole:** Permissions across the entire cluster (e.g., node permissions).
3. **RoleBinding:** Assigns a Role to a User, Group, or ServiceAccount in a namespace.
4. **ClusterRoleBinding:** Assigns a ClusterRole globally.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "watch", "list"]
```

### Service Accounts

A standard user maps to a human. A **ServiceAccount** gives an identity to a Pod, allowing the application running inside it to authenticate and make external API requests (e.g., reading secrets, querying the AWS API or Kubernetes API).

### Pod Security (formerly Pod Security Policies, now Pod Security Admission)

Stop containers from running as root!

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

### Secret Management in Production

Do NOT store raw Kubernetes Secrets in Git. They are base64 encoded and easily crackable.

- **External Secrets Operator:** syncs AWS Secrets Manager / Azure Key Vault to Kubernetes.
- **HashiCorp Vault:** Direct dynamic injection into applications.
- **SealedSecrets/SOPS:** Encrypts YAMLs utilizing asymmetric cryptography before committing to Git.

---

## Module 8: Monitoring and Logging

If a Pod crashes, logs disappear. Active monitoring and persistent logging architectures are mandatory.

### Metrics Server

A cluster-wide aggregator of resource usage data. Required for the `kubectl top node` and `kubectl top pod` commands.

### The Production Monitoring Stack: Prometheus + Grafana

- **Prometheus:** A time-series database that "scrapes" metrics via HTTP from your nodes, deployments, and databases.
- **Grafana:** The dashboarding tool that turns Prometheus data into beautiful visualization panels.
  _Deployment best practice: Use the `kube-prometheus-stack` Helm chart._

### Centralized Logging: EFK/ELK Stack vs Loki

In a large cluster, node-level log-rotation will delete your logs. We use agents (Fluentbit / Promtail) running as `DaemonSets` on every node to scoop up all logs and ship them.

- **ELK:** Elasticsearch (Store), Logstash/Fluentd (Agent), Kibana (UI). Very heavy, suitable for huge security analysis.
- **PLG (Prometheus, Loki, Grafana):** Loki acts like Prometheus for logs. Highly scalable, lightweight, indexless (it only indexes labels), and heavily preferred in modern cloud-native architectures.

---

## Module 9: DevOps Integration

How the code gets from a developer's laptop to running smoothly in Kubernetes.

### CI/CD Workflow (Docker + Kubernetes)

1. **Developer** commits code to GitHub.
2. **CI Server** (GitHub Actions, Jenkins, or GitLab CI) runs Unit Tests.
3. CI Server builds a Docker Image and tags it with the Git Hash (e.g., `app:a1b2c3d`).
4. CI Server pushes the image to a Container Registry (e.g., AWS ECR, Docker Hub).
5. CI updates the image tag in the Kubernetes deployment YAML file.

### GitOps (The Modern CD Paradigm)

Rather than having Jenkins push to the cluster (Push approach), GitOps utilizes a Pull approach. An operator sits _inside_ the cluster and constantly observes the GitHub repository YAMLs. State is enforced declaratively.

#### ArgoCD

ArgoCD is the premier GitOps tool.

- You install ArgoCD into the cluster.
- You configure ArgoCD to watch your "Config" Git repo.
- Any time someone merges a Pull Request containing a new image tag, ArgoCD automatically syncs and deploys the new pods.
- No CI server ever queries Kubernetes or needs cluster credentials. High security!

---

## Module 10: Production Ready Kubernetes

Architecting for 99.99% Node, Zone, and Regional Failures.

### Multi-Node Cluster Architecture

Never run a production cluster in one Availability Zone (AZ). Your node topology must span across at least three AZs (`us-east-1a`, `us-east-1b`, `us-east-1c`) to ensure application survival during datacenter outages.

### High Availability Control Plane

In managed services (EKS, GKE), the provider handles this. If building on raw VMs:

- Run 3 or 5 `etcd` node replicas. (Always an odd number for quorum consensus algorithms like Raft).
- Place API Server Load Balancers in front of multi-master nodes.

### Backup and Disaster Recovery

Use **Velero**. Velero allows DevOps engineers to back up entire Kubernetes namespaces, PVs (via volume snapshots), and deployments securely into an AWS S3 bucket.

```bash
velero backup create production-backup-2026 --include-namespaces=production
# In case of cluster destruction:
velero restore create --from-backup production-backup-2026
```

### Advanced Deployment Strategies

1. **Blue-Green Deployment:** Run two identical production environments (Blue and Green). Traffic currently goes to Blue. You deploy the new code to Green, verify it works, then switch the load balancer to point to Green. Almost zero downtime rollback.
2. **Canary Deployment:** Route 5% of your live user traffic to the newly deployed pods. If you see elevated 500 HTTP errors or high latency in Grafana, abort. If metrics are healthy, gradually increase to 20%, 50%, 100%. Usually coordinated by Argo Rollouts or Istio.

_End of Part 2. (Continues with Module 11)_

## Module 11: Troubleshooting

In a live production environment, debugging quickly is the difference between a minor hiccup and a major outage.

### Debugging Pods

Check pod statuses:

```bash
kubectl get pods --all-namespaces
kubectl describe pod <pod-name> -n <namespace>
```

Look at the `Events:` section at the very bottom of the output—it will tell you exactly why a pod isn't starting (e.g., `FailedScheduling`, `OOMKilled`, `ImagePullBackOff`).

### Checking Logs

```bash
# View current logs
kubectl logs pod-name -n namespace

# Follow logs in real-time
kubectl logs -f pod-name

# Logs for a specific container in a multi-container pod
kubectl logs pod-name -c container-name

# Get logs from a crashed pod (previous incarnation)
kubectl logs pod-name --previous
```

### The `kubectl debug` Command

If a pod uses a lightweight "distroless" image containing no shell (`sh` or `bash`) or utilities (`curl`), you can attach an ephemeral debug container:

```bash
kubectl debug -it pod-name --image=busybox:1.28 --target=app-container
```

### CrashLoopBackOff Fixes

**Cause:** The container starts, crashes immediately, and Kubernetes attempts to restart it exponentially.
**Fix Strategy:**

1. Run `kubectl logs pod-name --previous`. It is usually an application code exception or missing environment variable.
2. Check `kubectl describe pod` to ensure limits aren't restricting CPU/Memory excessively.
3. Validate database connections from within the cluster.

### Handling Node Failures

When a node stays `NotReady`, its pods show status `Unknown` or `NodeLost`.

```bash
kubectl describe node <node-name>
```

Look for `MemoryPressure` or `DiskPressure`. Often, the solution is gracefully draining the node and replacing it.

```bash
kubectl cordon <node-name>  # Marks node as unschedulable
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

---

## Module 12: Real Production Architecture

### Enterprise Architecture Diagram

A standard SaaS Application deployed on Kubernetes.

```text
                                +-------------------+
                                |    [Internet]     |
                                |     End Users     |
                                +---------+---------+
                                          |
                                          v
                                +-------------------+
                                |    Cloud CDN/     |
                                |   WAF Firewall    |
                                +---------+---------+
                                          |
                                          v
                                +-------------------+
                                |   Load Balancer   |  (AWS ALB, GCP External LB)
                                +---------+---------+
                                          |
                                          v
      -------------------------------------------------------------------------
      |                    Kubernetes Cluster Boundary                        |
      |                                   |                                   |
      |                                   v                                   |
      |                         +-------------------+                         |
      |                         | Ingress Controller| (NGINX/Traefik routing  |
      |                         |  (TLS Termination)|  traffic by hostname )  |
      |                         +----+---------+----+                         |
      |                              |         |                              |
      |              /api/v1/users   |         | /frontend                    |
      |                              v         v                              |
      |                  +-------------+     +-------------+                  |
      |                  |   Backend   |     |   Frontend  |                  |
      |                  |   Service   |     |   Service   |                  |
      |                  +------+------+     +------+------+                  |
      |                         |                   |                         |
      |               +---------+-------+   +-------+---------+               |
      |               v                 v   v                 v               |
      |         +-----------+     +-----------+         +-----------+         |
      |         |  Pod API  |     |  Pod API  |         |  Pod UI   |         |
      |         | Replica 1 |     | Replica 2 |         | Replica 1 |         |
      |         +-----+-----+     +-----+-----+         +-----------+         |
      |               |                 |                                     |
      -------------------------------------------------------------------------
                      |                 |
                      v                 v
                +-----------------------------+
                |   Managed Cloud Database    | (Amazon RDS / Cloud SQL)
                |     PostgreSQL / Redis      |
                +-----------------------------+
```

---

## Module 13: DevOps Best Practices

Rules written in blood and 3AM production outages.

- **Namespace Strategy:** Never use the `default` namespace. Use namespaces strictly to segregate environments (e.g., `prod-web`, `prod-data`, `monitoring`) or tenant squads.
- **Resource Requests and Limits:** Every container MUST have memory and CPU `requests` and `limits` defined. If omitted, a single memory leak in one app will trigger OOM kills for the entire worker node, crushing innocent neighbor pods.
  - _Best Practice:_ Memory request = limit. CPU limit is optional but highly recommended if unthrottled CPU ruins latency predictability.
- **Always specify Image Tags:** Never deploy `image: myapp:latest`. If the container restarts, it might pull a newer version silently, breaking production. Pin strict tags like `image: myapp:v2.4.15`.
- **Readiness Probes:** Always implement Readiness Probes. Without them, Kubernetes will route traffic to your application the millisecond the process starts, even if it is still establishing database connections resulting in 502 Bad Gateways for users.
- **Affinity & Anti-Affinity:** Use node selectors and pod anti-affinity to ensure Replicas 1, 2, and 3 are never scheduled on the exact same physical server to survive underlying hardware deaths.

---

## Module 14: Hands-on Labs

_Practice these to build muscle memory._

**Lab 1: The Rolling Update**

1. Create a deployment with 3 replicas of `nginx:1.14` using `kubectl create`.
2. Map it to a NodePort service.
3. Access the IP in your browser.
4. Update the deployment to `nginx:1.16` using `kubectl set image`.
5. Watch the pods terminating and creating `kubectl get pods -w`.

**Lab 2: ConfigMaps & Secrets**

1. Create a `settings.txt` locally.
2. Run `kubectl create configmap web-config --from-file=settings.txt`.
3. Mount it as a Volume into a test Pod and view the contents by `kubectl exec -it pod-name -- cat /mnt/settings.txt`.

**Lab 3: Implementing High Availability**

1. Create a Deployment with 4 replicas.
2. Force delete one using `kubectl delete pod <pod-name>`.
3. Observe how quickly the deployment spins up a replacement to maintain the 4-replica state.

---

## Module 15: Kubernetes Interview Preparation

Be prepared for scenario-based problem solving.

**Top 5 Core Questions:**

1. **Explain the architecture of Kubernetes.** (Mention Control Plane: API, etcd, scheduler, CM / Data Plane: Kubelet, Kube-proxy).
2. **What is the difference between a Deployment and a StatefulSet?** (Deployment provides arbitrary, stateless replicas. StatefulSets provide ordered deployment, sticky network identities, and dedicated storage per replica).
3. **How does an Ingress differ from a LoadBalancer?** (Layer 7 HTTP vs Layer 4 TCP; Ingress lives in the cluster while LB is external cloud infrastructure).
4. **What is a DaemonSet?** (Ensures all, or some, Nodes run a copy of a Pod. Used for log collectors and monitoring agents).
5. **How does Kubernetes manage Container Networking?** (Every pod gets an IP; pods can talk without NAT; managed by CNI plugins).

**Scenario Based Question:**
_Interviewer: "A developer deployed an application, but they claim they cannot hit the endpoint. Where do you start debugging?"_
_Ans:_

1. Check if the Pod is running `kubectl get pods` - look for `CrashLoopBackOff` or `ImagePullBackOff`.
2. If Running, access logs: `kubectl logs pod-name`.
3. Verify the Readiness probe is passing (check `Endpoints` of the Service `kubectl describe svc`). If there are no endpoints, the service selector doesn't match the pod labels, or readiness probe failed.
4. Verify the Ingress rules route to the correct service name and port.

**Production Debugging Example:**
_Interviewer: "A worker node suddenly shows high memory pressure and pods are getting OOMKilled. What went wrong?"_
_Ans:_ A deployed pod is likely leaking memory and lacked a defined resource `limit`. The Kubelet triggers OOM kills natively to protect node stability. The fix is applying explicit `requests` and `limits` to all pod manifests, isolating greedy applications to their boundaries.

---

\_C
