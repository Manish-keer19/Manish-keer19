# 🚀 Complete DevOps Engineer Roadmap
> **By a Senior DevOps Engineer** — Everything you need to know, in order, with zero fluff.

---

## 📌 Table of Contents

1. [Phase 0 — Foundations (Before DevOps)](#phase-0)
2. [Phase 1 — Linux & Scripting](#phase-1)
3. [Phase 2 — Networking & Security Basics](#phase-2)
4. [Phase 3 — Version Control (Git)](#phase-3)
5. [Phase 4 — Programming / Scripting Languages](#phase-4)
6. [Phase 5 — Containers (Docker)](#phase-5)
7. [Phase 6 — Container Orchestration (Kubernetes)](#phase-6)
8. [Phase 7 — CI/CD Pipelines](#phase-7)
9. [Phase 8 — Infrastructure as Code (IaC)](#phase-8)
10. [Phase 9 — Cloud Platforms](#phase-9)
11. [Phase 10 — Monitoring, Logging & Observability](#phase-10)
12. [Phase 11 — Configuration Management](#phase-11)
13. [Phase 12 — Security in DevOps (DevSecOps)](#phase-12)
14. [Phase 13 — Advanced Topics](#phase-13)
15. [Soft Skills & Career Tips](#soft-skills)
16. [Learning Resources](#resources)
17. [Roadmap Timeline Summary](#timeline)

---

<a name="phase-0"></a>
## 🧱 Phase 0 — Foundations (Before DevOps)
> *Don't skip this. Everything else sits on top of it.*

### What is DevOps?
DevOps is a **culture + practice + toolset** that bridges the gap between Development (writing code) and Operations (deploying and maintaining it). The goal is to ship software faster, more reliably, and at scale.

**Core DevOps Principles:**
- Collaboration between Dev & Ops teams
- Automation of everything possible
- Continuous Improvement (CI/CD)
- Infrastructure as Code
- Monitoring & Feedback loops

### Computer Science Basics You Must Know
- How computers work (CPU, RAM, Disk, Network)
- What an Operating System does
- How processes, threads, and memory work
- What a filesystem is and how it's structured
- What a server is vs a client
- Basics of HTTP, HTTPS, DNS, TCP/IP

---

<a name="phase-1"></a>
## 🐧 Phase 1 — Linux & Shell Scripting
> *Linux is the heart of DevOps. Almost every server runs Linux.*

### Linux Fundamentals
- **Distributions**: Ubuntu, CentOS/RHEL, Amazon Linux, Debian
- **Filesystem Hierarchy**: `/etc`, `/var`, `/home`, `/opt`, `/usr`, `/tmp`, `/proc`, `/sys`
- **File Permissions**: `chmod`, `chown`, `chgrp` — understand `rwx` and octal notation (755, 644)
- **User & Group Management**: `useradd`, `usermod`, `passwd`, `sudo`, `/etc/passwd`, `/etc/sudoers`
- **Process Management**: `ps`, `top`, `htop`, `kill`, `pkill`, `nice`, `nohup`, `systemd`, `journalctl`
- **Package Management**: `apt` (Debian/Ubuntu), `yum`/`dnf` (RHEL/CentOS), `snap`, `rpm`
- **Disk & Storage**: `df`, `du`, `lsblk`, `fdisk`, `mount`, `fstab`, LVM basics
- **Cron Jobs**: `crontab -e`, cron syntax (`* * * * *`), `at` command
- **Log Files**: `/var/log/syslog`, `/var/log/auth.log`, `journalctl -f`, `tail -f`
- **Environment Variables**: `export`, `.bashrc`, `.bash_profile`, `.env` files
- **File Transfer**: `scp`, `rsync`, `sftp`
- **Text Processing**: `grep`, `awk`, `sed`, `cut`, `sort`, `uniq`, `wc`, `xargs`
- **Archiving**: `tar`, `gzip`, `zip`, `unzip`

### Shell Scripting (Bash)
```bash
# Variables
NAME="DevOps"
echo "Hello, $NAME"

# Conditionals
if [ "$ENV" == "prod" ]; then
  echo "Production!"
fi

# Loops
for server in web1 web2 web3; do
  echo "Deploying to $server"
done

# Functions
deploy() {
  echo "Deploying version $1"
}
deploy "v1.2.0"

# Exit codes & error handling
set -e          # Exit on error
set -o pipefail # Fail pipeline on any error
trap 'echo "Error on line $LINENO"' ERR
```

**What to build:** Write scripts to: automate backups, rotate logs, check disk usage and alert, auto-start services on failure.

### Essential Linux Tools
- `vim` / `nano` — text editors
- `tmux` / `screen` — terminal multiplexers
- `ssh` — remote access
- `curl` / `wget` — HTTP requests from terminal
- `netstat` / `ss` — network statistics
- `strace` / `lsof` — debugging running processes

---

<a name="phase-2"></a>
## 🌐 Phase 2 — Networking & Security Basics
> *You can't debug production issues without understanding networking.*

### Networking Concepts
- **OSI Model** — 7 layers, know at least L3 (Network), L4 (Transport), L7 (Application)
- **TCP vs UDP** — connection-oriented vs connectionless
- **IP Addressing** — IPv4, IPv6, CIDR notation (`192.168.1.0/24`), subnetting
- **DNS** — A, CNAME, MX, TXT, NS records; how resolution works; TTL
- **HTTP/HTTPS** — request/response cycle, status codes (200, 301, 400, 401, 403, 404, 500, 502, 503)
- **Load Balancing** — Round Robin, Least Connections, IP Hash; L4 vs L7 load balancers
- **Reverse Proxy** — Nginx, HAProxy — what they do and why
- **Firewalls** — `iptables`, `ufw`, security groups in cloud
- **VPN / VPC** — Virtual Private Networks; Virtual Private Cloud in AWS/GCP/Azure
- **CDN** — Content Delivery Networks (Cloudflare, AWS CloudFront)
- **SSL/TLS** — certificates, Let's Encrypt, certificate chains, HTTPS setup

### Web Servers
- **Nginx** — configuration, virtual hosts, reverse proxy setup, SSL termination
- **Apache** — `.htaccess`, virtual hosts, modules
- **Caddy** — auto HTTPS, simple config

### Networking Tools
```bash
ping google.com           # ICMP connectivity
traceroute google.com     # Trace network path
nslookup example.com      # DNS lookup
dig example.com A         # Detailed DNS query
curl -I https://site.com  # HTTP headers
netstat -tulpn            # Open ports & listening services
ss -tulpn                 # Modern alternative to netstat
tcpdump -i eth0 port 80   # Capture packets
nmap -sV 192.168.1.1      # Port scanning
```

---

<a name="phase-3"></a>
## 🔀 Phase 3 — Version Control (Git)
> *Git is non-negotiable. You use it every single day.*

### Core Git Concepts
- **Repository** — local vs remote (GitHub, GitLab, Bitbucket)
- **Branching** — `main`/`master`, feature branches, hotfix branches
- **Merge vs Rebase** — when to use each; interactive rebase (`git rebase -i`)
- **Pull Requests / Merge Requests** — code review workflow
- **Tags** — semantic versioning (`v1.0.0`)
- **Stash** — `git stash`, `git stash pop`
- **Cherry-pick** — apply specific commits to another branch
- **Hooks** — pre-commit, pre-push, post-receive hooks

### Git Workflows to Know
- **GitFlow** — `develop`, `feature/*`, `release/*`, `hotfix/*`
- **Trunk-based Development** — short-lived branches, frequent merges to main
- **GitHub Flow** — simple: branch → PR → merge

### Git Commands Cheatsheet
```bash
git init
git clone <url>
git add .
git commit -m "feat: add deployment script"
git push origin feature/my-feature
git pull --rebase origin main
git log --oneline --graph
git diff HEAD~1
git reset --soft HEAD~1   # Undo last commit, keep changes
git bisect start          # Binary search for bug-introducing commit
```

### Conventional Commits
Use a standard commit message format:
```
feat: add kubernetes deployment config
fix: correct nginx proxy timeout
docs: update README with setup instructions
chore: upgrade node version to 20
ci: add github actions workflow
```

---

<a name="phase-4"></a>
## 💻 Phase 4 — Programming / Scripting Languages
> *You don't need to be a full software engineer, but you must be able to read and write code.*

### Must Know: Python
Python is THE language of DevOps tooling and automation.

```python
import subprocess
import os
import boto3  # AWS SDK
import requests
import yaml
import json

# Run shell commands
result = subprocess.run(['docker', 'ps'], capture_output=True, text=True)
print(result.stdout)

# Read config files
with open('config.yaml') as f:
    config = yaml.safe_load(f)

# AWS automation with boto3
ec2 = boto3.client('ec2', region_name='ap-south-1')
instances = ec2.describe_instances()
```

**Build with Python:**
- Scripts to interact with AWS/GCP/Azure APIs
- Parse and transform JSON/YAML
- Automate repetitive infrastructure tasks
- Write custom monitoring scripts
- Build simple CLI tools

### Know Enough Of: Go (Golang)
Many DevOps tools are written in Go (Docker, Kubernetes, Terraform, Prometheus). You should be able to read Go code and understand simple programs.

### Know Enough Of: JavaScript/Node.js
Useful for writing Lambda functions, working with REST APIs, and many web-based DevOps tools.

### YAML / JSON / TOML
You'll write YAML **constantly** in DevOps (Kubernetes manifests, CI/CD pipelines, Ansible playbooks, Docker Compose files).

```yaml
# Kubernetes Deployment example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
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
          image: my-app:v1.2.0
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: production
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

---

<a name="phase-5"></a>
## 🐳 Phase 5 — Containers (Docker)
> *Containers solve the "works on my machine" problem forever.*

### Core Docker Concepts
- **Image** — read-only template (think: class)
- **Container** — running instance of an image (think: object)
- **Dockerfile** — instructions to build an image
- **Registry** — Docker Hub, AWS ECR, GitHub Container Registry, Google Artifact Registry
- **Volumes** — persist data outside the container
- **Networks** — container-to-container communication
- **Docker Compose** — multi-container apps defined in YAML

### Dockerfile Best Practices
```dockerfile
# Use specific base image tags — never just "latest"
FROM node:20-alpine AS builder

# Set working directory
WORKDIR /app

# Copy dependency files first (better layer caching)
COPY package*.json ./
RUN npm ci --only=production

# Copy application code
COPY . .

# Build the app
RUN npm run build

# ---- Production Stage ----
FROM node:20-alpine AS production

WORKDIR /app

# Don't run as root!
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

CMD ["node", "dist/main.js"]
```

### Docker Commands
```bash
# Build & Run
docker build -t my-app:v1.0.0 .
docker run -d -p 3000:3000 --name my-app my-app:v1.0.0

# Inspect & Debug
docker logs -f my-app
docker exec -it my-app sh
docker inspect my-app
docker stats

# Registry
docker tag my-app:v1.0.0 registry.example.com/my-app:v1.0.0
docker push registry.example.com/my-app:v1.0.0

# Cleanup
docker system prune -af
docker volume prune
```

### Docker Compose
```yaml
version: '3.9'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGO_URI=mongodb://mongo:27017/mydb
    depends_on:
      mongo:
        condition: service_healthy
    restart: unless-stopped

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mongo_data:
```

---

<a name="phase-6"></a>
## ⚓ Phase 6 — Container Orchestration (Kubernetes)
> *Kubernetes (K8s) is the industry standard for running containers at scale.*

### Core Kubernetes Concepts

| Object | Purpose |
|--------|---------|
| **Pod** | Smallest deployable unit — one or more containers |
| **Deployment** | Manages replicas, rolling updates, rollbacks |
| **Service** | Stable network endpoint for pods (ClusterIP, NodePort, LoadBalancer) |
| **Ingress** | HTTP routing to services (Nginx Ingress, Traefik) |
| **ConfigMap** | Non-sensitive config data |
| **Secret** | Sensitive data (base64-encoded) |
| **Namespace** | Logical isolation of resources |
| **PersistentVolume** | Storage abstraction |
| **StatefulSet** | For stateful apps (databases) |
| **DaemonSet** | Run one pod per node (log collectors, monitors) |
| **HPA** | Horizontal Pod Autoscaler — scale based on CPU/memory |
| **RBAC** | Role-Based Access Control |

### kubectl Cheatsheet
```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Deploy & manage
kubectl apply -f deployment.yaml
kubectl get pods -n production
kubectl describe pod my-pod-xxxxx
kubectl logs -f my-pod-xxxxx
kubectl exec -it my-pod-xxxxx -- sh

# Scaling
kubectl scale deployment my-app --replicas=5

# Rolling update
kubectl set image deployment/my-app my-app=my-app:v1.3.0

# Rollback
kubectl rollout undo deployment/my-app
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app

# Port forwarding (for local debugging)
kubectl port-forward svc/my-service 8080:80

# Resource usage
kubectl top nodes
kubectl top pods
```

### Kubernetes Tools to Know
- **Helm** — Kubernetes package manager; write reusable charts
- **Kustomize** — YAML customization without templating
- **ArgoCD / Flux** — GitOps: K8s state driven by Git
- **Lens / k9s** — Kubernetes GUI/TUI clients
- **Skaffold** — Local K8s development workflow
- **Cert-Manager** — Automatic SSL certificates in K8s

### Managed Kubernetes Services
- **AWS EKS** — Elastic Kubernetes Service
- **GCP GKE** — Google Kubernetes Engine (best managed K8s)
- **Azure AKS** — Azure Kubernetes Service

---

<a name="phase-7"></a>
## ⚙️ Phase 7 — CI/CD Pipelines
> *CI/CD is the backbone of DevOps. Every code change should flow through a pipeline.*

### What CI/CD Means
- **Continuous Integration (CI)** — Every commit triggers: build → test → lint → security scan
- **Continuous Delivery (CD)** — Automatically deploy to staging; one-click to prod
- **Continuous Deployment** — Automatically deploy to prod after all checks pass

### Pipeline Stages (Typical)
```
Code Push → Lint → Unit Tests → Build Docker Image → Integration Tests
→ Security Scan → Push to Registry → Deploy to Staging → E2E Tests
→ Deploy to Production → Post-deploy Health Checks
```

### GitHub Actions (Most Popular)
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  IMAGE_NAME: my-app
  REGISTRY: ghcr.io/${{ github.repository_owner }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/my-app \
            my-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

### Other CI/CD Tools
- **GitLab CI/CD** — Built into GitLab; powerful `.gitlab-ci.yml`
- **Jenkins** — Self-hosted; highly customizable; Jenkinsfile (Groovy)
- **CircleCI** — Cloud-native; fast and simple
- **Tekton** — Kubernetes-native CI/CD
- **ArgoCD** — GitOps CD for Kubernetes
- **Drone CI** — Container-native, lightweight

### Key CI/CD Concepts
- **Pipeline as Code** — Always store pipeline config in the repo
- **Secrets Management** — Never hardcode secrets; use vault or platform secrets
- **Artifact Storage** — Store build artifacts (Docker images, binaries)
- **Environment Promotion** — dev → staging → prod with approvals
- **Blue/Green Deployment** — Two identical envs; switch traffic
- **Canary Deployment** — Route % of traffic to new version
- **Feature Flags** — Deploy code without enabling it

---

<a name="phase-8"></a>
## 🏗️ Phase 8 — Infrastructure as Code (IaC)
> *Never click around in cloud consoles. All infrastructure should be code.*

### Terraform (Industry Standard)
```hcl
# main.tf — Deploy an AWS EC2 instance

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "ap-south-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  name    = "my-vpc"
  cidr    = "10.0.0.0/16"
  azs     = ["ap-south-1a", "ap-south-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  enable_nat_gateway = true
}

# EC2 Instance
resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  subnet_id              = module.vpc.private_subnets[0]
  vpc_security_group_ids = [aws_security_group.app.id]
  iam_instance_profile   = aws_iam_instance_profile.app.name

  user_data = templatefile("${path.module}/userdata.sh", {
    app_version = var.app_version
  })

  tags = {
    Name        = "app-server"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

### Terraform Workflow
```bash
terraform init        # Download providers & modules
terraform fmt         # Format code
terraform validate    # Validate syntax
terraform plan        # Preview changes (ALWAYS do this first)
terraform apply       # Apply changes
terraform destroy     # Destroy all resources (careful!)
terraform output      # Show outputs
terraform state list  # List managed resources
```

### Terraform Best Practices
- Use **modules** for reusable components
- Store state remotely in **S3 + DynamoDB** (locking)
- Use **workspaces** or separate state files per environment
- Always run `terraform plan` before `apply`
- Use **Terragrunt** for DRY Terraform configurations
- Use **tfsec** / **Checkov** for security scanning

### Other IaC Tools
- **AWS CloudFormation** — AWS-native IaC (YAML/JSON)
- **AWS CDK** — Define AWS infrastructure in code (Python, TypeScript)
- **Pulumi** — IaC in real languages (Python, Go, TypeScript)
- **Crossplane** — Kubernetes-based IaC

---

<a name="phase-9"></a>
## ☁️ Phase 9 — Cloud Platforms
> *Pick one cloud deeply, learn the others at a surface level.*

### Recommended: Start with AWS (Largest market share)

#### Core AWS Services Every DevOps Engineer Must Know

**Compute:**
- **EC2** — Virtual machines; instance types, AMIs, Auto Scaling Groups, Launch Templates
- **Lambda** — Serverless functions; event-driven architecture
- **ECS / EKS** — Container services (ECS for Docker, EKS for Kubernetes)
- **Fargate** — Serverless containers (no EC2 management)

**Networking:**
- **VPC** — Virtual Private Cloud; subnets, route tables, Internet Gateway, NAT Gateway
- **Security Groups** — Instance-level firewall
- **NACLs** — Subnet-level firewall
- **ALB / NLB** — Application/Network Load Balancers
- **CloudFront** — CDN
- **Route 53** — DNS service

**Storage:**
- **S3** — Object storage; versioning, lifecycle policies, static website hosting
- **EBS** — Block storage for EC2
- **EFS** — Managed NFS (shared file system)
- **RDS** — Managed databases (PostgreSQL, MySQL, Aurora)
- **DynamoDB** — Managed NoSQL
- **ElastiCache** — Managed Redis/Memcached

**DevOps & Operations:**
- **CodePipeline / CodeBuild / CodeDeploy** — AWS native CI/CD
- **ECR** — Elastic Container Registry
- **CloudWatch** — Logs, metrics, alarms, dashboards
- **CloudTrail** — Audit log of all API calls
- **Systems Manager (SSM)** — Remote access without SSH, Parameter Store, Session Manager
- **Secrets Manager** — Store and rotate secrets
- **IAM** — Identity & Access Management (CRITICAL to master)
- **KMS** — Key Management Service for encryption

**IAM Best Practices:**
- Follow **Principle of Least Privilege** always
- Use **IAM Roles** for services (never embed access keys in code)
- Enable **MFA** on all accounts
- Use **AWS Organizations** + **SCPs** for multi-account setups

---

<a name="phase-10"></a>
## 📊 Phase 10 — Monitoring, Logging & Observability
> *You can't fix what you can't see. Observability is non-negotiable in production.*

### The Three Pillars of Observability
1. **Metrics** — Numerical data over time (CPU %, request rate, error rate, latency)
2. **Logs** — Timestamped event records (application events, errors, access logs)
3. **Traces** — End-to-end request journey through microservices

### Metrics Stack: Prometheus + Grafana
```yaml
# prometheus.yml — scrape configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "alerts/*.yml"

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

**Grafana** — Build dashboards for all your metrics. Import community dashboards for Kubernetes, Node Exporter, MongoDB, etc.

**Key Metrics to Monitor:**
- **USE Method**: Utilization, Saturation, Errors (for infrastructure)
- **RED Method**: Rate, Errors, Duration (for services)
- CPU, Memory, Disk I/O, Network I/O
- HTTP request rate, error rate, p50/p95/p99 latency
- Pod restarts, OOMKilled events in Kubernetes

### Logging Stack: ELK or Loki
**ELK Stack:**
- **Elasticsearch** — Store and index logs
- **Logstash** — Log ingestion and transformation pipeline
- **Kibana** — Visualize and search logs

**Grafana Loki** (lighter weight):
- **Loki** — Log aggregation (like Prometheus but for logs)
- **Promtail** — Log shipping agent
- **Grafana** — Query and visualize logs

### Distributed Tracing
- **Jaeger** — Open-source tracing (Kubernetes-friendly)
- **Zipkin** — Another popular open-source tracer
- **AWS X-Ray** — AWS-native tracing
- **OpenTelemetry** — Vendor-neutral SDK for instrumentation (use this!)

### Alerting Best Practices
```yaml
# Prometheus alert rule example
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High HTTP error rate ({{ $value | humanizePercentage }})"
          description: "Error rate is above 5% for the past 5 minutes"
```

**Alerting Tools:** PagerDuty, OpsGenie, Slack (for non-critical), VictorOps

### Uptime Monitoring
- **Blackbox Exporter** (Prometheus) — Probe HTTP, TCP, DNS endpoints
- **UptimeRobot** — Free external monitoring
- **AWS CloudWatch Synthetics** — Canary scripts for endpoint checks

---

<a name="phase-11"></a>
## 🔧 Phase 11 — Configuration Management
> *Manage server configuration at scale without logging into every server manually.*

### Ansible (Most Popular)
Agentless, uses SSH. Written in YAML (Playbooks).

```yaml
# playbook.yml — Deploy and configure Nginx
---
- name: Configure web servers
  hosts: webservers
  become: yes  # Run as sudo

  vars:
    app_version: "v1.2.0"
    nginx_port: 80

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart Nginx

    - name: Start and enable Nginx
      systemd:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart Nginx
      systemd:
        name: nginx
        state: restarted
```

```ini
# inventory/hosts.ini
[webservers]
web1 ansible_host=10.0.1.10
web2 ansible_host=10.0.1.11

[dbservers]
db1 ansible_host=10.0.2.10
```

```bash
ansible-playbook -i inventory/hosts.ini playbook.yml
ansible-playbook playbook.yml --tags "nginx" --limit "web1"
ansible webservers -i inventory -m ping
```

### Ansible Best Practices
- Use **Roles** to organize playbooks
- Use **Ansible Vault** for secrets
- Store inventory in version control
- Use **Molecule** for testing roles
- Idempotency is critical — running twice should give same result

### Other Config Management Tools
- **Chef** — Ruby-based; powerful but complex; agent-based
- **Puppet** — Declarative; good for large scale; agent-based
- **SaltStack** — Fast; both push and pull models

---

<a name="phase-12"></a>
## 🔒 Phase 12 — Security in DevOps (DevSecOps)
> *Security is not a separate phase — it should be baked into every step.*

### Shift Left Security
"Shift left" means finding security issues early (in development) rather than late (in production).

### Security in CI/CD Pipeline

**Static Application Security Testing (SAST):**
- **Bandit** — Python code security scanner
- **Semgrep** — Multi-language code scanner
- **SonarQube** — Code quality + security analysis

**Software Composition Analysis (SCA) — Dependency Scanning:**
- **Snyk** — Find vulnerabilities in dependencies
- **Trivy** — Container & filesystem vulnerability scanner
- **Dependabot** — GitHub automated dependency updates

**Secret Scanning:**
- **GitGuardian** — Detect leaked secrets in commits
- **TruffleHog** — Scan git history for secrets
- **git-secrets** — Prevent committing secrets

**Container Security:**
```bash
# Scan Docker image for vulnerabilities
trivy image my-app:v1.0.0

# Scan filesystem
trivy fs .

# Scan Kubernetes manifests
trivy k8s --report summary cluster
```

**Infrastructure Security Scanning:**
- **tfsec** — Terraform security scanner
- **Checkov** — Policy-as-code for IaC
- **kube-bench** — CIS Kubernetes benchmark
- **Falco** — Runtime security for Kubernetes

### Secrets Management
```bash
# Never do this:
export DB_PASSWORD="mypassword123"  # BAD

# Do this instead:
# AWS Secrets Manager
aws secretsmanager get-secret-value --secret-id prod/db/password

# HashiCorp Vault
vault kv get secret/prod/database

# Kubernetes Secrets (use with External Secrets Operator)
kubectl create secret generic db-secret \
  --from-literal=password=$(aws secretsmanager get-secret-value ...)
```

**Tools:** HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault

### Network Security
- Use **private subnets** for databases and application servers
- Expose only what needs to be public (load balancers, CDN)
- Use **VPN/Bastion hosts** for admin access, never expose SSH to internet
- Enable **AWS GuardDuty** / **GCP Security Command Center** for threat detection
- Use **WAF** (Web Application Firewall) in front of public APIs

### Identity & Access Management Security
- **Principle of Least Privilege** — Always
- Use **IAM Roles**, never long-lived access keys
- Rotate credentials regularly
- Enable **CloudTrail** audit logging
- Use **AWS Config** / **GCP Org Policy** for compliance

### Compliance & Standards to Know
- **CIS Benchmarks** — Security configuration standards for OS, cloud, K8s
- **SOC 2** — Audit framework for cloud service providers
- **PCI DSS** — For payment card data
- **GDPR** — European data privacy regulation
- **ISO 27001** — Information security management

---

<a name="phase-13"></a>
## 🚀 Phase 13 — Advanced Topics
> *Once you're solid in the fundamentals, these make you a Senior DevOps Engineer.*

### GitOps
**GitOps = Git is the single source of truth for infrastructure AND applications.**
- All desired state is stored in Git
- Automated operators reconcile real state to desired state
- **ArgoCD** and **Flux** are the two main GitOps tools for Kubernetes

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/my-app-infra
    targetRevision: HEAD
    path: k8s/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Service Mesh
For microservices at scale — handles traffic management, mTLS, observability.
- **Istio** — Full-featured, complex; industry standard
- **Linkerd** — Lightweight, simpler
- **Consul Connect** — HashiCorp's service mesh

### Platform Engineering & Internal Developer Platforms (IDP)
- **Backstage** (by Spotify) — Developer portal; service catalog, documentation
- Build **Golden Paths** — opinionated, pre-configured ways to deploy apps

### Multi-Cloud & Hybrid Cloud
- Understand the tradeoffs of multi-cloud
- Tools: **Crossplane**, **Anthos** (GCP), **Arc** (Azure), **Terraform** (cloud-agnostic)

### FinOps — Cloud Cost Optimization
- Right-size EC2 instances; use **Spot Instances** for non-critical workloads
- Use **Reserved Instances** / **Savings Plans** for predictable workloads
- Set up cost alerts and budgets (AWS Budgets)
- Tools: **Infracost** (Terraform cost estimation), **Kubecost** (Kubernetes cost)

### Chaos Engineering
*Deliberately introduce failures to find weaknesses before users do.*
- **Chaos Monkey** (Netflix) — Random instance termination
- **LitmusChaos** — Kubernetes chaos engineering
- **AWS Fault Injection Simulator (FIS)**

### Performance Engineering
- Load testing with **k6**, **Locust**, **Apache JMeter**
- Profiling applications in production
- Database query optimization and connection pooling

### Site Reliability Engineering (SRE) Concepts
- **SLI** (Service Level Indicator) — What you measure
- **SLO** (Service Level Objective) — Target for that metric (e.g., 99.9% uptime)
- **SLA** (Service Level Agreement) — Contractual promise to customers
- **Error Budget** — How much downtime/errors you can afford
- **Toil Reduction** — Automate repetitive operational work
- **Runbooks** — Step-by-step guides for handling incidents
- **Post-mortems** — Blameless review of incidents

---

<a name="soft-skills"></a>
## 🧠 Soft Skills & Career Tips

### Mindset of a DevOps Engineer
- **Automation first** — If you did it twice manually, automate it
- **Everything as code** — Infrastructure, config, pipelines, documentation
- **Fail fast, recover faster** — Build systems that detect and recover from failure
- **Collaboration** — Work closely with Dev, QA, Security, and Product teams
- **Continuous learning** — The tooling changes every 2-3 years

### Documentation
- Write **runbooks** for every critical operation
- Document **architecture decisions** (Architecture Decision Records / ADRs)
- Keep **post-mortems** for every major incident
- Maintain an updated **on-call handbook**

### Incident Management
1. Detect (monitoring alert fires)
2. Triage (assess severity, P1/P2/P3)
3. Communicate (notify stakeholders)
4. Mitigate (stop the bleeding — rollback, scale up, etc.)
5. Resolve (fix root cause)
6. Post-mortem (blameless, improve processes)

### On-Call Best Practices
- Always have a **rollback plan** before deploying
- Deploy during low-traffic windows
- Automate deployment health checks
- Alert on symptoms, not just causes

---

<a name="resources"></a>
## 📚 Learning Resources

### Books
- *The Phoenix Project* — Gene Kim (DevOps culture, must read)
- *The DevOps Handbook* — Gene Kim, Jez Humble
- *Accelerate* — Nicole Forsgren (data-driven DevOps)
- *Site Reliability Engineering* — Google (free online at sre.google)
- *Kubernetes in Action* — Marko Lukša

### Platforms
- **KodeKloud** — Best hands-on DevOps courses (highly recommended)
- **A Cloud Guru / Pluralsight** — Cloud certifications
- **Linux Foundation** — Kubernetes certifications (CKA, CKAD, CKS)
- **Udemy** — Specific tool courses (TechWorld with Nana for K8s)
- **YouTube: TechWorld with Nana** — Best free DevOps content

### Certifications (In Priority Order)
1. **Linux Foundation CKA** (Certified Kubernetes Administrator) — Most valuable
2. **AWS Solutions Architect Associate** — Broad cloud knowledge
3. **AWS DevOps Engineer Professional** — Advanced AWS DevOps
4. **HashiCorp Terraform Associate** — Validates IaC knowledge
5. **Linux Foundation CKAD** (Certified Kubernetes Application Developer)
6. **CKS** (Certified Kubernetes Security Specialist) — Advanced

### Practice Environments
- **KillerCoda** — Free browser-based K8s practice
- **Play with Docker / Play with Kubernetes** — Free sandboxes
- **LocalStack** — Simulate AWS locally
- **Minikube / Kind / k3s** — Local Kubernetes clusters

---

<a name="timeline"></a>
## 🗓️ Roadmap Timeline Summary

| Phase | Topic | Duration | Priority |
|-------|-------|----------|----------|
| 0 | CS Foundations | 2 weeks | 🔴 Critical |
| 1 | Linux & Shell Scripting | 4–6 weeks | 🔴 Critical |
| 2 | Networking & Security Basics | 2–3 weeks | 🔴 Critical |
| 3 | Git & Version Control | 1–2 weeks | 🔴 Critical |
| 4 | Python Scripting | 3–4 weeks | 🔴 Critical |
| 5 | Docker & Containers | 3–4 weeks | 🔴 Critical |
| 6 | Kubernetes | 6–8 weeks | 🔴 Critical |
| 7 | CI/CD Pipelines | 3–4 weeks | 🔴 Critical |
| 8 | Terraform / IaC | 4–5 weeks | 🟠 High |
| 9 | Cloud (AWS) | 6–8 weeks | 🟠 High |
| 10 | Monitoring & Observability | 3–4 weeks | 🟠 High |
| 11 | Ansible / Config Management | 2–3 weeks | 🟡 Medium |
| 12 | DevSecOps | 2–3 weeks | 🟠 High |
| 13 | Advanced Topics | Ongoing | 🟡 Medium |

**Total for job-ready Junior DevOps: ~6-9 months of focused learning**
**Total for Mid-level DevOps: ~12-18 months with hands-on experience**

---

## 🛠️ Tools Summary at a Glance

| Category | Tools |
|----------|-------|
| **OS** | Ubuntu, CentOS, Amazon Linux |
| **Scripting** | Bash, Python |
| **Version Control** | Git, GitHub, GitLab |
| **Containers** | Docker, Podman |
| **Orchestration** | Kubernetes, Helm, ArgoCD |
| **CI/CD** | GitHub Actions, GitLab CI, Jenkins |
| **IaC** | Terraform, Ansible, Pulumi |
| **Cloud** | AWS, GCP, Azure |
| **Monitoring** | Prometheus, Grafana, ELK Stack |
| **Security** | Trivy, Snyk, Vault, tfsec |
| **Service Mesh** | Istio, Linkerd |
| **Tracing** | Jaeger, OpenTelemetry |

---

> 💡 **Final Advice from a Senior DevOps Engineer:**
> 
> Don't try to learn everything at once. Pick a project — even a personal one like ShellElearning — and **actually deploy it using the DevOps practices** in this roadmap. Nothing replaces the experience of debugging a failing Kubernetes pod at 2am or figuring out why your Terraform state is corrupted. The tools change, but the problem-solving mindset stays forever.
>
> **Start here:** Linux → Docker → Kubernetes → Terraform → AWS → CI/CD → Monitor everything.
>
> *Good luck. The industry needs more great DevOps engineers.*

---

*Roadmap Version: 2025–2026 | Last Updated: March 2026*
