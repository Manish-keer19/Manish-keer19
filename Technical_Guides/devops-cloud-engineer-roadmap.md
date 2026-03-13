# 🚀 DevOps & Cloud Engineer Roadmap — Beginner to Advanced

> A complete, structured guide covering every tool, technology, and concept you need to master to become a professional DevOps Cloud Engineer.

---

## 📌 How to Use This Roadmap

- Follow the phases **in order** — each builds on the previous
- Don't rush. Spend real time on each section before moving on
- Build projects at every stage to reinforce learning
- Estimated total time: **12–24 months** (depending on pace)

---

## 🟢 PHASE 1 — Foundations (Beginner) | ~2–3 months

### 1.1 Linux & Command Line
> Everything in DevOps runs on Linux. This is non-negotiable.

| Topic | What to Learn |
|-------|--------------|
| Linux Basics | File system, permissions, users & groups |
| Shell Commands | `ls`, `cd`, `grep`, `awk`, `sed`, `find`, `curl`, `chmod`, `chown` |
| Shell Scripting | Bash scripting, variables, loops, conditionals, functions |
| Process Management | `ps`, `top`, `htop`, `kill`, `systemctl`, `journalctl` |
| Networking CLI | `ping`, `netstat`, `ss`, `nslookup`, `traceroute`, `iptables` |
| Package Management | `apt`, `yum`, `dnf`, `snap` |
| Text Editors | `vim`, `nano` |

**Tools:** Ubuntu/Debian, CentOS/RHEL, Bash

---

### 1.2 Networking Fundamentals
> You must understand how the internet and networks work.

| Topic | What to Learn |
|-------|--------------|
| OSI Model | 7 layers and what happens at each |
| Protocols | TCP/IP, UDP, HTTP/HTTPS, DNS, DHCP, SSH, FTP, SMTP |
| IP Addressing | IPv4, IPv6, subnetting, CIDR notation |
| Firewalls & Security Groups | Inbound/outbound rules, ports |
| Load Balancing | Round-robin, least connections, sticky sessions |
| VPN & Tunneling | Site-to-site, client VPN, SSH tunneling |

---

### 1.3 Version Control — Git
> Every DevOps workflow starts with Git.

| Topic | What to Learn |
|-------|--------------|
| Git Basics | `init`, `clone`, `add`, `commit`, `push`, `pull` |
| Branching | `branch`, `merge`, `rebase`, `cherry-pick` |
| Workflows | GitFlow, trunk-based development, feature branches |
| Collaboration | Pull Requests, code reviews, merge conflicts |
| Git Internals | `.git` folder, refs, objects, HEAD |

**Platforms:** GitHub, GitLab, Bitbucket

---

### 1.4 Programming & Scripting
> You don't need to be a full developer, but you must script.

| Language | Use Case |
|----------|----------|
| **Bash** | Automation, system scripting (mandatory) |
| **Python** | Automation, tooling, AWS SDKs, data parsing (mandatory) |
| **YAML** | Configs for Docker, Kubernetes, CI/CD (mandatory) |
| **JSON** | API responses, cloud configs (mandatory) |
| **Go** | Optional — used in Kubernetes, Terraform internals |

---

## 🔵 PHASE 2 — Core DevOps Skills (Beginner → Intermediate) | ~3–4 months

### 2.1 Containers — Docker
> Containers are the foundation of modern DevOps.

| Topic | What to Learn |
|-------|--------------|
| Container Concepts | Images vs containers, layers, registry |
| Dockerfile | `FROM`, `RUN`, `COPY`, `CMD`, `ENTRYPOINT`, `EXPOSE` |
| Docker CLI | `build`, `run`, `exec`, `ps`, `logs`, `stop`, `rm`, `rmi` |
| Networking | Bridge, host, overlay networks |
| Volumes | Bind mounts, named volumes, persistent storage |
| Docker Compose | Multi-container apps with `docker-compose.yml` |
| Docker Hub | Pushing/pulling images from public/private registries |
| Best Practices | Multi-stage builds, minimal images, `.dockerignore` |

**Tools:** Docker Desktop, Docker Hub, Docker Compose

---

### 2.2 CI/CD — Continuous Integration & Delivery
> Automate building, testing, and deploying code.

| Topic | What to Learn |
|-------|--------------|
| CI/CD Concepts | Pipeline stages, artifacts, triggers, environments |
| Pipeline Structure | Build → Test → Lint → Security Scan → Deploy |
| GitHub Actions | Workflows, actions, runners, secrets, matrix builds |
| GitLab CI | `.gitlab-ci.yml`, stages, jobs, runners |
| Jenkins | Freestyle jobs, Pipelines (Jenkinsfile), plugins |
| ArgoCD / FluxCD | GitOps continuous delivery for Kubernetes |
| CircleCI / TravisCI | SaaS CI tools (good to know) |

**Key Files:** `Jenkinsfile`, `.github/workflows/*.yml`, `.gitlab-ci.yml`

---

### 2.3 Infrastructure as Code (IaC)
> Stop clicking in consoles. Define everything as code.

| Tool | Purpose | Priority |
|------|---------|----------|
| **Terraform** | Multi-cloud IaC (HCL language) | ⭐ Must Learn |
| **Ansible** | Configuration management, agentless automation | ⭐ Must Learn |
| **Pulumi** | IaC with real programming languages (Python/JS) | Nice to know |
| **Chef / Puppet** | Legacy config management (enterprise orgs) | Optional |
| **AWS CloudFormation** | AWS-native IaC (JSON/YAML) | Learn if AWS-focused |
| **Azure Bicep / ARM** | Azure-native IaC | Learn if Azure-focused |

**Terraform Concepts:** Providers, resources, state files, modules, workspaces, remote backends

**Ansible Concepts:** Playbooks, inventory, roles, handlers, variables, vault

---

## 🟡 PHASE 3 — Cloud Platforms (Intermediate) | ~3–4 months

### 3.1 Choose Your Primary Cloud
> Pick ONE to master first, then learn the others.

| Cloud | Market Share | Best For |
|-------|-------------|----------|
| **AWS** | ~32% | Most jobs, widest ecosystem |
| **Azure** | ~22% | Enterprise, Microsoft shops |
| **GCP** | ~11% | Data engineering, Kubernetes |

---

### 3.2 AWS — Amazon Web Services (Recommended First)

**Compute:**
- EC2 — Virtual machines, instance types, AMIs, Auto Scaling Groups
- Lambda — Serverless functions, event triggers, layers
- ECS / EKS — Container and Kubernetes managed services
- Elastic Beanstalk — PaaS for easy deployments

**Storage:**
- S3 — Object storage, buckets, policies, versioning, lifecycle rules
- EBS — Block storage for EC2
- EFS — Elastic shared file system
- Glacier — Archival storage

**Networking:**
- VPC — Virtual Private Cloud, subnets, route tables
- Security Groups & NACLs — Firewall rules
- Route 53 — DNS service
- CloudFront — CDN
- API Gateway — REST/GraphQL API management
- ELB / ALB / NLB — Load balancers

**Databases:**
- RDS — Managed relational DBs (MySQL, PostgreSQL, etc.)
- DynamoDB — NoSQL
- ElastiCache — Redis/Memcached
- Aurora — High-performance managed DB

**Identity & Security:**
- IAM — Users, roles, policies, groups
- AWS Organizations — Multi-account management
- KMS — Key management
- Secrets Manager — Store and rotate secrets
- GuardDuty / Security Hub — Threat detection

**Monitoring & Observability:**
- CloudWatch — Metrics, logs, alarms, dashboards
- CloudTrail — API audit logging
- AWS Config — Resource compliance and history
- X-Ray — Distributed tracing

**DevOps Services:**
- CodePipeline, CodeBuild, CodeDeploy — AWS-native CI/CD
- ECR — Container registry
- CloudFormation — IaC

**Certifications (in order):**
1. AWS Cloud Practitioner (CLF-C02)
2. AWS Solutions Architect Associate (SAA-C03) ⭐ Most valuable
3. AWS DevOps Engineer Professional (DOP-C02)

---

### 3.3 Azure (Secondary)

| Service | AWS Equivalent |
|---------|---------------|
| Azure VM | EC2 |
| Azure Blob Storage | S3 |
| Azure Kubernetes Service (AKS) | EKS |
| Azure DevOps / Pipelines | CodePipeline + GitHub Actions |
| Azure Active Directory (AAD) | IAM + Cognito |
| Azure Monitor | CloudWatch |
| Azure Functions | Lambda |
| Azure Virtual Network (VNet) | VPC |

**Certifications:** AZ-900 → AZ-104 → AZ-400 (DevOps Engineer)

---

### 3.4 GCP (Tertiary)

| Service | AWS Equivalent |
|---------|---------------|
| Compute Engine | EC2 |
| Cloud Storage | S3 |
| GKE (Google Kubernetes Engine) | EKS |
| Cloud Run | Fargate |
| BigQuery | Redshift |
| Cloud Build | CodeBuild |

**Certifications:** Associate Cloud Engineer → Professional DevOps Engineer

---

## 🟠 PHASE 4 — Kubernetes & Container Orchestration (Intermediate) | ~2–3 months

### 4.1 Kubernetes Core Concepts
> The most in-demand DevOps skill today.

| Concept | What to Learn |
|---------|--------------|
| Architecture | Control Plane (API Server, etcd, Scheduler, Controller Manager) + Worker Nodes (kubelet, kube-proxy) |
| Workloads | Pods, Deployments, StatefulSets, DaemonSets, Jobs, CronJobs |
| Networking | Services (ClusterIP, NodePort, LoadBalancer), Ingress, DNS |
| Storage | PersistentVolumes (PV), PersistentVolumeClaims (PVC), StorageClasses |
| Configuration | ConfigMaps, Secrets, Environment Variables |
| Scaling | HPA (Horizontal Pod Autoscaler), VPA, Cluster Autoscaler |
| RBAC | Roles, ClusterRoles, RoleBindings, ServiceAccounts |
| Namespaces | Resource isolation and organization |
| Resource Management | Requests, Limits, LimitRanges, ResourceQuotas |

**Tools:** `kubectl`, `k9s`, `Lens` (Kubernetes IDE)

---

### 4.2 Kubernetes Ecosystem

| Tool | Purpose |
|------|---------|
| **Helm** | Kubernetes package manager — charts for deployments |
| **Kustomize** | Template-free Kubernetes config management |
| **ArgoCD** | GitOps continuous delivery |
| **FluxCD** | GitOps alternative to ArgoCD |
| **Istio / Linkerd** | Service mesh — traffic management, mTLS |
| **Cert-Manager** | Automatic TLS certificate management |
| **External DNS** | Auto-manage DNS records from K8s |
| **Velero** | Kubernetes backup and restore |
| **Karpenter** | Node autoscaling for AWS EKS |

**Certifications:**
- CKA — Certified Kubernetes Administrator ⭐ Most valuable
- CKAD — Certified Kubernetes Application Developer
- CKS — Certified Kubernetes Security Specialist

---

## 🔴 PHASE 5 — Monitoring, Observability & Security (Intermediate → Advanced) | ~2–3 months

### 5.1 Monitoring & Observability

**The Three Pillars:**

| Pillar | Tools |
|--------|-------|
| **Metrics** | Prometheus, Grafana, Datadog, CloudWatch |
| **Logs** | ELK Stack (Elasticsearch, Logstash, Kibana), Loki + Grafana, Splunk, Fluentd |
| **Traces** | Jaeger, Zipkin, AWS X-Ray, OpenTelemetry |

**Key Tools:**

| Tool | Purpose |
|------|---------|
| **Prometheus** | Metrics collection and alerting rules |
| **Grafana** | Dashboards and visualization |
| **AlertManager** | Alert routing and notification |
| **ELK / EFK Stack** | Centralized log management |
| **Datadog** | Full-stack monitoring SaaS |
| **PagerDuty / OpsGenie** | Incident management and on-call |
| **OpenTelemetry** | Vendor-neutral observability framework |

---

### 5.2 Security (DevSecOps)
> Security must be built into every stage of the pipeline.

| Area | Tools & Concepts |
|------|-----------------|
| **Secret Management** | HashiCorp Vault, AWS Secrets Manager, Azure Key Vault |
| **SAST** | Static code analysis — SonarQube, Semgrep, Snyk |
| **DAST** | Dynamic analysis — OWASP ZAP, Burp Suite |
| **Container Security** | Trivy, Clair, Snyk Container — image scanning |
| **Infrastructure Scanning** | Checkov, tfsec, Terrascan — IaC security |
| **Runtime Security** | Falco — threat detection in Kubernetes |
| **Compliance** | CIS Benchmarks, SOC2, ISO 27001, HIPAA, PCI-DSS |
| **Zero Trust** | mTLS, network policies, RBAC, least privilege |
| **SBOM** | Software Bill of Materials — Syft, Grype |

---

## 🟣 PHASE 6 — Advanced Topics (Advanced) | ~3–6 months

### 6.1 Advanced Kubernetes

| Topic | Tools |
|-------|-------|
| Multi-cluster management | Rancher, Karmada, Fleet |
| Service Mesh | Istio, Linkerd, Consul Connect |
| Policy Enforcement | OPA (Open Policy Agent), Gatekeeper, Kyverno |
| Custom Resources | CRDs (Custom Resource Definitions), Operators |
| Operator Pattern | Operator SDK, Kubebuilder |
| eBPF Networking | Cilium, Hubble |

---

### 6.2 Advanced Terraform

| Topic | What to Learn |
|-------|--------------|
| Modules | Writing reusable modules, module registry |
| Remote State | S3 + DynamoDB backend, Terraform Cloud |
| Workspaces | Multi-environment management |
| Terragrunt | DRY Terraform configurations |
| Testing | Terratest, Checkov |
| Drift Detection | `terraform plan` in CI, drift alerts |

---

### 6.3 GitOps & Platform Engineering

| Concept | Tools |
|---------|-------|
| GitOps | ArgoCD, FluxCD, Weave GitOps |
| Internal Developer Platform (IDP) | Backstage (Spotify), Port |
| Service Catalog | Backstage plugins, templates |
| Golden Paths | Standardized templates for dev teams |
| Platform as a Product | Thinking like a platform engineer |

---

### 6.4 Serverless & Event-Driven Architecture

| Tool | Cloud |
|------|-------|
| AWS Lambda + API Gateway | AWS |
| Azure Functions | Azure |
| Google Cloud Functions / Cloud Run | GCP |
| Knative | Kubernetes-native serverless |
| Event Bridges | AWS EventBridge, Azure Event Grid |
| Message Queues | SQS, SNS, Kafka, RabbitMQ |

---

### 6.5 Databases & Data Engineering (DevOps angle)

| Topic | Tools |
|-------|-------|
| Managed DBs | RDS, Aurora, Cloud SQL, Azure Database |
| Database Migrations | Flyway, Liquibase |
| DB Backups & DR | Automated snapshots, point-in-time recovery |
| Caching | Redis, Memcached, ElastiCache |
| Search | Elasticsearch, OpenSearch |
| Streaming | Apache Kafka, Kinesis |

---

### 6.6 Cost Optimization

| Tool | Purpose |
|------|---------|
| AWS Cost Explorer | Visualize and analyze AWS spend |
| Infracost | Estimate Terraform cost changes in CI |
| Kubecost | Kubernetes cost allocation |
| Spot Instances | 70–90% savings on interruptible workloads |
| Reserved Instances | Commit for 1–3 years for discounts |
| FinOps Framework | Organizational approach to cloud cost |

---

## 🧰 FULL TOOL SUMMARY

### Must Know (Non-Negotiable)
```
Linux/Bash · Git · Docker · Terraform · Ansible
Kubernetes · Helm · AWS (or Azure/GCP) · Python
GitHub Actions (or GitLab CI) · Prometheus + Grafana
```

### Should Know (Highly Recommended)
```
ArgoCD · HashiCorp Vault · ELK Stack · Trivy
Istio · Kustomize · AWS CloudWatch · Datadog
Checkov · SonarQube · Jenkins · Backstage
```

### Nice to Know (Differentiators)
```
Pulumi · Go · Falco · Cilium · OPA/Gatekeeper
Terragrunt · Knative · Kafka · Kubebuilder
Velero · Karpenter · OpenTelemetry
```

---

## 📜 CERTIFICATIONS ROADMAP

```
BEGINNER
  └── AWS Cloud Practitioner (CLF-C02)
  └── Linux Foundation: Introduction to Linux (LFS101)

INTERMEDIATE
  └── AWS Solutions Architect Associate (SAA-C03)  ← Most Valuable
  └── HashiCorp Terraform Associate
  └── Certified Kubernetes Administrator (CKA)     ← Most Valuable

ADVANCED
  └── AWS DevOps Engineer Professional (DOP-C02)
  └── Certified Kubernetes Security Specialist (CKS)
  └── AWS Solutions Architect Professional (SAP-C02)
  └── Google Professional DevOps Engineer
  └── Azure DevOps Engineer Expert (AZ-400)
```

---

## 🗓️ SUGGESTED LEARNING TIMELINE

| Month | Focus |
|-------|-------|
| 1–2 | Linux, Bash, Networking, Git |
| 3 | Python scripting + Docker |
| 4 | CI/CD (GitHub Actions, Jenkins) |
| 5–6 | Terraform + Ansible + AWS basics |
| 7 | AWS deep dive → SAA-C03 cert |
| 8–9 | Kubernetes + Helm + CKA prep |
| 10 | Monitoring (Prometheus, Grafana, ELK) |
| 11 | Security (Vault, Trivy, DevSecOps) |
| 12 | GitOps (ArgoCD) + Advanced Kubernetes |
| 13–18 | Platform engineering, advanced cloud, cost optimization |
| 18–24 | Specialization, open source contributions, senior-level projects |

---

## 🏗️ PROJECT IDEAS BY LEVEL

### Beginner Projects
- Automate server setup with a Bash script
- Build and push a Dockerized web app to Docker Hub
- Set up a GitHub Actions CI pipeline for a Python app
- Deploy EC2 instance using Terraform

### Intermediate Projects
- Deploy a microservices app on Kubernetes with Helm
- Build a full CI/CD pipeline: Code → Build → Test → Deploy to EKS
- Create a multi-environment Terraform setup (dev/staging/prod)
- Set up Prometheus + Grafana monitoring for a Kubernetes cluster

### Advanced Projects
- Build an Internal Developer Platform with Backstage
- Implement GitOps with ArgoCD across multiple clusters
- Create a Kubernetes Operator with Kubebuilder
- Build a full DevSecOps pipeline with SAST, DAST, and image scanning
- Design a multi-cloud disaster recovery architecture

---

## 📚 TOP LEARNING RESOURCES

| Resource | Type | Best For |
|----------|------|----------|
| A Cloud Guru / Pluralsight | Video courses | Cloud certs |
| KodeKloud | Hands-on labs | K8s, DevOps tools |
| Linux Foundation (training.linuxfoundation.org) | Courses | CKA/CKAD/CKS |
| Terraform docs (terraform.io) | Official docs | Terraform |
| Kubernetes docs (kubernetes.io) | Official docs | K8s |
| AWS Skill Builder | Free/paid | AWS services |
| GitHub (open source projects) | Practice | Real code |
| DevOps Roadmap (roadmap.sh/devops) | Visual guide | Overview |
| The DevOps Handbook (book) | Book | Culture & practices |
| Site Reliability Engineering (Google book) | Book | SRE principles |

---

*Last updated: 2025 — This roadmap reflects current industry demand and hiring trends.*
