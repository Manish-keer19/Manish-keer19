# Jenkins Masterclass: The CI/CD Powerhouse (Zero to Expert) 🤵

## Table of Contents
1.  [Introduction & Architecture](#1-introduction--architecture)
2.  [Installation (The Modern Docker Way)](#2-installation-the-modern-docker-way)
3.  [The Pipeline (Jenkinsfile)](#3-the-pipeline-jenkinsfile)
4.  [Advanced Pipeline Features](#4-advanced-pipeline-features)
5.  [Managing Credentials & Secrets](#5-managing-credentials--secrets)
6.  [Industry Standards (Docker Agents & Shared Libs)](#6-industry-standards-docker-agents--shared-libs)
7.  [Best Practices Checklist](#7-best-practices-checklist)

---

## 1. Introduction & Architecture

### What is Jenkins?
Jenkins is the grandfather of CI/CD. It is an open-source automation server.
*   **CI (Continuous Integration):** Developers push code -> Jenkins detects it -> Runs Tests -> Builds Artifacts.
*   **CD (Continuous Deployment):** Artifacts ready -> Jenkins deploys to Staging/Production.

### Jenkins vs GitHub Actions
*   **GitHub Actions:** Cloud-native, zero setup, easy YAML. Best for most modern web apps.
*   **Jenkins:** Self-hosted, infinite customizability (Groovy scripts), Enterprise RBAC. Best for complex, legacy, or highly secure on-premise environments (Banks, Gov).

### Architecture
1.  **Controller (Master):** The Brain. Stores configs, handles user UI, schedules jobs. **DO NOT RUN BUILDS HERE.**
2.  **Agents (Nodes):** The Muscle. Linux/Windows servers that actually run `npm install` or `docker build`.
3.  **Executors:** The slots available on an agent to run jobs.

---

## 2. Installation (The Modern Docker Way)

Don't install Jenkins directly on your OS. Run it in Docker.

```bash
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins \
  jenkins/jenkins:lts-jdk17
```
*   Visit `http://localhost:8080`.
*   Unlock using the password found in `docker logs jenkins`.
*   Install "Suggested Plugins".

---

## 3. The Pipeline (Jenkinsfile)

The **Declarative Pipeline** is the industry standard. It creates a `Jenkinsfile` in your Git repo.

```groovy
pipeline {
    agent any # Run on any available agent

    environment {
        NODE_ENV = 'production'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'npm test'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
            junit 'test-results/**/*.xml'
        }
        failure {
            echo 'Build Failed! Sending Slack notification...'
        }
        success {
            echo 'Great success!'
        }
    }
}
```

---

## 4. Advanced Pipeline Features

### Parameters
Make your build interactive.
```groovy
parameters {
    string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: 'Target Environment')
    booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip Unit Tests?')
}
...
stage('Deploy') {
    steps {
        echo "Deploying to ${params.DEPLOY_ENV}"
    }
}
```

### Parallel Execution
Speed up slow tests by running them at the same time.
```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') { steps { sh 'npm run test:unit' } }
        stage('E2E Tests') { steps { sh 'npm run test:e2e' } }
        stage('Lint') { steps { sh 'npm run lint' } }
    }
}
```

---

## 5. Managing Credentials & Secrets

**NEVER** check passwords into Git.
1.  Go to **Manage Jenkins** > **Credentials**.
2.  Add a "Secret Text" (e.g., ID: `AWS_SECRET_KEY`) or "Username with Password" (e.g., ID: `DOCKER_HUB_CREDS`).

**Usage in Pipeline:**
```groovy
stage('Login to Docker Hub') {
    environment {
        # Securely injects into env vars 'DOCKER_USER' and 'DOCKER_PASS'
        DOCKER_CREDS = credentials('DOCKER_HUB_CREDS') 
    }
    steps {
        sh 'echo $DOCKER_CREDS_PASSWORD | docker login -u $DOCKER_CREDS_USR --password-stdin'
    }
}
```

---

## 6. Industry Standards (Docker Agents & Shared Libs) 🚀

### A. Docker Agents (The "Clean Room" Approach)
Don't install Node 14, Node 16, Python 3, and Java 8 all on the same agent machine. It's a mess. 
Instead, run the build step **inside a Docker container**.

```groovy
pipeline {
    agent {
        docker { 
            image 'node:18-alpine' 
            args '-v /root/.npm:/root/.npm' # Cache npm
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'node -v' # Output: v18.x (Inside container)
                sh 'npm install'
            }
        }
    }
}
```

### B. Shared Libraries
Big companies have 500 repos. They don't write the same Jenkinsfile logic 500 times.
They create a **Shared Library** repo with reusable functions.

**In Shared Lib Repo (`vars/deployToS3.groovy`):**
```groovy
def call(String bucketName) {
    sh "aws s3 sync ./dist s3://${bucketName}"
}
```

**In Application Jenkinsfile:**
```groovy
@Library('my-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                deployToS3('my-production-bucket') # Single line!
            }
        }
    }
}
```

---

## 7. Best Practices Checklist
1.  **Pipeline as Code:** No GUI configurations. Everything in `Jenkinsfile`.
2.  **Zero Executors on Master:** Set "# of executors" on the built-in node to 0. Master is for orchestration, not heavy lifting.
3.  **Ephemeral Agents:** Use Docker or Cloud Agents (AWS EC2 Fleet) that spin up on demand and die after the build.
4.  **Triggers:** Don't poll SCM every minute (`H/5 * * * *`). Use **Webhooks** (GitHub calls Jenkins when code is pushed).
