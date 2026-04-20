<div align="center">

# 🏦 Banking CI/CD Pipeline — Full DevOps Automation

### Java Spring Boot · Jenkins · Docker · Terraform · Ansible · Prometheus · Grafana · AWS EC2

[![Java](https://img.shields.io/badge/Java-OpenJDK_17_Corretto-ED8B00?style=flat-square&logo=openjdk&logoColor=white)](https://aws.amazon.com/corretto/)
[![Jenkins](https://img.shields.io/badge/Jenkins-2.462.3-D24939?style=flat-square&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-25.0.5-2496ED?style=flat-square&logo=docker&logoColor=white)](https://hub.docker.com/r/jhansi977/banking-project-demo)
[![Terraform](https://img.shields.io/badge/Terraform-1.9.8-7B42BC?style=flat-square&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Ansible](https://img.shields.io/badge/Ansible-2.15.3-EE0000?style=flat-square&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-2.53.2-E6522C?style=flat-square&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-v11.3.0-F46800?style=flat-square&logo=grafana&logoColor=white)](https://grafana.com/)
[![AWS](https://img.shields.io/badge/AWS_EC2-us--east--1-FF9900?style=flat-square&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![Status](https://img.shields.io/badge/Build_4-✅_Passing-22c55e?style=flat-square)]()

---

> **A single `git push` triggers a 7-stage Jenkins pipeline that builds, containerizes, provisions a fresh EC2, deploys the banking app, and monitors it — all in 3 minutes 23 seconds. Fully automated. Zero manual steps.**

---

| 🔧 Jenkins 2.462.3 | 🐳 Docker 25.0.5 | 🏗️ Terraform 1.9.8 | 📜 Ansible 2.15.3 | 🔥 Prometheus 2.53.2 | 📊 Grafana v11.3.0 |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **☕ Java 17 Corretto** | **📦 Maven 3.9.9** | **☁️ AWS t2.micro** | **ok=5 · failed=0** | **2/2 targets UP** | **Live Dashboard** |

</div>

---

## 📖 Table of Contents

- [What is this project?](#-what-is-this-project)
- [Architecture Overview](#-architecture-overview)
- [Infrastructure — EC2 Instances](#-infrastructure--ec2-instances)
- [Pipeline — 7 Stages](#-pipeline--7-stages)
- [Key Configuration Files](#-key-configuration-files)
- [Monitoring Stack](#-monitoring-stack)
- [How to Reproduce This Project](#-how-to-reproduce-this-project)
- [Screenshots](#-screenshots)
- [Real Issues I Solved](#-real-issues-i-solved)
- [Tech Stack](#-tech-stack)

---

## 💡 What is this project?

This is a **real-world end-to-end DevOps project** built completely from scratch during Star Agile DevOps Training. It deploys a Java Spring Boot banking application using a fully automated CI/CD pipeline.

**What makes it complete:**
- Application code is a Spring Boot "Customer Banking Services" web app
- Every push to GitHub automatically triggers the full pipeline
- Infrastructure is provisioned by **Terraform** (no pre-existing EC2)
- Application is deployed by **Ansible** (no manual SSH)
- Monitoring is set up with **Prometheus + Grafana** (live dashboards)

**DockerHub Image:** `jhansi977/banking-project-demo:1.0` (696MB)

---

## 🏗️ Architecture Overview

```
Developer
   │
   │ git push
   ▼
GitHub (Jhansi-112/Banking-CI-CD-Pipeline)
   │
   │ Webhook trigger
   ▼
Jenkins Server (50.17.98.208:8080)  ←── i-0fbe600dbd28f8da3 · t2.medium
   │
   ├─ Stage 1: Checkout SCM (228ms)
   ├─ Stage 2: Tool Install — Maven M2_HOME (96ms)
   ├─ Stage 3: Build — mvn clean package (12s)
   ├─ Stage 4: Test Reports — publishHTML (184ms)
   ├─ Stage 5: Docker Image — docker build :1.0 (1s)
   ├─ Stage 6: Docker Login + Push — DockerHub (4s + 4.8s)
   └─ Stage 7: Config & Deployment (3m 3s)
          │
          ├─ terraform init/validate/apply --auto-approve
          │        └─ Creates EC2: i-0a71acc8b0b2edcb5 (52.55.21.187) in 1m 26s
          │
          └─ ansible-playbook ansibleplaybook.yml
                   └─ SSH into 52.55.21.187
                   ├─ yum update
                   ├─ yum install docker
                   ├─ systemctl start docker
                   └─ docker run -itd -p 8084:8081 jhansi977/banking-project-demo:1.0
                            └─ Container: 0c8064116d89 (exciting_morse) ✅ LIVE

🌐 Application: http://52.55.21.187:8084
   "CUSTOMER BANKING SERVICES"

📊 Monitoring EC2 (54.91.220.8) ←── i-0cfc4717d4c4bfdc5
   ├─ Prometheus 2.53.2 (port 9090) — scraping node_exporter
   ├─ Node Exporter 1.8.2 (52.55.21.187:9100) — system metrics
   └─ Grafana v11.3.0 (port 3000) — Server Metrics Dashboard
```

---

## 🖥️ Infrastructure — EC2 Instances

| Instance | ID | Public IP | Private IP | Type | Purpose |
|---|---|---|---|---|---|
| jenkins | i-0fbe600dbd28f8da3 | 50.17.98.208 | 172.31.21.48 | t2.medium | Jenkins + All DevOps tools |
| test-server *(Terraform)* | i-0a71acc8b0b2edcb5 | 52.55.21.187 | 172.31.44.226 | t2.micro | Banking app container |
| Monitoring-prometheus | i-0cfc4717d4c4bfdc5 | 54.91.220.8 | 172.31.32.120 | — | Prometheus + Grafana |

> **Note:** The `test-server` EC2 is **created automatically by Terraform** on every pipeline run — it is not pre-existing.

---

## 🔄 Pipeline — 7 Stages

### Jenkins Server Setup (one-time)

```bash
# Install Java 17
sudo yum install java-17-amazon-corretto-devel -y
yum install jenkins -y

# Set environment variables in .bashrc
JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
M2_HOME=/opt/maven
M2=$M2_HOME/bin
PATH=$PATH:$JAVA_HOME/bin:$M2

# Install Maven 3.9.9
cd /opt
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
tar -xvzf apache-maven-3.9.9-bin.tar.gz
mv apache-maven-3.9.9 maven

# Install other tools
yum install git docker ansible -y
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform

# Docker permissions
usermod -aG docker ec2-user
usermod -aG docker jenkins
systemctl enable docker && systemctl restart docker

# Jenkins sudo permissions
vi /etc/sudoers
# Add: jenkins ALL=NOPASSWD: ALL
```

### Jenkinsfile (all 7 stages)

```groovy
pipeline {
    agent any
    tools {
        maven "M2_HOME"
    }
    stages {
        stage('Build') {
            steps {
                git 'https://github.com/Jhansi-112/Banking-CI-CD-Pipeline.git'
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage('Generate Test Reports') {
            steps {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false,
                  keepAll: false,
                  reportDir: '/var/lib/jenkins/workspace/BankingProject/target/surefire-reports',
                  reportFiles: 'index.html', reportName: 'HTML Report'])
            }
        }
        stage('Create Docker Image') {
            steps {
                sh 'docker build -t jhansi977/banking-project-demo:1.0 .'
            }
        }
        stage('Docker-Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-login',
                  passwordVariable: 'dockerpassword', usernameVariable: 'dockerlogin')]) {
                    sh 'docker login -u ${dockerlogin} -p ${dockerpassword}'
                }
            }
        }
        stage('Push-Image') {
            steps {
                sh 'docker push jhansi977/banking-project-demo:1.0'
            }
        }
        stage('Config & Deployment') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                  credentialsId: 'AWS-ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    dir('terraform-files') {
                        sh 'sudo chmod 600 mykey-project.pem'
                        sh 'terraform init'
                        sh 'terraform validate'
                        sh 'terraform apply --auto-approve'
                    }
                }
            }
        }
    }
}
```

---

## 📁 Key Configuration Files

### `terraform-files/main.tf`
```hcl
resource "aws_instance" "test-server" {
  ami                    = "ami-06b21ccaeff8cd686"
  instance_type          = "t2.micro"
  key_name               = "mykey-project"
  vpc_security_group_ids = ["sg-060316f6d3f0dec32"]

  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("./mykey-project.pem")
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = ["echo 'wait to start the instance' "]
  }

  tags = { Name = "test-server" }

  provisioner "local-exec" {
    command = "echo ${aws_instance.test-server.public_ip} > inventory"
  }

  provisioner "local-exec" {
    command = "ansible-playbook /var/lib/jenkins/workspace/BankingProject/terraform-files/ansibleplaybook.yml"
  }
}
```

### `terraform-files/provider.tf`
```hcl
provider "aws" {
  region = "us-east-1"
}
```

### `terraform-files/ansible.cfg`
```ini
[defaults]
inventory          = ./inventory
deprecation_warning = False
remote_user        = ec2-user
host_key_checking  = False
private_key_file   = ./mykey-project.pem

[privilage_esclation]
become          = true
become_method   = sudo
become_user     = root
become_ask_pass = False
```

### `terraform-files/ansibleplaybook.yml`
```yaml
- name: Configure Docker on EC2 Instances
  hosts: all
  become: true
  connection: ssh
  tasks:
    - name: updating apt
      command: sudo yum update

    - name: Install Docker
      command: sudo yum install docker -y

    - name: Start Docker Service
      command: sudo systemctl start docker

    - name: Deploy Docker Container
      command: docker run -itd -p 8084:8081 jhansi977/banking-project-demo:1.0
```

---

## 📊 Monitoring Stack

### Prometheus Setup (Monitoring EC2: 54.91.220.8)

```bash
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.53.2/prometheus-2.53.2.linux-amd64.tar.gz
tar -xvzf prometheus-2.53.2.linux-amd64.tar.gz
mv prometheus-2.53.2.linux-amd64 prometheus
cd prometheus
vi prometheus.yml   # Configure scrape targets (see below)
./prometheus
```

**prometheus.yml scrape config:**
```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["54.91.220.8:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["52.55.21.187:9100"]
```

### Node Exporter Setup (test-server: 52.55.21.187)

```bash
cd /opt
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xvzf node_exporter-1.8.2.linux-amd64.tar.gz
mv node_exporter-1.8.2.linux-amd64 node_exporter
cd node_exporter
./node_exporter
# Running at 52.55.21.187:9100
```

### Grafana Setup (same Monitoring EC2: 54.91.220.8)

```bash
cd /opt
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-11.3.0.linux-amd64.tar.gz
tar -xvzf grafana-enterprise-11.3.0.linux-amd64.tar.gz
mv grafana-v11.3.0 grafana
cd grafana/bin
./grafana-server
# Running at 54.91.220.8:3000
```

**Prometheus Targets — Both UP ✅**

| Target | Endpoint | State | Last Scrape | Duration |
|---|---|---|---|---|
| node_exporter | 52.55.21.187:9100 | ✅ UP | 13.8s ago | 16ms |
| prometheus | 54.91.220.8:9090 | ✅ UP | 8.4s ago | 5.78ms |

**Grafana Dashboard:** "Server Metrics — CPU/Memory/Disk/Network"
- Host: `52.55.21.187:9100` · Interval: 1m · Last 15 minutes
- Panels: CPU%, CPU Cores (1), Memory (0–1.13 GiB), Load, Disk (~20%), Network

---

## 🚀 How to Reproduce This Project

### Step 1 — Launch Jenkins EC2 (t2.medium, Amazon Linux, us-east-1)
```bash
# Install all tools using the commands in "Jenkins Server Setup" section above
```

### Step 2 — Fork this repo and configure GitHub Webhook
```
Webhook URL: http://<jenkins-public-ip>:8080/github-webhook/
Content type: application/json
Events: push
```

### Step 3 — Add Jenkins Credentials
```
Manage Jenkins → Credentials → Global:
1. ID: Docker-login    → DockerHub username + password
2. ID: terraform-ansible → SSH private key (PEM file content)
3. ID: AWS-ID          → AWS Access Key ID + Secret Access Key
```

### Step 4 — Configure Jenkins Pipeline Job
```
New Item → Pipeline → Named: BankingProject
Pipeline Definition → Pipeline script from SCM
SCM: Git → Repository URL: https://github.com/Jhansi-112/Banking-CI-CD-Pipeline.git
Script Path: jenkinsfile
```

### Step 5 — Add terraform-files/ to your repo
```
Add: main.tf, provider.tf, ansible.cfg, ansibleplaybook.yml, inventory (empty)
Add your .pem key as mykey-project.pem (referenced in main.tf)
```

### Step 6 — Click Build Now or push to GitHub
```
Watch Jenkins pipeline run all 7 stages automatically!
App will be live at: http://<test-server-public-ip>:8084
```

### Step 7 — Setup Monitoring (separate EC2)
```bash
# Follow "Monitoring Stack" section commands above
# Update prometheus.yml with the actual IP of test-server after Terraform creates it
```

---

## 📸 Screenshots

> Screenshots are in the [`/screenshots`](./screenshots/) folder.

| Screenshot | Description |
|---|---|
| `jenkins-tools-version.png` | jenkins 2.462.3 · java 17 · maven 3.9.9 verified |
| `all-tools-version.png` | git 2.40.1 · docker 25.0.5 · ansible 2.15.3 · terraform 1.9.8 |
| `jenkins-credentials.png` | Docker-login · terraform-ansible · AWS-ID configured |
| `dockerhub-image-pushed.png` | jhansi977/banking-project-demo · tags 1.0 + 3.0 |
| `jenkins-pipeline-success.png` | Build #4 — all 7 stages green · avg 3min 23s |
| `jenkins-pipeline-steps.png` | Full pipeline steps with timings |
| `terraform-ec2-created.png` | test-server created · IP 52.55.21.187 · 1m 26s |
| `aws-ec2-running.png` | i-0a71acc8b0b2edcb5 · Running · t2.micro |
| `banking-app-live.png` | Customer Banking Services live at 52.55.21.187:8084 |
| `docker-ps-running.png` | Container 0c8064116d89 · Up · 0.0.0.0:8084→8081 |
| `prometheus-targets-up.png` | node_exporter + prometheus both UP |
| `node-exporter-metrics.png` | Live metrics at 52.55.21.187:9100/metrics |
| `grafana-cpu-dashboard.png` | CPU panel · ~100% idle · live last 15min |
| `grafana-memory-dashboard.png` | Memory 0–1.13 GiB · full breakdown |
| `grafana-load-disk.png` | Load 0–0.06 · Disk 20% used |
| `grafana-datasource-success.png` | "Successfully queried the Prometheus API." |

---

## 🧠 Real Issues I Solved

These are **actual problems** I debugged during this project:

| Issue | Root Cause | Fix |
|---|---|---|
| Jenkins couldn't run Docker/Terraform | Jenkins user had no sudo permissions | Added `jenkins ALL=NOPASSWD: ALL` in sudoers |
| Ansible didn't run after Terraform | Inventory not updated with new EC2 IP | Terraform `local-exec`: `echo public_ip > inventory` |
| Terraform SSH connection failed | PEM file permissions too open (644) | `sudo chmod 600 mykey-project.pem` |
| Docker push: authentication required | Credentials not injected in pipeline | Used `withCredentials([usernamePassword(...)])` |
| Grafana showed No Data | Prometheus datasource URL wrong | Set URL to `http://54.91.220.8:9090` |
| Node Exporter not scraped | Wrong IP in prometheus.yml | Updated target to `52.55.21.187:9100` |
| App not responding on port | Container maps 8084→8081 not 8080 | Ansible: `docker run -itd -p 8084:8081 ...` |

---

## 🛠️ Tech Stack

| Layer | Tool | Version |
|---|---|---|
| Application | Java Spring Boot | OpenJDK 17 Corretto-17.0.12.7.1 |
| Build | Maven | 3.9.9 |
| CI/CD | Jenkins | 2.462.3 |
| Source Control | GitHub + Webhooks | — |
| Containerization | Docker | 25.0.5 |
| Registry | DockerHub | jhansi977/banking-project-demo |
| Infrastructure as Code | Terraform | 1.9.8 (hashicorp/aws v5.73.0) |
| Config Management | Ansible | 2.15.3 (core) + Python 3.9.16 |
| Cloud | AWS EC2 | us-east-1 (N. Virginia) |
| Monitoring | Prometheus | 2.53.2 |
| Metrics Agent | Node Exporter | 1.8.2 |
| Dashboards | Grafana | v11.3.0 (Enterprise) |

---

## 🔗 Links

- 📄 **[Interactive Portfolio Page](./portfolio.html)**
- 🏗️ **[Architecture Diagram](./architecture.html)**
- 🐳 **[DockerHub Image](https://hub.docker.com/r/jhansi977/banking-project-demo)**

---

## 👩‍💻 About

Built by **Jhansi** — DevOps engineer from Hyderabad 🇮🇳

> *"This project taught me that real DevOps isn't just running commands — it's debugging Terraform provisioner timing, fixing Ansible SSH key permissions, troubleshooting Docker port mappings, and getting Prometheus to scrape a freshly deployed EC2. Every issue in this README is a real lesson."*

---

<div align="center">

**⭐ Star this repo if it helped you learn DevOps!**

[![GitHub stars](https://img.shields.io/github/stars/Jhansi-112/Banking-CI-CD-Pipeline?style=social)](https://github.com/Jhansi-112/Banking-CI-CD-Pipeline)

</div>
