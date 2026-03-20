# 🚀 Jenkins Mastery Guide
### Beginner to Advanced with CI/CD, Docker & Real-World Projects

---

> **Who is this guide for?**
> Complete beginners who just heard about Jenkins, developers who want to automate their workflows, and DevOps engineers looking for a production-grade reference. This guide will take you from zero to job-ready.

---

## Table of Contents

1. [Introduction to Jenkins](#1-introduction-to-jenkins)
2. [CI/CD Concepts — Deep Explanation](#2-cicd-concepts--deep-explanation)
3. [Jenkins Architecture](#3-jenkins-architecture)
4. [Installing Jenkins](#4-installing-jenkins)
5. [Jenkins UI Overview](#5-jenkins-ui-overview)
6. [Jenkins Jobs](#6-jenkins-jobs)
7. [Jenkins Pipeline — Core Topic](#7-jenkins-pipeline--core-topic)
8. [Pipeline Stages & Steps](#8-pipeline-stages--steps)
9. [GitHub Integration](#9-github-integration)
10. [Jenkins + Docker](#10-jenkins--docker)
11. [Jenkins Without Docker](#11-jenkins-without-docker)
12. [Credentials Management](#12-credentials-management)
13. [Artifacts Management](#13-artifacts-management)
14. [Parallel Execution](#14-parallel-execution)
15. [Multi-Branch Pipelines](#15-multi-branch-pipelines)
16. [Deployment Strategies](#16-deployment-strategies)
17. [Jenkins + Kubernetes](#17-jenkins--kubernetes)
18. [Logs & Debugging](#18-logs--debugging)
19. [Security Best Practices](#19-security-best-practices)
20. [Real-World CI/CD Project — Full MERN Pipeline](#20-real-world-cicd-project--full-mern-pipeline)
21. [Advanced Concepts](#21-advanced-concepts)
22. [Common Mistakes](#22-common-mistakes)
23. [Interview Questions & Answers](#23-interview-questions--answers)
24. [DevOps Roadmap Using Jenkins](#24-devops-roadmap-using-jenkins)

---

## 1. Introduction to Jenkins

### What is Jenkins?

Jenkins is an open-source **automation server** written in Java. It is the most widely used CI/CD tool in the world, enabling teams to automatically build, test, and deploy their software — every single time a developer pushes code.

Think of Jenkins as a tireless robot that sits between your code repository and your production server. Every time a developer commits code, Jenkins wakes up, grabs that code, runs all your tests, builds your application, packages it, and ships it — without any human clicking a single button.

```
Developer pushes code
        │
        ▼
  GitHub/GitLab/Bitbucket
        │
        ▼ (webhook triggers Jenkins)
     Jenkins
        │
   ┌────┴────┐
   │         │
Build       Test
   │         │
   └────┬────┘
        │
      Deploy
        │
        ▼
  Production Server
```

### Why Jenkins?

| Feature | Benefit |
|---|---|
| **Free & Open Source** | No licensing costs — critical for startups and enterprises alike |
| **Plugin Ecosystem** | 1,800+ plugins for Git, Docker, Kubernetes, Slack, AWS, and more |
| **Language Agnostic** | Works with Java, Node.js, Python, Ruby, Go, .NET — anything |
| **Distributed Builds** | Run builds across many machines in parallel |
| **Pipeline as Code** | Version-controlled `Jenkinsfile` lives alongside your app code |
| **Community** | Massive community, extensive documentation, years of production hardening |

### Role in CI/CD

Jenkins is the **orchestration engine** of your CI/CD pipeline. It doesn't compile code, run tests, or deploy applications by itself — instead, it **orchestrates** the tools that do those things (Maven, npm, Docker, kubectl, Ansible, etc.) and connects them in a defined workflow.

### Real-World Use Cases

- **Facebook/Meta scale:** Thousands of builds per day, running tests on every code change
- **E-commerce platforms:** Auto-deploy to staging after every merge, require manual approval before production
- **Fintech apps:** Run security scans (SAST/DAST), compliance checks, and unit tests before any deployment
- **Microservices:** Build and deploy 50+ independent services with separate pipelines
- **Your MERN app:** Pull from GitHub → install packages → test → Docker build → push to Hub → deploy to VPS

---

## 2. CI/CD Concepts — Deep Explanation

### The Problem CI/CD Solves

Before CI/CD, teams would work in isolation for weeks, then merge all code together — creating "merge hell" and "it works on my machine" disasters. Releases were manual, error-prone, and happened once a month.

CI/CD changes this fundamentally.

### Continuous Integration (CI)

**Definition:** The practice of merging all developer code changes into a shared repository **frequently** (multiple times a day), with each merge triggering an **automated build and test** cycle.

**What CI does:**
- Detects integration bugs within minutes of a commit
- Ensures the codebase is always in a working state
- Reduces "merge hell" by integrating small changes often
- Provides fast feedback to developers

```
Developer A commits  ──┐
Developer B commits  ──┼──▶  Shared Repo  ──▶  Jenkins  ──▶  Build + Test
Developer C commits  ──┘                                    ▶  Notify result
```

### Continuous Delivery (CD — Delivery)

**Definition:** An extension of CI where code that passes all tests is **automatically prepared for release** to production — but a **human must approve** the final deployment.

**Key distinction:** Every build is deployable, but a human clicks "approve" before it goes live.

**Use case:** Banks, healthcare systems, regulated environments where a release manager must sign off.

### Continuous Deployment (CD — Deployment)

**Definition:** The fully automated version — every change that passes all automated tests is **automatically deployed to production** without human intervention.

**Key distinction:** No human approval step. 100% automated.

**Use case:** Netflix, Amazon, high-velocity SaaS companies deploying hundreds of times per day.

### The Full CI/CD Lifecycle — ASCII Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    CI/CD LIFECYCLE                              │
└─────────────────────────────────────────────────────────────────┘

   CODE          BUILD         TEST          RELEASE       DEPLOY
     │             │             │              │              │
  ┌──┴──┐       ┌──┴──┐       ┌──┴──┐       ┌──┴──┐       ┌──┴──┐
  │ Dev │──git──▶Compile│──────▶ Unit│──────▶Package│──────▶ Prod│
  │Commit│  push │ &   │       │Tests│       │ &   │       │     │
  └─────┘       │Link │       │     │       │Tag  │       │     │
                └─────┘       │Integ│       └──┬──┘       └─────┘
                              │Tests│          │             ▲
                              │     │       ┌──┴──┐          │
                              │SAST │       │Staging│──approve┘
                              │Scan │       │Deploy│  (Delivery)
                              └─────┘       └─────┘  or auto
                                                     (Deployment)

◀──────────────── Continuous Integration ──────────────▶
◀──────────────────────── Continuous Delivery ─────────────────▶
◀──────────────────────────── Continuous Deployment ──────────────────▶
```

### CI/CD Key Principles

| Principle | Description |
|---|---|
| **Commit often** | Small, frequent commits are easier to test and debug |
| **Fix broken builds immediately** | A failing pipeline is the team's #1 priority |
| **Build once, deploy many** | The same artifact goes to staging AND production |
| **Everything automated** | If a human does it manually, automate it |
| **Fast feedback** | Developers should know within minutes if their code broke something |

---

## 3. Jenkins Architecture

### Overview

Jenkins follows a **Controller-Agent** (formerly Master-Slave) architecture that allows it to distribute work across multiple machines.

```
┌──────────────────────────────────────────────────┐
│                  JENKINS CONTROLLER               │
│                                                  │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │  Job Queue  │  │  Scheduler   │  │ Plugins │ │
│  └─────────────┘  └──────────────┘  └─────────┘ │
│  ┌─────────────────────────────────────────────┐ │
│  │           Jenkins Web UI (Port 8080)         │ │
│  └─────────────────────────────────────────────┘ │
└──────────┬───────────────────────────────────────┘
           │  SSH / JNLP / WebSocket
    ┌──────┴────────────────────────────┐
    │                                   │
┌───┴────────┐                   ┌──────┴───────┐
│  AGENT 1   │                   │   AGENT 2    │
│ (Linux VM) │                   │  (Docker)    │
│            │                   │              │
│ Executor 1 │                   │  Executor 1  │
│ Executor 2 │                   │  Executor 2  │
└────────────┘                   └──────────────┘
```

### Jenkins Controller (Master)

The Controller is the **brain** of Jenkins. It:

- Hosts the Jenkins Web UI
- Stores all configuration, job definitions, and build history
- Manages the job queue and decides which agent runs which job
- Does NOT run builds itself (best practice — reserve the controller for orchestration only)
- Coordinates all agents

**Important:** In production, never run builds on the controller. Keep it dedicated to management.

### Jenkins Agents (Nodes)

Agents are the **workers** — they actually execute pipeline steps. An agent can be:

- A bare-metal server
- A virtual machine (AWS EC2, Azure VM, GCP instance)
- A Docker container (ephemeral, spun up for a build, destroyed afterward)
- A Kubernetes pod (most scalable option)

Agents connect to the controller via:
- **SSH** — controller initiates connection (most common for static agents)
- **JNLP/WebSocket** — agent initiates connection (common for agents behind firewalls)

### Executors

An **executor** is a single compute thread on an agent. If an agent has 3 executors, it can run 3 builds simultaneously.

```
AGENT (4 CPUs, 8GB RAM)
├── Executor 1 → Running: Build #101 (feature/login)
├── Executor 2 → Running: Build #102 (feature/cart)
├── Executor 3 → Idle
└── Executor 4 → Idle
```

### How Jenkins Works Internally — Step by Step

1. Developer pushes code to GitHub
2. GitHub webhook fires an HTTP POST to `http://jenkins:8080/github-webhook/`
3. Jenkins receives the webhook and adds a job to the **build queue**
4. The **scheduler** finds an available executor on a matching agent
5. Jenkins fetches the `Jenkinsfile` from the repo
6. Jenkins executes each stage defined in the `Jenkinsfile` on the agent
7. Build results (logs, artifacts, status) are sent back to the controller
8. Jenkins updates the job dashboard and sends notifications (Slack, email, etc.)

---

## 4. Installing Jenkins

### 4.1 On Ubuntu (Recommended for Production)

**Prerequisites:** Ubuntu 20.04 / 22.04 / 24.04

#### Step 1: Install Java (Jenkins requires Java 17 or 21)

```bash
# Update package list
sudo apt update

# Install Java 21 (OpenJDK)
sudo apt install -y openjdk-21-jdk

# Verify installation
java -version
# Expected: openjdk version "21.x.x"

# Set JAVA_HOME (optional but recommended)
echo 'export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> ~/.bashrc
source ~/.bashrc
```

#### Step 2: Add Jenkins Repository and Install

```bash
# Add Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to sources list
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package list (now includes Jenkins)
sudo apt update

# Install Jenkins
sudo apt install -y jenkins
```

#### Step 3: Start and Enable Jenkins

```bash
# Start Jenkins service
sudo systemctl start jenkins

# Enable auto-start on reboot
sudo systemctl enable jenkins

# Check status
sudo systemctl status jenkins
# Should show: Active: active (running)
```

#### Step 4: Open Firewall (if UFW is active)

```bash
sudo ufw allow 8080
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

#### Step 5: Initial Setup

```bash
# Get the initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
# Copy this password — you'll need it in the browser
```

Open your browser: `http://YOUR_SERVER_IP:8080`

1. Paste the initial admin password
2. Select **"Install Suggested Plugins"**
3. Create your admin user
4. Configure Jenkins URL
5. Click **"Start using Jenkins"**

---

### 4.2 On Docker (Best for Local Development)

**Prerequisites:** Docker installed on your machine

#### Quick Start

```bash
# Pull and run Jenkins
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

#### Production-Ready Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock  # Allow Jenkins to use host Docker
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
    user: root  # Needed for Docker socket access

volumes:
  jenkins_home:
    driver: local
```

```bash
# Start Jenkins
docker-compose up -d

# View logs
docker-compose logs -f jenkins

# Get initial password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

#### Port Reference

| Port | Purpose |
|---|---|
| `8080` | Jenkins Web UI |
| `50000` | Agent inbound connections (JNLP) |

---

### 4.3 On Windows (Optional)

```powershell
# Using Chocolatey
choco install jenkins

# Or: Download the Jenkins .msi installer from https://www.jenkins.io/download/
# Run the installer → installs Jenkins as a Windows Service
# Access: http://localhost:8080
```

---

## 5. Jenkins UI Overview

### Dashboard

The main page after login. Shows:
- All your jobs and their current status
- Build queue (jobs waiting to run)
- Build executor status (currently running jobs)
- Links to create new items, manage Jenkins, and view credentials

**Build Status Icons:**

| Icon | Color | Meaning |
|---|---|---|
| ☀️ | Blue/Green | All recent builds passing |
| ⛅ | Yellow | Some recent builds failing |
| 🌩️ | Red | Most recent builds failing |
| ⚪ | Grey | Job never built / disabled |
| 🔵 | Animated | Build currently running |

### New Item

Click **"New Item"** to create jobs. Types available:
- **Freestyle Project** — Simple GUI-based job
- **Pipeline** — Code-based Jenkinsfile pipeline
- **Multibranch Pipeline** — Auto-discovers branches with Jenkinsfiles
- **Folder** — Organizes multiple jobs
- **Multi-configuration project** — Matrix builds across different environments

### Manage Jenkins

The control center for your Jenkins instance:
- **System Configuration** — Global settings, tools, environment variables
- **Plugins** — Install, update, remove plugins
- **Nodes & Clouds** — Manage agents
- **Credentials** — Store secrets, API keys, SSH keys
- **Security** — Authentication, authorization, CSRF protection
- **System Log** — Jenkins internal logs
- **Script Console** — Run Groovy scripts directly (powerful + dangerous)

### Build History

Each job has a build history panel showing:
- Build number (#1, #2, #3...)
- Timestamp
- Duration
- Status (success/failure/unstable)
- Link to console output

### Console Output

The most important debugging tool. Shows every command executed during the build, its output, and exit codes. Access via: **Job → Build # → Console Output**

```
Started by GitHub push by alice
[Pipeline] Start of Pipeline
[Pipeline] agent
[Pipeline] { (Declarative: Agent any)
[Pipeline] stage
[Pipeline] { (Checkout)
Cloning repository https://github.com/alice/myapp
 > git init
 > git fetch --tags
[Pipeline] stage
[Pipeline] { (Install)
+ npm install
added 1247 packages in 23s
[Pipeline] stage
[Pipeline] { (Test)
+ npm test
PASS  src/tests/app.test.js
Tests: 15 passed, 0 failed
[Pipeline] }
Finished: SUCCESS
```

---

## 6. Jenkins Jobs

### 6.1 Freestyle Jobs

**What they are:** The original Jenkins job type. Configuration is done entirely through the web UI — no code required. You fill in forms: SCM URL, build triggers, build steps (shell commands), post-build actions.

**When to use:** Simple, one-off tasks; legacy projects; quick experiments.

**Limitations:** Configuration is not version-controlled, hard to maintain, limited flexibility compared to pipelines.

#### Creating a Freestyle Job

1. Click **New Item** → Enter a name → Select **Freestyle project** → OK
2. **Source Code Management:** Select Git → Enter repository URL → Add credentials
3. **Build Triggers:** Check "GitHub hook trigger for GITScm polling" for auto-build
4. **Build Steps:** Click "Add build step" → "Execute shell"

```bash
# Example build steps for a Node.js app
npm install
npm test
npm run build
```

5. **Post-build Actions:** Archive artifacts, send email, notify Slack
6. Click **Save** → Click **Build Now** to test

### 6.2 Pipeline Jobs (The Modern Standard)

**Why pipelines are better than freestyle jobs:**

| Feature | Freestyle Job | Pipeline Job |
|---|---|---|
| Version controlled | ❌ Stored in Jenkins DB | ✅ `Jenkinsfile` in your repo |
| Code review | ❌ Not possible | ✅ PR review for pipeline changes |
| Visualization | ❌ Basic | ✅ Stage view, Blue Ocean |
| Complex workflows | ❌ Very limited | ✅ Parallel, conditionals, loops |
| Recovery | ❌ Hard to recover config | ✅ Config in Git history |
| Reusability | ❌ Copy-paste between jobs | ✅ Shared Libraries |
| Audit trail | ❌ No history of changes | ✅ Git commits track every change |

> 💡 **Industry standard:** All professional DevOps teams use Pipeline jobs. Freestyle jobs are considered legacy. Learn pipelines.

---

## 7. Jenkins Pipeline — Core Topic

### The Jenkinsfile

A `Jenkinsfile` is a text file that defines your entire pipeline as code. It lives in the **root of your repository** alongside your application code. This means:

- Pipeline changes go through code review like any other code change
- You can roll back pipeline changes by reverting a commit
- Every branch can have its own pipeline behavior

There are two syntaxes: **Declarative** (recommended) and **Scripted** (legacy/advanced).

---

### 7.1 Declarative Pipeline (Recommended)

Declarative pipelines have a strict, structured syntax that is easier to read and write. This is what you should learn first and use in production.

#### Basic Structure

```groovy
pipeline {
    agent any               // Where to run this pipeline

    environment {           // Environment variables (optional)
        APP_NAME = 'myapp'
        NODE_ENV = 'production'
    }

    stages {                // All stages go here
        stage('Checkout') {
            steps {
                git 'https://github.com/alice/myapp.git'
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }

    post {                  // Actions after pipeline completes
        success {
            echo 'Build succeeded! 🎉'
        }
        failure {
            echo 'Build failed! 💔'
            // mail to: 'team@company.com', subject: 'Build Failed'
        }
        always {
            echo 'Pipeline finished.'
        }
    }
}
```

#### Real Example: Node.js App with Tests and Docker

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myusername/myapp'
        DOCKER_TAG = "${BUILD_NUMBER}"
        NODE_VERSION = '18'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building branch: ${env.BRANCH_NAME}"
                echo "Build number: ${env.BUILD_NUMBER}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npm ci'           // ci is faster and more reliable than install
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test -- --coverage'
            }
            post {
                always {
                    // Publish test results (requires JUnit plugin)
                    junit 'test-results/*.xml'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh 'docker-compose -f docker-compose.staging.yml up -d'
            }
        }
    }

    post {
        always {
            sh 'docker logout'
            cleanWs()   // Clean workspace after build
        }
        success {
            slackSend(color: 'good', message: "✅ Build #${BUILD_NUMBER} succeeded on ${BRANCH_NAME}")
        }
        failure {
            slackSend(color: 'danger', message: "❌ Build #${BUILD_NUMBER} failed on ${BRANCH_NAME}")
        }
    }
}
```

---

### 7.2 Scripted Pipeline (Advanced/Legacy)

Scripted pipelines use a `node` block instead of `pipeline`, and give you the full power of Groovy. More flexible, but harder to read.

#### Basic Structure

```groovy
node {                          // Run on any available agent
    def appImage

    stage('Checkout') {
        checkout scm
    }

    stage('Build') {
        sh 'npm ci'
        sh 'npm run build'
    }

    stage('Test') {
        try {
            sh 'npm test'
        } catch (err) {
            currentBuild.result = 'UNSTABLE'
            echo "Tests failed: ${err}"
        }
    }

    stage('Docker') {
        appImage = docker.build("myapp:${env.BUILD_NUMBER}")
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
            appImage.push()
            appImage.push('latest')
        }
    }
}
```

#### When to Use Scripted vs Declarative

| Use Declarative when... | Use Scripted when... |
|---|---|
| Standard CI/CD pipelines | Complex conditional logic |
| Team collaboration | Dynamic stage generation |
| Readability is important | Programmatic pipeline construction |
| You're learning Jenkins | Working with legacy codebases |

> 💡 **Recommendation:** Always start with Declarative. Only switch to Scripted if you hit a concrete limitation.

---

## 8. Pipeline Stages & Steps

### `agent` — Where to Run

```groovy
pipeline {
    // Run on any available agent
    agent any

    // Run on a specific labeled agent
    agent { label 'linux-docker' }

    // Run in a Docker container
    agent {
        docker {
            image 'node:18-alpine'
            args '-v /tmp:/tmp'
        }
    }

    // Run in a Kubernetes pod
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: node
                image: node:18
            '''
        }
    }

    // No global agent — define per stage
    agent none
}
```

### `stages` and `stage`

```groovy
stages {
    stage('Stage Name') {
        // Every stage needs a steps block
        steps {
            echo 'Hello from this stage'
        }
    }
}
```

### `steps` — The Work Units

Common step commands:

```groovy
steps {
    // Run a shell command (Linux/Mac)
    sh 'npm install'
    sh '''
        echo "Multi-line"
        npm ci
        npm test
    '''

    // Run a PowerShell command (Windows agents)
    bat 'npm install'
    powershell 'Get-Process'

    // Print to console
    echo "Current build: ${env.BUILD_NUMBER}"

    // Checkout source code
    checkout scm

    // Copy files
    sh 'cp -r dist/ /var/www/html/'

    // Archive files as artifacts
    archiveArtifacts artifacts: 'dist/**/*', fingerprint: true

    // Publish JUnit test results
    junit 'test-results/junit.xml'

    // Timeout — fail if step takes too long
    timeout(time: 5, unit: 'MINUTES') {
        sh 'npm test'
    }

    // Retry — retry on failure
    retry(3) {
        sh 'curl https://flaky-service.com/api'
    }

    // Sleep
    sleep time: 30, unit: 'SECONDS'
}
```

### `environment` — Environment Variables

```groovy
pipeline {
    environment {
        // Static values
        APP_NAME = 'myapp'
        DEPLOY_ENV = 'production'

        // Use credentials — securely injects secret as env var
        API_KEY = credentials('my-api-key-credential-id')

        // Use Jenkins built-in variables
        BUILD_TAG_FULL = "${env.JOB_NAME}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Use Variables') {
            environment {
                // Stage-level environment variable (overrides global)
                DEPLOY_ENV = 'staging'
            }
            steps {
                echo "App: ${APP_NAME}"
                echo "Deploy to: ${DEPLOY_ENV}"   // → staging
            }
        }
    }
}
```

**Jenkins Built-in Environment Variables:**

| Variable | Value |
|---|---|
| `BUILD_NUMBER` | Current build number (1, 2, 3...) |
| `BUILD_ID` | Build timestamp-based ID |
| `JOB_NAME` | Name of the Jenkins job |
| `BRANCH_NAME` | Current Git branch (multibranch only) |
| `GIT_COMMIT` | Current commit SHA |
| `GIT_BRANCH` | Branch being built |
| `WORKSPACE` | Path to the build workspace directory |
| `JENKINS_URL` | Jenkins instance URL |

### `post` — Post-Build Actions

```groovy
post {
    always {
        // Runs regardless of build result
        cleanWs()
        echo "Build finished with status: ${currentBuild.result}"
    }
    success {
        // Runs only if build succeeded
        archiveArtifacts artifacts: 'dist/**'
        slackSend message: "✅ Deployed successfully!"
    }
    failure {
        // Runs only if build failed
        emailext(
            to: 'team@company.com',
            subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Check console: ${env.BUILD_URL}"
        )
    }
    unstable {
        // Runs if tests passed but with failures (UNSTABLE)
        echo "Tests have some failures — investigate!"
    }
    changed {
        // Runs if build result changed from previous build
        echo "Build status changed!"
    }
    aborted {
        // Runs if build was manually aborted
        echo "Build was aborted."
    }
}
```

### `when` — Conditional Stage Execution

```groovy
stage('Deploy to Production') {
    when {
        // Only run when branch is main
        branch 'main'
    }
    steps {
        sh './deploy-prod.sh'
    }
}

stage('Run on Tag') {
    when {
        // Only run when a tag is pushed
        tag 'v*'
    }
    steps {
        sh './release.sh'
    }
}

stage('Skip on Weekend') {
    when {
        expression {
            // Custom Groovy expression
            return new Date().getDay() != 0 && new Date().getDay() != 6
        }
    }
    steps {
        sh './deploy.sh'
    }
}
```

---

## 9. GitHub Integration

### Step 1: Install Required Plugins

In Jenkins: **Manage Jenkins → Plugins → Available**

Install:
- `Git plugin`
- `GitHub plugin`
- `GitHub Integration plugin`

### Step 2: Configure Jenkins URL

**Manage Jenkins → System → Jenkins Location**

Set Jenkins URL to your public URL (e.g., `http://YOUR_IP:8080/`). This is required for webhooks to work.

### Step 3: Add GitHub Credentials

**Manage Jenkins → Credentials → System → Global → Add Credentials**

**For HTTPS (Personal Access Token):**
- Kind: `Username with password`
- Username: your GitHub username
- Password: your GitHub Personal Access Token (PAT)
- ID: `github-credentials`

**For SSH:**
- Kind: `SSH Username with private key`
- Username: `git`
- Private Key: paste your `~/.ssh/id_rsa` contents
- ID: `github-ssh-credentials`

### Step 4: Generate GitHub Personal Access Token

1. GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
2. Generate new token → Select scopes:
   - `repo` (full repo access)
   - `admin:repo_hook` (for webhooks)
3. Copy the token — you'll only see it once

### Step 5: Set Up Webhook in GitHub

1. Go to your GitHub repository
2. **Settings → Webhooks → Add webhook**
3. Set:
   - **Payload URL:** `http://YOUR_JENKINS_IP:8080/github-webhook/`
   - **Content type:** `application/json`
   - **Secret:** (optional but recommended — add to Jenkins later)
   - **Events:** Select "Just the push event" (or customize)
4. Click **Add webhook**

Test: GitHub will send a ping event. Check for a green checkmark.

### Step 6: Configure Job to Use Webhook

In your Pipeline job: **Configure → Build Triggers**

Check: ✅ **"GitHub hook trigger for GITScm polling"**

### Step 7: Pipeline with GitHub SCM

```groovy
pipeline {
    agent any

    triggers {
        // Poll SCM every 5 minutes (fallback if webhook fails)
        // pollSCM('H/5 * * * *')

        // Trigger on GitHub webhook push
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                // Use credentials to checkout private repo
                git(
                    url: 'https://github.com/yourorg/yourrepo.git',
                    branch: 'main',
                    credentialsId: 'github-credentials'
                )
            }
        }

        stage('Build') {
            steps {
                sh 'npm ci && npm run build'
            }
        }
    }
}
```

### Step 8: Full Webhook Test Flow

```
1. Make a commit to your repo
       ↓
2. GitHub sends POST to http://jenkins:8080/github-webhook/
       ↓
3. Jenkins receives it, queues the build
       ↓
4. Build runs automatically
       ↓
5. Check Jenkins dashboard — build should appear
```

---

## 10. Jenkins + Docker

### Why Docker in Jenkins Pipelines?

| Problem Without Docker | Solution With Docker |
|---|---|
| "Works on my machine" | Same container image everywhere |
| Conflicting Node/Java versions on agents | Each pipeline uses its own container |
| Agents need manual dependency installation | Pull any image on demand |
| Testing across multiple OS versions | Matrix build with different base images |
| Slow environment setup | Pre-built images start in seconds |

### Step 1: Install Docker on Jenkins Agent

```bash
# Ubuntu
sudo apt update
sudo apt install -y docker.io

# Add jenkins user to docker group (so Jenkins can run docker without sudo)
sudo usermod -aG docker jenkins

# Restart Jenkins to pick up group change
sudo systemctl restart jenkins

# Verify
docker --version
```

### Step 2: Install Docker Pipeline Plugin

**Manage Jenkins → Plugins → Available**
- Install: `Docker Pipeline`
- Install: `Docker plugin`

### Step 3: Use Docker as the Build Environment

This is the cleanest pattern — your pipeline runs entirely inside a Docker container:

```groovy
pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            // Mount npm cache for faster builds
            args '-v npm-cache:/root/.npm'
        }
    }

    stages {
        stage('Install') {
            steps {
                sh 'node --version'   // Uses Node 18 from the container
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
    // Container is automatically destroyed after pipeline completes
}
```

### Step 4: Build and Push Docker Images in Pipeline

```groovy
pipeline {
    agent any

    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'youruser/yourapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the image
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                // Run tests inside the freshly built container
                sh "docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} npm test"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Cleanup Local Images') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                sh "docker rmi ${IMAGE_NAME}:latest || true"
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
```

### Dockerfile Example (for the app being built)

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Production image
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json .
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

### Using Docker Compose in Pipelines

```groovy
stage('Integration Tests') {
    steps {
        // Spin up app + database together for integration testing
        sh 'docker-compose -f docker-compose.test.yml up -d'
        sh 'sleep 10'  // Wait for services to be ready
        sh 'npm run test:integration'
        sh 'docker-compose -f docker-compose.test.yml down'
    }
}
```

---

## 11. Jenkins Without Docker

For traditional builds using tools installed directly on the agent system.

### Tool Configuration

**Manage Jenkins → Tools** — configure paths for:
- JDK installations
- Node.js installations (via NodeJS plugin)
- Maven installations
- Gradle installations

### Pipeline Using Installed Tools

```groovy
pipeline {
    agent { label 'linux-build-server' }

    tools {
        nodejs 'NodeJS-18'      // Name configured in Manage Jenkins → Tools
        jdk 'JDK-21'
    }

    stages {
        stage('Build') {
            steps {
                sh 'node --version'
                sh 'npm ci'
                sh 'npm run build'
            }
        }
    }
}
```

### Maven Project (Java)

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk 'JDK-21'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}
```

---

## 12. Credentials Management

### Why Credentials Management Matters

**NEVER do this:**

```groovy
// 🚨 EXTREMELY DANGEROUS — commits your secrets to Git history
sh 'docker login -u myuser -p MySuperSecretPassword123'
sh 'scp -i /home/jenkins/.ssh/mykey.pem file.txt ec2-user@1.2.3.4:/app/'
```

If your Jenkinsfile is in a public repo, your credentials are now public. Even in a private repo, any developer who can read the Jenkinsfile can steal your secrets.

### Types of Credentials in Jenkins

| Type | Use Case |
|---|---|
| **Username with password** | Docker Hub, npm registry, database |
| **Secret text** | API keys, tokens, environment variables |
| **SSH Username with private key** | SSH access to servers, Git over SSH |
| **Certificate** | Java KeyStore, PKCS#12 certificates |
| **Secret file** | `.env` files, Kubernetes kubeconfig |

### Adding Credentials

**Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

Fill in:
- **Kind:** Select the type
- **Scope:** Global (accessible everywhere) or System (Jenkins-only)
- **ID:** A unique identifier you'll reference in Jenkinsfiles (e.g., `dockerhub-credentials`)
- **Description:** Human-readable label

### Using Credentials in Pipelines

#### Method 1: `withCredentials` Block (Most Secure)

```groovy
stage('Deploy') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-credentials',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            ),
            string(
                credentialsId: 'slack-token',
                variable: 'SLACK_TOKEN'
            ),
            sshUserPrivateKey(
                credentialsId: 'prod-server-ssh',
                keyFileVariable: 'SSH_KEY',
                usernameVariable: 'SSH_USER'
            )
        ]) {
            sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                ssh -i $SSH_KEY $SSH_USER@production-server "docker pull myapp:latest"
            '''
        }
        // Credentials are automatically masked in logs and cleaned up after the block
    }
}
```

#### Method 2: `environment` Block with `credentials()`

```groovy
pipeline {
    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        // Jenkins automatically creates:
        //   DOCKERHUB_CREDS_USR → username
        //   DOCKERHUB_CREDS_PSW → password
    }

    stages {
        stage('Push') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
            }
        }
    }
}
```

### Credentials Are Masked in Logs

Jenkins automatically masks credential values in console output:

```
+ docker login -u myuser --password-stdin
****   ← password is masked, never shown in logs
Login Succeeded
```

### Using `.env` Files as Secret Files

```groovy
withCredentials([file(credentialsId: 'app-env-file', variable: 'ENV_FILE')]) {
    sh 'cp $ENV_FILE .env'
    sh 'npm start &'
    sh 'rm -f .env'    // Clean up sensitive file
}
```

---

## 13. Artifacts Management

### What Are Artifacts?

Artifacts are the **output files** produced by a build — compiled binaries, JAR files, Docker images, test reports, coverage reports, deployment packages. Jenkins can store these and make them downloadable from the job's build page.

### Archiving Artifacts

```groovy
stage('Build') {
    steps {
        sh 'npm run build'
        sh 'tar -czf app-${BUILD_NUMBER}.tar.gz dist/'
    }
    post {
        success {
            // Archive the build output
            archiveArtifacts(
                artifacts: 'app-*.tar.gz, dist/**/*.js',
                fingerprint: true,          // Generate MD5 fingerprint
                allowEmptyArchive: false    // Fail if no artifacts found
            )
        }
    }
}
```

### Stash and Unstash (Between Stages/Nodes)

When different stages run on different agents, use `stash`/`unstash` to pass files between them:

```groovy
stage('Build') {
    agent { label 'build-server' }
    steps {
        sh 'npm ci && npm run build'
        // Stash the built files
        stash name: 'build-output', includes: 'dist/**/*'
    }
}

stage('Deploy') {
    agent { label 'deploy-server' }
    steps {
        // Retrieve the stashed files on a different agent
        unstash 'build-output'
        sh 'rsync -av dist/ /var/www/html/'
    }
}
```

### Publishing Test Reports

```groovy
post {
    always {
        // JUnit test results (most CI systems understand this format)
        junit allowEmptyResults: true, testResults: 'test-results/**/*.xml'

        // Code coverage (requires HTML Publisher plugin)
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'coverage/lcov-report',
            reportFiles: 'index.html',
            reportName: 'Coverage Report'
        ])
    }
}
```

---

## 14. Parallel Execution

### Why Parallel?

Running stages sequentially when they don't depend on each other wastes time. For example, running unit tests, linting, and security scans can happen simultaneously.

```
Sequential (slow):     Lint(2m) → Tests(5m) → Security(3m) → TOTAL: 10m
Parallel (fast):       Lint ──┐
                       Tests──┼──────────────── TOTAL: 5m
                       Security─┘
```

### Basic Parallel Stages

```groovy
pipeline {
    agent none

    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
                stash name: 'source', includes: '**/*'
            }
        }

        stage('Parallel Quality Checks') {
            parallel {
                stage('Unit Tests') {
                    agent { docker { image 'node:18-alpine' } }
                    steps {
                        unstash 'source'
                        sh 'npm ci && npm test'
                    }
                    post {
                        always {
                            junit 'test-results/*.xml'
                        }
                    }
                }

                stage('Lint') {
                    agent { docker { image 'node:18-alpine' } }
                    steps {
                        unstash 'source'
                        sh 'npm ci && npm run lint'
                    }
                }

                stage('Security Scan') {
                    agent { docker { image 'node:18-alpine' } }
                    steps {
                        unstash 'source'
                        sh 'npm audit --audit-level=high'
                    }
                }

                stage('Build') {
                    agent { docker { image 'node:18-alpine' } }
                    steps {
                        unstash 'source'
                        sh 'npm ci && npm run build'
                        stash name: 'build-output', includes: 'dist/**'
                    }
                }
            }
        }

        stage('Deploy') {
            agent any
            steps {
                unstash 'build-output'
                sh './deploy.sh'
            }
        }
    }
}
```

### Parallel with failFast

```groovy
stage('Tests') {
    failFast true    // If any parallel branch fails, abort all others immediately
    parallel {
        stage('Chrome Tests') {
            steps { sh 'npm run test:chrome' }
        }
        stage('Firefox Tests') {
            steps { sh 'npm run test:firefox' }
        }
        stage('Safari Tests') {
            steps { sh 'npm run test:safari' }
        }
    }
}
```

---

## 15. Multi-Branch Pipelines

### What is a Multi-Branch Pipeline?

A Multi-Branch Pipeline job **automatically discovers branches** in your repository and creates a separate pipeline for each branch that contains a `Jenkinsfile`.

```
Repository:
├── main         → Jenkins Pipeline: "myapp/main"
├── develop      → Jenkins Pipeline: "myapp/develop"
├── feature/auth → Jenkins Pipeline: "myapp/feature/auth"
└── feature/cart → Jenkins Pipeline: "myapp/feature/cart"
```

Every branch builds independently. When you delete a branch, Jenkins can automatically delete its pipeline.

### Creating a Multi-Branch Pipeline

1. **New Item → Multibranch Pipeline**
2. **Branch Sources:** Add source → GitHub
   - Credentials: select your GitHub credentials
   - Repository URL: your repo URL
3. **Behaviors:**
   - Discover branches: All branches
   - Filter by name (optional): Use regex like `main|develop|feature/.*`
4. **Build Configuration:** By Jenkinsfile
5. **Scan Multibranch Pipeline Triggers:** Periodically (or use webhooks)
6. Click **Save** — Jenkins immediately scans and creates pipelines for all branches

### Branch-Specific Logic in Jenkinsfile

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'npm ci && npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Deploy to Staging') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'release/*'
                }
            }
            steps {
                echo "Deploying ${BRANCH_NAME} to Staging"
                sh './scripts/deploy-staging.sh'
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                // Require manual approval before production deploy
                message "Deploy ${BUILD_NUMBER} to Production?"
                ok "Yes, Deploy!"
                submitter "admin,lead-dev"
            }
            steps {
                echo "Deploying to Production!"
                sh './scripts/deploy-prod.sh'
            }
        }

        stage('Feature Branch Notice') {
            when {
                branch 'feature/*'
            }
            steps {
                echo "Feature branch build — no deployment"
            }
        }
    }
}
```

---

## 16. Deployment Strategies

### 16.1 SSH Deployment

Deploy to a remote server over SSH.

```groovy
stage('Deploy via SSH') {
    steps {
        withCredentials([sshUserPrivateKey(
            credentialsId: 'prod-server-ssh-key',
            keyFileVariable: 'SSH_KEY',
            usernameVariable: 'SSH_USER'
        )]) {
            sh '''
                # Copy the build output to the server
                scp -i $SSH_KEY -o StrictHostKeyChecking=no \
                    app-${BUILD_NUMBER}.tar.gz \
                    $SSH_USER@prod-server:/opt/deploy/

                # Run deployment commands on the remote server
                ssh -i $SSH_KEY -o StrictHostKeyChecking=no $SSH_USER@prod-server << 'REMOTE'
                    cd /opt/deploy
                    tar -xzf app-${BUILD_NUMBER}.tar.gz
                    pm2 restart myapp || pm2 start ecosystem.config.js
                    rm -f app-*.tar.gz
                REMOTE
            '''
        }
    }
}
```

### 16.2 Docker Deployment

```groovy
stage('Deploy with Docker') {
    steps {
        withCredentials([sshUserPrivateKey(
            credentialsId: 'prod-ssh-key',
            keyFileVariable: 'SSH_KEY',
            usernameVariable: 'SSH_USER'
        )]) {
            sh """
                ssh -i $SSH_KEY $SSH_USER@prod-server '
                    docker pull ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker stop myapp || true
                    docker rm myapp || true
                    docker run -d \
                        --name myapp \
                        --restart unless-stopped \
                        -p 3000:3000 \
                        --env-file /etc/myapp/.env \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker image prune -f
                '
            """
        }
    }
}
```

### 16.3 Kubernetes Deployment (Basic)

```groovy
stage('Deploy to Kubernetes') {
    steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh """
                export KUBECONFIG=$KUBECONFIG

                # Update the image in the deployment
                kubectl set image deployment/myapp \
                    myapp=${IMAGE_NAME}:${BUILD_NUMBER} \
                    --namespace=production

                # Wait for rollout to complete
                kubectl rollout status deployment/myapp \
                    --namespace=production \
                    --timeout=300s

                # Verify pods are running
                kubectl get pods --namespace=production
            """
        }
    }
}
```

---

## 17. Jenkins + Kubernetes (Advanced)

### Why Run Jenkins Agents in Kubernetes?

| Static Agent Problem | Kubernetes Solution |
|---|---|
| Agents sit idle between builds (waste money) | Pods are created on demand, destroyed after build |
| Fixed number of agents limits concurrency | Auto-scales: 100 builds? 100 pods. |
| Agent configuration drift over time | Fresh pod from image every build |
| Different projects need different tools | Each pipeline specifies its own pod spec |

### Setup: Kubernetes Plugin

**Manage Jenkins → Plugins → Install:** `Kubernetes`

**Manage Jenkins → Clouds → Add a new cloud → Kubernetes**

Configure:
- **Kubernetes URL:** `https://kubernetes.default.svc` (if Jenkins is inside the cluster)
- **Jenkins URL:** `http://jenkins.jenkins.svc.cluster.local:8080`
- **Namespace:** `jenkins`

### Pipeline with Kubernetes Agents

```groovy
pipeline {
    agent {
        kubernetes {
            label 'mypod'
            defaultContainer 'node'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: node:18-alpine
    command:
    - sleep
    args:
    - 9999999
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  - name: docker
    image: docker:dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }

    stages {
        stage('Build App') {
            container('node') {
                steps {
                    sh 'npm ci && npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            container('docker') {
                steps {
                    sh "docker build -t myapp:${BUILD_NUMBER} ."
                }
            }
        }
    }
}
```

---

## 18. Logs & Debugging

### Accessing Console Output

**Job → Build Number → Console Output**

This is your primary debugging tool. Read it carefully from top to bottom when a build fails.

### Understanding Log Output

```
[Pipeline] Start of Pipeline           ← Pipeline started
[Pipeline] agent                       ← Allocating agent
[Pipeline] stage (Checkout)            ← Stage starting
+ git clone https://github.com/...    ← Actual command (+ prefix)
HEAD is now at a1b2c3d                 ← Command output
[Pipeline] stage (Install)
+ npm ci                               ← Command being run
npm warn deprecated pkg               ← npm output
added 1247 packages in 23.4s
[Pipeline] stage (Test)
+ npm test
FAIL src/app.test.js                   ← Test failure
  ● Test › should return 200
    Expected: 200
    Received: 404                      ← The actual error
ERROR: script returned exit code 1     ← Jenkins sees non-zero exit
Finished: FAILURE                      ← Build marked as failed
```

### Common Errors and Fixes

**Error: `Permission denied`**
```bash
# Jenkins can't execute a file
# Fix: add execute permission
chmod +x your-script.sh

# Or in Jenkinsfile
sh 'chmod +x deploy.sh && ./deploy.sh'
```

**Error: `docker: command not found`**
```bash
# Docker not installed on agent, or jenkins user not in docker group
sudo apt install docker.io
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

**Error: `stderr: Permission denied (publickey)`**
```bash
# SSH key not configured correctly
# Check: credentialsId matches exactly
# Check: public key is in ~/.ssh/authorized_keys on target server
# Check: known_hosts — add StrictHostKeyChecking=no for first time
```

**Error: `No such DSL method 'docker'`**
```
# Docker Pipeline plugin is not installed
# Install: Manage Jenkins → Plugins → Docker Pipeline
```

**Error: `Could not find credentials with ID 'my-creds'`**
```
# Credential ID doesn't match
# Check: Manage Jenkins → Credentials → verify the exact ID
# IDs are case-sensitive!
```

**Error: Workspace full / disk space**
```groovy
// Add to pipeline post section
post {
    always {
        cleanWs()
    }
}

// Or configure Discard Old Builds in job settings:
// Keep max 10 builds, discard builds older than 30 days
properties([
    buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '30'))
])
```

### Debug Mode in Pipeline

```groovy
stage('Debug') {
    steps {
        // Print all environment variables
        sh 'env | sort'

        // Print working directory and contents
        sh 'pwd && ls -la'

        // Check who Jenkins is running as
        sh 'whoami && id'

        // Check available disk space
        sh 'df -h'

        // Check network connectivity
        sh 'curl -s https://registry.hub.docker.com/v2/ | head'
    }
}
```

---

## 19. Security Best Practices

### 19.1 Authentication — Who Can Log In

**Manage Jenkins → Security → Security Realm**

Options:
- **Jenkins' own user database** — Simple, good for small teams. Create users in Jenkins.
- **LDAP** — Integrates with Active Directory. Ideal for enterprise.
- **GitHub OAuth** — Users log in with their GitHub account. Great for developer teams.
- **SAML 2.0** — Enterprise SSO (Okta, Azure AD).

```
Install plugin: GitHub OAuth
Configure:
  Client ID: (from GitHub OAuth App)
  Client Secret: (from GitHub OAuth App)
  → Users authenticate via GitHub
```

### 19.2 Authorization — What Can They Do

**Manage Jenkins → Security → Authorization**

**Role-Based Strategy (install "Role-based Authorization Strategy" plugin):**

```
Roles:
  admin     → Full access
  developer → Create/build jobs in their team folder
  viewer    → Read-only access to job status and logs
  deploy    → Can trigger deploy jobs but not create/delete
```

**Matrix-based security:**

| | admin | dev | viewer |
|---|---|---|---|
| Overall/Administer | ✅ | ❌ | ❌ |
| Job/Create | ✅ | ✅ | ❌ |
| Job/Build | ✅ | ✅ | ❌ |
| Job/Read | ✅ | ✅ | ✅ |
| Job/Delete | ✅ | ❌ | ❌ |

### 19.3 Credentials Safety Rules

```
✅ DO:
  - Store all secrets in Jenkins Credentials Manager
  - Use withCredentials{} blocks
  - Set credentials scope to minimum required
  - Rotate credentials regularly
  - Use SSH keys instead of passwords where possible
  - Use secret text/file types for API keys

❌ DON'T:
  - Put passwords in Jenkinsfile (even base64 encoded)
  - Put passwords in environment variables via System settings (visible in UI)
  - Log credential values (Jenkins masks them, but don't try to print them)
  - Share admin credentials between humans and automation
  - Use the Jenkins admin account for pipeline operations
```

### 19.4 Additional Hardening

```bash
# Keep Jenkins updated
sudo apt update && sudo apt upgrade jenkins

# Disable CLI over remoting (legacy security issue)
# Manage Jenkins → Security → Disable CLI over Remoting

# Enable CSRF Protection (default in modern Jenkins)
# Manage Jenkins → Security → CSRF Protection → Enable

# Set up reverse proxy with HTTPS (never expose Jenkins on plain HTTP)
# Use Nginx with Let's Encrypt SSL in front of Jenkins

# Restrict which jobs can run on controller
# Manage Jenkins → Nodes → controller → Number of executors → 0
```

---

## 20. Real-World CI/CD Project — Full MERN Pipeline

### Project: ShellELearning Academy — MERN Stack App

**Stack:** MongoDB + Express.js + React + Node.js  
**Repository structure:**
```
shellelearning/
├── client/          (React frontend)
├── server/          (Node.js/Express backend)
├── Dockerfile
├── docker-compose.yml
├── .env.example
└── Jenkinsfile      ← Our pipeline
```

### Prerequisites Checklist

- [ ] Jenkins running with Docker access
- [ ] GitHub repo with `Jenkinsfile` at root
- [ ] Credentials configured in Jenkins:
  - `github-credentials` — GitHub PAT
  - `dockerhub-credentials` — Docker Hub username+password
  - `prod-server-ssh` — SSH key to production server
  - `app-env-file` — Secret `.env` file for the app
- [ ] Webhook configured on GitHub
- [ ] Docker Hub repository created: `youruser/shellelearning`

### Complete Jenkinsfile

```groovy
pipeline {
    agent any

    // Discard old builds to save disk space
    options {
        buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '30'))
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        APP_NAME        = 'shellelearning'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE    = "youruser/${APP_NAME}"
        DOCKER_TAG      = "${env.BUILD_NUMBER}"
        PROD_SERVER     = '123.45.67.89'
        PROD_USER       = 'ubuntu'
        NODE_ENV        = 'test'
    }

    stages {

        // ─────────────────────────────────────────────
        stage('Checkout') {
        // ─────────────────────────────────────────────
            steps {
                echo "🔄 Checking out branch: ${env.BRANCH_NAME ?: 'main'}"
                checkout scm
                sh 'git log --oneline -5'  // Show recent commits
            }
        }

        // ─────────────────────────────────────────────
        stage('Install & Quality Checks') {
        // ─────────────────────────────────────────────
            parallel {
                stage('Backend: Install') {
                    agent {
                        docker { image 'node:18-alpine' }
                    }
                    steps {
                        dir('server') {
                            sh 'npm ci'
                            sh 'npm run lint || echo "Lint warnings found"'
                        }
                    }
                }

                stage('Frontend: Install') {
                    agent {
                        docker { image 'node:18-alpine' }
                    }
                    steps {
                        dir('client') {
                            sh 'npm ci'
                            sh 'npm run lint || echo "Lint warnings found"'
                        }
                    }
                }
            }
        }

        // ─────────────────────────────────────────────
        stage('Run Tests') {
        // ─────────────────────────────────────────────
            parallel {
                stage('Backend Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                        }
                    }
                    steps {
                        dir('server') {
                            sh 'npm ci'
                            sh 'npm test -- --coverage --forceExit'
                        }
                    }
                    post {
                        always {
                            junit allowEmptyResults: true,
                                  testResults: 'server/test-results/*.xml'
                        }
                    }
                }

                stage('Frontend Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                        }
                    }
                    steps {
                        dir('client') {
                            sh 'npm ci'
                            sh 'npm test -- --watchAll=false --coverage'
                        }
                    }
                    post {
                        always {
                            junit allowEmptyResults: true,
                                  testResults: 'client/test-results/*.xml'
                        }
                    }
                }
            }
        }

        // ─────────────────────────────────────────────
        stage('Build React Frontend') {
        // ─────────────────────────────────────────────
            agent {
                docker { image 'node:18-alpine' }
            }
            steps {
                dir('client') {
                    sh 'npm ci'
                    sh 'npm run build'
                    // Stash the built frontend for Docker build stage
                    stash name: 'frontend-build', includes: 'build/**/*'
                }
            }
        }

        // ─────────────────────────────────────────────
        stage('Build Docker Image') {
        // ─────────────────────────────────────────────
            steps {
                // Retrieve stashed frontend build
                unstash 'frontend-build'
                sh 'mv build client/build'

                script {
                    sh """
                        docker build \
                            --build-arg NODE_ENV=production \
                            --label "build.number=${BUILD_NUMBER}" \
                            --label "git.commit=${env.GIT_COMMIT}" \
                            -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                            -t ${DOCKER_IMAGE}:latest \
                            .
                    """
                }
                echo "✅ Docker image built: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }

        // ─────────────────────────────────────────────
        stage('Security Scan') {
        // ─────────────────────────────────────────────
            steps {
                // Scan for vulnerabilities in npm packages
                dir('server') {
                    sh 'npm audit --audit-level=critical || true'
                }
                dir('client') {
                    sh 'npm audit --audit-level=critical || true'
                }
                // Optionally: scan Docker image with Trivy
                // sh "docker run --rm aquasec/trivy image ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }

        // ─────────────────────────────────────────────
        stage('Push to Docker Hub') {
        // ─────────────────────────────────────────────
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                    echo "✅ Pushed ${DOCKER_IMAGE}:${DOCKER_TAG} to Docker Hub"
                }
            }
        }

        // ─────────────────────────────────────────────
        stage('Deploy to Staging') {
        // ─────────────────────────────────────────────
            when {
                branch 'develop'
            }
            steps {
                echo "🚀 Deploying to Staging..."
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'staging-server-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \$SSH_USER@staging-server '
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop ${APP_NAME}-staging || true
                            docker rm ${APP_NAME}-staging || true
                            docker run -d \\
                                --name ${APP_NAME}-staging \\
                                --restart unless-stopped \\
                                -p 3001:3000 \\
                                --env-file /etc/shellelearning/.env.staging \\
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '
                    """
                }
                echo "✅ Staging deployed at: http://staging.shellelearning.com"
            }
        }

        // ─────────────────────────────────────────────
        stage('Approval Gate') {
        // ─────────────────────────────────────────────
            when {
                branch 'main'
            }
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input(
                        message: "Deploy build #${BUILD_NUMBER} to PRODUCTION?",
                        ok: "Yes, Deploy to Production!",
                        submitter: "admin,senior-dev",
                        parameters: [
                            choice(
                                name: 'DEPLOY_TYPE',
                                choices: ['rolling', 'blue-green'],
                                description: 'Choose deployment strategy'
                            )
                        ]
                    )
                }
            }
        }

        // ─────────────────────────────────────────────
        stage('Deploy to Production') {
        // ─────────────────────────────────────────────
            when {
                branch 'main'
            }
            steps {
                echo "🚀 Deploying to Production..."
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'prod-server-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    ),
                    file(
                        credentialsId: 'app-env-file',
                        variable: 'ENV_FILE'
                    )
                ]) {
                    sh """
                        # Copy env file to server
                        scp -i \$SSH_KEY -o StrictHostKeyChecking=no \
                            \$ENV_FILE \$SSH_USER@\$PROD_SERVER:/etc/shellelearning/.env

                        # Pull and run new container
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \$SSH_USER@\$PROD_SERVER '
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}

                            # Zero-downtime: start new container first
                            docker run -d \\
                                --name ${APP_NAME}-new \\
                                -p 3002:3000 \\
                                --env-file /etc/shellelearning/.env \\
                                ${DOCKER_IMAGE}:${DOCKER_TAG}

                            # Wait for health check
                            sleep 10
                            curl -f http://localhost:3002/api/health || exit 1

                            # Swap: stop old, rename new
                            docker stop ${APP_NAME} || true
                            docker rm ${APP_NAME} || true
                            docker rename ${APP_NAME}-new ${APP_NAME}
                            docker update --restart unless-stopped ${APP_NAME}

                            # Expose on main port via nginx reload
                            sudo nginx -s reload

                            # Cleanup old images
                            docker image prune -f
                        '
                    """
                }
                echo "✅ Production deployed: https://shellelearningacademy.com"
            }
        }

    }

    // ─────────────────────────────────────────────
    post {
    // ─────────────────────────────────────────────
        always {
            // Cleanup
            sh 'docker logout || true'
            sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
            cleanWs()

            // Report build duration
            echo "Build duration: ${currentBuild.durationString}"
        }

        success {
            echo "✅ Pipeline succeeded!"
            // Uncomment to enable Slack notifications:
            // slackSend(
            //     channel: '#deployments',
            //     color: 'good',
            //     message: "✅ *${APP_NAME}* build #${BUILD_NUMBER} deployed successfully!\n" +
            //              "Branch: ${BRANCH_NAME} | Duration: ${currentBuild.durationString}"
            // )
        }

        failure {
            echo "❌ Pipeline failed!"
            // emailext(
            //     to: 'devteam@shellelearning.com',
            //     subject: "❌ Build Failed: ${APP_NAME} #${BUILD_NUMBER}",
            //     body: """
            //         Build ${BUILD_NUMBER} failed on branch ${BRANCH_NAME}.
            //         Console: ${BUILD_URL}console
            //         Please investigate immediately.
            //     """
            // )
        }
    }
}
```

### Supporting Dockerfile

```dockerfile
FROM node:18-alpine AS base
WORKDIR /app

# Install backend dependencies
FROM base AS backend-deps
COPY server/package*.json ./server/
RUN cd server && npm ci --only=production

# Build frontend (already built, just copy)
FROM base AS production
COPY --from=backend-deps /app/server/node_modules ./server/node_modules
COPY server/ ./server/
COPY client/build/ ./client/build/

WORKDIR /app/server
EXPOSE 3000
ENV NODE_ENV=production
USER node
CMD ["node", "index.js"]
```

---

## 21. Advanced Concepts

### 21.1 Shared Libraries

Shared Libraries let you **extract common pipeline code** into a centralized repository, reusable across all your pipelines. Instead of copy-pasting the same `dockerBuild` + `dockerPush` logic across 50 pipelines, you define it once.

**Library repository structure:**
```
jenkins-shared-library/
├── vars/
│   ├── dockerBuild.groovy      ← Pipeline step
│   ├── dockerPush.groovy
│   ├── deployToServer.groovy
│   └── sendSlackNotification.groovy
└── src/
    └── com/company/
        └── Utils.groovy        ← Helper classes
```

**`vars/dockerBuild.groovy`:**
```groovy
def call(String imageName, String tag = 'latest') {
    sh "docker build -t ${imageName}:${tag} ."
    sh "docker tag ${imageName}:${tag} ${imageName}:latest"
    echo "Built ${imageName}:${tag}"
}
```

**Configure library in Jenkins:**
**Manage Jenkins → System → Global Pipeline Libraries**
- Name: `company-shared-lib`
- Default version: `main`
- Source: GitHub repo URL

**Using the library in a Jenkinsfile:**
```groovy
@Library('company-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Use the shared step!
                dockerBuild('myapp', env.BUILD_NUMBER)
            }
        }
        stage('Push') {
            steps {
                dockerPush('myapp', env.BUILD_NUMBER, 'dockerhub-credentials')
            }
        }
    }
}
```

### 21.2 Pipeline as Code Principles

- **Jenkinsfile in Git** — Every pipeline change is a commit. Reviewable. Reversible. Auditable.
- **Immutable artifacts** — Build once, deploy the same artifact to all environments
- **Environment parity** — Staging should mirror production as closely as possible
- **Idempotent deployments** — Running deploy twice should produce the same result

### 21.3 Infrastructure as Code with Jenkins

Jenkins can trigger IaC tools:

```groovy
stage('Provision Infrastructure') {
    steps {
        dir('infrastructure/terraform') {
            withCredentials([string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                             string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh 'terraform init'
                sh 'terraform plan -out=tfplan'
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }
}

stage('Configure with Ansible') {
    steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ansible-ssh', keyFileVariable: 'KEY')]) {
            sh 'ansible-playbook -i inventory.yml --private-key=$KEY playbooks/deploy.yml'
        }
    }
}
```

### 21.4 Blue-Green Deployment

Blue-Green deployment eliminates downtime by running two identical production environments:

```
PRODUCTION:
  ┌─────────────┐     ┌─────────────┐
  │  BLUE env   │     │  GREEN env  │
  │  (current)  │     │  (new)      │
  │  v1.0 live  │     │  v1.1 ready │
  └──────┬──────┘     └──────┬──────┘
         │                   │
         └─────────┬─────────┘
                   │
            Load Balancer / Router
                   │
                 Users
```

Deployment process:
1. Green is running current (v1.0), Blue is idle
2. Deploy new version to Blue
3. Run smoke tests against Blue
4. Switch load balancer from Green → Blue (zero downtime)
5. Old Green environment kept for quick rollback
6. Next deploy: deploy to Green, switch from Blue → Green

```groovy
stage('Blue-Green Deploy') {
    steps {
        sh """
            # Deploy to blue
            docker run -d --name app-blue -p 3001:3000 myapp:${BUILD_NUMBER}

            # Health check on blue
            sleep 10
            curl -f http://localhost:3001/health

            # Switch nginx upstream from green to blue
            sed -i 's/3000/3001/' /etc/nginx/conf.d/upstream.conf
            nginx -s reload

            # Stop old green
            docker stop app-green && docker rm app-green
            docker rename app-blue app-green
        """
    }
}
```

---

## 22. Common Mistakes

### 22.1 Beginner Mistakes

| Mistake | Why It's Wrong | Fix |
|---|---|---|
| Hardcoding passwords in Jenkinsfile | Secrets exposed in Git history forever | Use Jenkins Credentials Manager |
| Running everything as root | Any exploit = full system compromise | Use dedicated service users |
| Using `chmod 777` on workspace | Everyone can modify build files | Use targeted permissions |
| Not cleaning workspace | Disk fills up, stale artifacts cause bugs | Add `cleanWs()` in `post { always }` |
| Ignoring failed builds | Broken pipelines become the norm | Enforce "fix broken builds first" policy |
| Not archiving test results | Can't track test trends over time | Always publish JUnit/test reports |
| Using `sleep` instead of health checks | Fragile, slow | Use retry loops with real health checks |
| No build timeout | Hung builds block the queue forever | Set `timeout(time: 30, unit: 'MINUTES')` |

### 22.2 Intermediate Mistakes

| Mistake | Why It's Wrong | Fix |
|---|---|---|
| Running builds on the controller | Controller overload, security risk | Use dedicated agents |
| `chmod -R 777` on web directories | Massive security vulnerability | Use `644` for files, `755` for dirs |
| Not using `--password-stdin` for docker login | Password visible in process list | Always use `--password-stdin` |
| Not setting `agent none` for parallel stages | All stages fight for same executor | Use `agent none` + per-stage agents |
| Rebuilding Docker images on every stage | Wastes time, creates inconsistency | Build once, stash/push, reuse |
| Not pinning Docker image versions | `node:latest` breaks when it updates | Use `node:18.17.1-alpine` |
| Storing secrets in `environment {}` | Visible in Jenkins UI under job config | Use `withCredentials{}` block |
| Ignoring Jenkins logs | Misses warnings about deprecated features | Review logs weekly |

---

## 23. Interview Questions & Answers

### Beginner Level

**Q1: What is Jenkins and what is it used for?**

> Jenkins is an open-source automation server written in Java. It's used to automate the build, test, and deployment lifecycle of software applications — the core of CI/CD. It watches your source code repository and automatically runs predefined workflows (pipelines) whenever changes are detected.

**Q2: What is a Jenkinsfile?**

> A Jenkinsfile is a text file written in Groovy DSL that defines a Jenkins pipeline as code. It lives in the root of your repository alongside application code, making pipeline configuration version-controlled, reviewable, and auditable.

**Q3: What is the difference between Freestyle and Pipeline jobs?**

> Freestyle jobs are GUI-based with configuration stored in Jenkins' database — not version controlled and limited in capability. Pipeline jobs use a Jenkinsfile stored in your repository — version controlled, code-reviewable, supports complex workflows (parallel stages, conditionals, loops), and is the industry standard.

**Q4: What is CI/CD?**

> CI (Continuous Integration) is the practice of frequently merging code changes with automated builds and tests. CD can mean Continuous Delivery (automated preparation for release, human approves deployment) or Continuous Deployment (fully automated — every passing build auto-deploys to production). Together they enable teams to ship software faster with higher confidence.

**Q5: How do you trigger a Jenkins build automatically?**

> Using webhooks: configure GitHub to send an HTTP POST to `http://jenkins:8080/github-webhook/` on every push. Jenkins receives the webhook and starts the build immediately. This is faster and more reliable than polling.

### Intermediate Level

**Q6: What are Jenkins agents and why are they needed?**

> Agents (nodes) are machines that execute pipeline steps on behalf of the Jenkins controller. They're needed because: (1) the controller should only orchestrate, not build; (2) different projects need different environments; (3) parallel builds require multiple machines; (4) agents can auto-scale on demand (Kubernetes).

**Q7: Explain `withCredentials` and why it should be used.**

> `withCredentials` is a Jenkins pipeline step that securely injects credentials into the pipeline scope. It: automatically masks credential values in logs, cleans up credentials after the block exits, and never exposes secrets as plain text. It's the safest way to use passwords, API keys, and SSH keys in pipelines.

**Q8: What is a Multi-Branch Pipeline?**

> A Multi-Branch Pipeline automatically discovers all branches in a repository and creates separate pipelines for each branch containing a Jenkinsfile. This allows each branch (feature/login, develop, main) to have its own build, test, and optionally deploy workflow — enabling proper GitFlow CI/CD.

**Q9: How do you run parallel stages in Jenkins?**

> Using the `parallel` keyword inside a `stage`. Each nested stage runs on a separate executor simultaneously. Use `failFast: true` to abort all parallel branches if one fails. This is critical for reducing total pipeline time — separate test suites can run concurrently.

**Q10: What is the difference between `sh` and `bat` in pipelines?**

> `sh` runs shell commands on Unix/Linux/Mac agents. `bat` runs batch commands on Windows agents. For cross-platform pipelines, you can use an `isUnix()` check to conditionally run either.

### Advanced Level

**Q11: What are Jenkins Shared Libraries and why are they important?**

> Shared Libraries are centralized Groovy code repositories that define reusable pipeline steps, functions, and classes. They solve the copy-paste problem — instead of duplicating `dockerBuild()` logic across 50 Jenkinsfiles, you define it once in the library and reference it everywhere. When the logic changes, you update one place.

**Q12: How do you implement blue-green deployments with Jenkins?**

> Blue-green deployment in Jenkins: (1) deploy new version to the inactive environment (e.g., Blue); (2) run health checks against Blue; (3) use the `input` step to require manual approval if needed; (4) switch the load balancer to point to Blue; (5) keep Green running for instant rollback. The pipeline scripts load balancer config changes (nginx, HAProxy, AWS ALB target groups) as code.

**Q13: How do you scale Jenkins for hundreds of builds per day?**

> Use Kubernetes agents — pods are created on demand for each build and destroyed afterward. Zero idle agent cost, infinite parallelism (limited only by cluster capacity). Combine with declarative pipelines using `agent { kubernetes { ... } }` spec per stage. Also: use build caching (node_modules cache volumes), optimize Dockerfiles with multi-stage builds, and archive only essential artifacts.

**Q14: What is the `post` section and what conditions can it have?**

> The `post` section defines actions to run after a pipeline or stage completes. Conditions: `always` (always runs), `success` (only on success), `failure` (only on failure), `unstable` (tests passed but some failures), `changed` (result differs from previous build), `aborted` (manually stopped), `cleanup` (runs last, even after other post conditions).

**Q15: How would you implement a compliance gate in a Jenkins pipeline?**

> Add a stage with manual `input` for compliance sign-off: `input(message: 'Compliance review approved?', submitter: 'compliance-team')`. This pauses the pipeline and waits for someone from the compliance team to approve before continuing to the deploy stages. Combine with automated security scans (OWASP ZAP, Trivy, SonarQube) as required gates before the manual approval step.

---

## 24. DevOps Roadmap Using Jenkins

### Learning Path (Structured Week-by-Week)

```
PHASE 1: FOUNDATION (Weeks 1-2)
────────────────────────────────
Week 1:
  ✅ Install Linux (Ubuntu VM or WSL2)
  ✅ Learn Linux basics: files, permissions, processes, networking
  ✅ Git fundamentals: clone, commit, push, branch, merge

Week 2:
  ✅ Install Jenkins (Docker method for local dev)
  ✅ Create first Freestyle job
  ✅ Create first Pipeline job with simple Jenkinsfile
  ✅ Connect a GitHub repository

PHASE 2: CORE JENKINS (Weeks 3-4)
────────────────────────────────
Week 3:
  ✅ Write Declarative Pipelines with all sections
  ✅ Configure Credentials Manager
  ✅ Set up GitHub webhooks for auto-trigger
  ✅ Implement parallel stages

Week 4:
  ✅ Multi-Branch Pipelines
  ✅ Build a complete Node.js CI pipeline
  ✅ Configure agents
  ✅ Understand logs and troubleshoot 3 real errors

PHASE 3: DOCKER INTEGRATION (Weeks 5-6)
────────────────────────────────────────
Week 5:
  ✅ Docker fundamentals: build, run, push, compose
  ✅ Write a production Dockerfile
  ✅ Jenkins pipeline: build + push Docker image to Docker Hub
  ✅ Use Docker as the build environment (agent docker)

Week 6:
  ✅ Docker Compose for test environments
  ✅ Multi-stage Docker builds
  ✅ Full pipeline: GitHub → Test → Docker Build → Push

PHASE 4: DEPLOYMENT (Weeks 7-8)
────────────────────────────────
Week 7:
  ✅ Deploy to a VPS via SSH from Jenkins
  ✅ Configure SSH credentials properly
  ✅ Blue-Green or rolling deployment strategy
  ✅ Implement the complete MERN pipeline from Section 20

Week 8:
  ✅ Jenkins + Nginx reverse proxy with HTTPS
  ✅ Environment-specific deployments (staging vs production)
  ✅ Manual approval gates for production
  ✅ Build the full production-grade pipeline

PHASE 5: ADVANCED DEVOPS (Weeks 9-12)
───────────────────────────────────────
Week 9-10:
  ✅ Kubernetes basics: pods, deployments, services
  ✅ Deploy Jenkins agents in Kubernetes
  ✅ kubectl deployments from Jenkins

Week 11-12:
  ✅ Terraform basics + Jenkins-triggered IaC
  ✅ Ansible for configuration management
  ✅ Monitoring: Prometheus + Grafana
  ✅ Jenkins Shared Libraries

PHASE 6: JOB-READY (Ongoing)
──────────────────────────────
  ✅ Build a real project and put it on GitHub (your portfolio)
  ✅ Write a blog post / documentation about your pipeline
  ✅ Contribute to open source DevOps projects
  ✅ Get certified: AWS DevOps Professional, CKA, or Terraform Associate
```

### The Tech Stack of a Senior DevOps Engineer

```
Version Control:    Git, GitHub/GitLab
CI/CD:              Jenkins, GitHub Actions, ArgoCD
Containers:         Docker, Docker Compose
Orchestration:      Kubernetes (EKS/GKE/AKS)
Infrastructure:     Terraform, Pulumi
Configuration:      Ansible, Chef
Cloud:              AWS / Azure / GCP
Monitoring:         Prometheus, Grafana, ELK Stack
Security:           Vault, SAST/DAST tools
Scripting:          Bash, Python, Groovy
```

### Salary Trajectory (India — Approximate)

| Level | Experience | Skills | Salary Range (LPA) |
|---|---|---|---|
| Junior DevOps | 0–2 years | Jenkins, Docker, Linux, Git | ₹4–8 LPA |
| Mid DevOps | 2–4 years | + Kubernetes, Terraform, Cloud | ₹8–18 LPA |
| Senior DevOps | 4–7 years | + Architecture, SRE, Security | ₹18–35 LPA |
| Lead/Principal | 7+ years | Team leadership, company-wide DevOps | ₹35–60+ LPA |
| DevOps Consultant | Any | Freelancing, specialized expertise | ₹50–100+ LPA |

### Portfolio Projects to Build

1. **MERN Stack CI/CD Pipeline** — Full pipeline from commit to production (covered in this guide)
2. **Microservices Pipeline** — 3+ services, each with independent pipelines
3. **Infrastructure Automation** — Terraform + Jenkins provisions a full cloud environment
4. **Security-First Pipeline** — OWASP scan, container scanning, compliance gates
5. **Multi-Cloud Deployment** — Deploy to AWS and Azure from same Jenkins pipeline

---

## Quick Reference Summary

```
INSTALL JENKINS:
  Ubuntu: sudo apt install jenkins (after adding repo)
  Docker: docker run -p 8080:8080 jenkins/jenkins:lts

BASIC PIPELINE:
  pipeline { agent any; stages { stage('X') { steps { sh '...' } } } }

CREDENTIALS:
  withCredentials([usernamePassword(credentialsId:'id', usernameVariable:'U', passwordVariable:'P')]) { sh '...' }

DOCKER BUILD+PUSH:
  sh "docker build -t image:${BUILD_NUMBER} ."
  sh "docker push image:${BUILD_NUMBER}"

PARALLEL STAGES:
  stage('Parallel') { parallel { stage('A') { ... } stage('B') { ... } } }

SSH DEPLOY:
  sh "ssh -i $KEY $USER@server 'docker pull image && docker restart app'"

WEBHOOK:
  GitHub → Settings → Webhooks → http://JENKINS:8080/github-webhook/

IMPORTANT PLUGINS:
  Git, GitHub Integration, Docker Pipeline, Kubernetes, Role Strategy,
  Pipeline, Blue Ocean, Slack Notification, Email Extension, NodeJS
```

---

*Guide Version: 1.0 | Jenkins LTS 2.x | Tested with Ubuntu 22.04 + Docker 24 + Kubernetes 1.28 | Last Updated: 2025*

---

> **You're now Jenkins-ready.**
> The next step is hands-on practice. Clone a repo, write a Jenkinsfile, break it, debug it, fix it. That's how senior DevOps engineers are made — not by reading, but by doing.
