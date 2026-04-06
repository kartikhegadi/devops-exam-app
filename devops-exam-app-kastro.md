# 🚀 DevOps Exam App — Setup Guide by Kastro

> A complete step-by-step guide to setting up the DevOps Exam App with Jenkins CI/CD pipeline, Docker, SonarQube, Trivy, and Docker Scout.

---

## 📋 Table of Contents

1. [Launch Ubuntu VM](#1-launch-ubuntu-vm)
2. [Connect to VM](#2-connect-to-vm)
3. [Install Jenkins](#3-install-jenkins)
4. [Install Docker](#4-install-docker)
5. [Install Trivy](#5-install-trivy)
6. [Install Docker Scout](#6-install-docker-scout)
7. [Install SonarQube](#7-install-sonarqube)
8. [Jenkins Plugin Installation](#8-jenkins-plugin-installation)
9. [Tools Configuration in Jenkins](#9-tools-configuration-in-jenkins)
10. [Docker Hub Credentials Configuration](#10-docker-hub-credentials-configuration)
11. [SonarQube System Configuration in Jenkins](#11-sonarqube-system-configuration-in-jenkins)
12. [MySQL Database Access](#12-mysql-database-access)
13. [Jenkins Pipeline Script](#13-jenkins-pipeline-script)

---

## 1. Launch Ubuntu VM

**Specs:** Ubuntu 24.04 | `t2.large` | 30 GB Storage

### 🔓 Open the Following Ports

| Type              | Protocol | Port Range |
|-------------------|----------|------------|
| All TCP           | TCP      | 0 – 65535  |
| All ICMP - IPv4   | ICMP     | All        |
| SSH               | TCP      | 22         |
| Custom TCP        | TCP      | 3000       |
| Custom TCP        | TCP      | 8081       |
| Custom TCP        | TCP      | 8080       |
| HTTPS             | TCP      | 443        |
| Custom TCP        | TCP      | 6443       |
| HTTP              | TCP      | 80         |
| SonarQube         | TCP      | 9000       |

---

## 2. Connect to VM

```bash
sudo su
sudo apt update
```

---

## 3. Install Jenkins

Create and run the Jenkins installation script:

```bash
vi jenkins.sh
```

Paste the following content:

```bash
#!/bin/bash

# Install OpenJDK 17 JRE Headless
sudo apt install openjdk-17-jre-headless -y

# Download Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to package manager sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package manager repositories
sudo apt-get update

# Install Jenkins
sudo apt-get install jenkins -y
```

Save and run:

```bash
# Save the file
# Press: Esc → :wq → Enter

sudo chmod +x jenkins.sh
./jenkins.sh
jenkins --version
```

> ✅ Then set up the **Jenkins Dashboard** via your browser at `http://<VM-IP>:8080`

---

## 4. Install Docker

Create and run the Docker installation script:

```bash
vi docker.sh
```

Paste the following content:

```bash
#!/bin/bash

# Update package manager repositories
sudo apt-get update

# Install necessary dependencies
sudo apt-get install -y ca-certificates curl

# Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to Apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package manager repositories
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Save and run:

```bash
# Press: Esc → :wq → Enter

sudo chmod +x docker.sh
./docker.sh
docker --version
```

### 🧪 Test Docker

```bash
docker pull hello-world
```

> ⚠️ If you get a permissions error, run:
> ```bash
> sudo chmod 666 /var/run/docker.sock
> docker pull hello-world
> ```

### ⚙️ Post-Installation Configuration

```bash
# Add Jenkins user to docker group
sudo usermod -aG docker jenkins

# Restart Jenkins to apply changes
sudo systemctl restart jenkins

# Verify Jenkins can access Docker
sudo -u jenkins docker ps

# Update & Install Python pip
sudo apt-get update
sudo apt-get install -y python3-venv python3-pip

# Install docker-compose plugin
sudo apt-get update
sudo apt-get install -y docker-compose-plugin
```

---

## 5. Install Trivy

Create and run the Trivy installation script:

```bash
vi trivy.sh
```

Paste the following content:

```bash
#!/bin/bash
sudo apt-get install wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

Save and run:

```bash
# Press: Esc → :wq → Enter

sudo chmod +x trivy.sh
./trivy.sh
trivy --version
```

---

## 6. Install Docker Scout

> 🔐 Make sure you are **logged into DockerHub** in the browser first.

```bash
# Login to DockerHub from terminal
docker login -u <DockerHubUserName>
# Enter your DockerHub password when prompted

# Install Docker Scout
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin

# Grant required permissions
sudo chmod 777 /var/run/docker.sock
```

---

## 7. Install SonarQube

Run SonarQube as a Docker container:

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
docker images
docker ps
```

> 🌐 Access SonarQube at `http://<VM-IP>:9000`
>
> **Default credentials:**
> - Username: `admin`
> - Password: `admin`
>
> ⚠️ You will be prompted to set a new password on first login.

---

## 8. Jenkins Plugin Installation

Go to **Manage Jenkins → Plugins → Available Plugins** and install the following:

| Plugin |
|--------|
| SonarQube Scanner |
| Docker |
| Docker Commons |
| Docker Pipeline |
| Docker API |
| docker-build-step |
| Pipeline Stage View |
| Email Extension Template |
| Kubernetes |
| Kubernetes CLI |
| Kubernetes Client API |
| Kubernetes Credentials |
| Kubernetes Credentials Provider |
| Config File Provider |
| Prometheus Metrics |

> ⚠️ The **Prometheus Metrics** plugin may require a Jenkins restart. Check the box **"Restart Jenkins when installation is complete"** when prompted.

---

## 9. Tools Configuration in Jenkins

**Path:** `Manage Jenkins → System Configuration → Tools`

### SonarQube Scanner
- Click **Add SonarQube Scanner**
- Name: `sonar-scanner`
- ✅ Check **Install automatically**
- Version: `7.1.0.4889` (latest)

### Docker
- Click **Add Docker**
- Name: `docker`
- ✅ Check **Install automatically**
- Click **Add Installer** → Select **Download from docker.com**
- Version: `latest`

Click **Apply → Save**

---

## 10. Docker Hub Credentials Configuration

**Path:** `Manage Jenkins → Security → Credentials → Global → Add Credentials`

| Field       | Value                        |
|-------------|------------------------------|
| Kind        | Username with Password       |
| Scope       | Global                       |
| Username    | `<Your DockerHub Username>`  |
| Password    | `<Your DockerHub Password>`  |
| ID          | `docker`                     |
| Description | `docker`                     |

Click **Create**.

---

## 11. SonarQube System Configuration in Jenkins

### Step 1 — Generate SonarQube Token

In the **SonarQube console:**

`Administration → Security → Users → Tokens (click 3-dash icon)`

- Name: `token`
- Expires in: `90 days`
- Click **Generate** → Copy the token

### Step 2 — Add Token to Jenkins Credentials

**Path:** `Manage Jenkins → Security → Credentials → Global → Add Credentials`

| Field       | Value                        |
|-------------|------------------------------|
| Kind        | Secret text                  |
| Scope       | Global                       |
| Secret      | `<Paste SonarQube Token>`    |
| ID          | `sonar-token`                |
| Description | `sonar-token`                |

### Step 3 — Create SonarQube Webhook

In the **SonarQube console:**

`Administration → Configuration → Webhooks → Create`

| Field | Value                                        |
|-------|----------------------------------------------|
| Name  | `jenkins`                                    |
| URL   | `http://<Jenkins-IP>:8080/sonarqube-webhook/` |

### Step 4 — Configure SonarQube in Jenkins System Settings

**Path:** `Manage Jenkins → System Configuration → System → SonarQube Servers → Add SonarQube`

| Field                       | Value                              |
|-----------------------------|------------------------------------|
| Name                        | `sonar`                            |
| Server URL                  | `http://<VM-IP>:9000`              |
| Server Authentication Token | Select `sonar-token` from dropdown |

Click **Apply → Save**

### Step 5 — Docker Registry Pipeline Syntax

In your Jenkins job:

`Pipeline Syntax → Sample step: withDockerRegistry`

- Docker registry URL: *(leave empty for Docker Hub)*
- Registry credentials: `docker`
- Docker installation: `docker`

Click **Generate Pipeline Syntax** → Copy and paste in your pipeline script.

---

## 12. MySQL Database Access

### Connect to the MySQL Container

```bash
docker exec -it mysql_db mysql -u root -p
# Password: rootpass
```

### Common SQL Operations

```sql
-- Show all databases
SHOW DATABASES;

-- Use the application database
USE devops_exam;

-- Show all tables
SHOW TABLES;

-- View structure of the results table
DESCRIBE results;

-- View all exam results
SELECT * FROM results;

-- View results sorted by highest score
SELECT * FROM results ORDER BY score DESC;

-- View average score
SELECT AVG(score) FROM results;

-- View number of submissions
SELECT COUNT(*) FROM results;
```

---

## 13. Jenkins Pipeline Script

> Complete pipeline with **Trivy FS Scan**, **SonarQube Analysis**, **Docker Build & Push**, **Docker Scout Analysis**, and **Docker Compose Deployment**.

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kastrov/devopsexamapp:latest"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/KastroVKiran/devops-exam-app.git',
                    branch: 'master'
            }
        }

        stage('File System Scan') {
            steps {
                sh "trivy fs --security-checks vuln,config --format table -o trivy-fs-report.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=devops-exam-app \
                    -Dsonar.projectKey=devops-exam-app \
                    -Dsonar.sources=. \
                    -Dsonar.language=py \
                    -Dsonar.python.version=3 \
                    -Dsonar.host.url=http://localhost:9000
                    """
                }
            }
        }

        stage('Verify Docker Compose') {
            steps {
                sh '''
                docker compose version || { echo "Docker Compose not available"; exit 1; }
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('backend') {
                    script {
                        withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                            sh "docker build -t ${DOCKER_IMAGE} ."
                            sh "docker push ${DOCKER_IMAGE}"
                        }
                    }
                }
            }
        }

        stage('Docker Scout Image Analysis') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker-scout quickview ${DOCKER_IMAGE}"
                        sh "docker-scout cves ${DOCKER_IMAGE}"
                        sh "docker-scout recommendations ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                # Clean up any existing containers
                docker compose down --remove-orphans || true

                # Start services with build
                docker compose up -d --build

                # Wait for MySQL to be ready
                echo "Waiting for MySQL to be ready..."
                timeout 120s bash -c '
                while ! docker compose exec -T mysql mysqladmin ping -uroot -prootpass --silent;
                do
                    sleep 5;
                    docker compose logs mysql --tail=5 || true;
                done'

                # Additional wait for full initialization
                sleep 10
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "=== Container Status ==="
                docker compose ps -a
                echo "=== Testing Flask Endpoint ==="
                curl -I http://localhost:5000 || true
                '''
            }
        }
    }

    post {
        success {
            echo '🚀 Deployment successful!'
            sh 'docker compose ps'
            archiveArtifacts artifacts: 'trivy-fs-report.html', allowEmptyArchive: true
        }
        failure {
            echo '❗ Pipeline failed. Check logs above.'
            sh '''
            echo "=== Error Investigation ==="
            docker compose logs --tail=50 || true
            '''
        }
        always {
            sh '''
            echo "=== Final Logs ==="
            docker compose logs --tail=20 || true
            '''
            archiveArtifacts artifacts: 'trivy-fs-report.html', allowEmptyArchive: true
        }
    }
}
```

---

## 🗺️ Pipeline Overview

```
Git Checkout
    ↓
File System Scan (Trivy)
    ↓
SonarQube Analysis
    ↓
Verify Docker Compose
    ↓
Build & Push Docker Image
    ↓
Docker Scout Image Analysis
    ↓
Deploy with Docker Compose
    ↓
Verify Deployment
```

---

> 📌 **Tip:** Always ensure ports are open in your cloud security group before accessing any service.
> Made with ❤️ by **Kastro**
