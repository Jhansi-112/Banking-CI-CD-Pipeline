# Banking CI/CD Pipeline 🏦

> Automated infrastructure provisioning and application deployment using Jenkins, Docker, Terraform, Ansible — with real-time monitoring via Prometheus and Grafana on AWS EC2.

---

## 🔍 Project Summary

| Item | Detail |
|------|--------|
| **Application** | Java Spring Boot — Customer Banking Services |
| **Docker Image** | `jhansi977/banking-project-demo:1.0` — 696MB |
| **Live URL** | `http://52.55.21.187:8084` |
| **Pipeline** | 7 stages · Build #4 · Total time: **3 min 23 sec** |
| **Ansible Result** | `ok=5` `changed=4` `failed=0` `unreachable=0` |
| **Prometheus** | 2 targets — both `UP` |
| **Region** | AWS us-east-1 (N. Virginia) · Oct 28 2024 |

---

## 🖥️ EC2 Instances Used

```
┌─────────────────────┬──────────────────────┬───────────────┬──────────┐
│ Name                │ Instance ID          │ Public IP     │ Type     │
├─────────────────────┼──────────────────────┼───────────────┼──────────┤
│ Jenkins (DevOps)    │ i-0fbe600dbd28f8da3  │ 50.17.98.208  │ t2.medium│
│ test-server (App)   │ i-0a71acc8b0b2edcb5  │ 52.55.21.187  │ t2.micro │
│ Monitoring          │ i-0cfc4717d4c4bfdc5  │ 54.91.220.8   │  —       │
└─────────────────────┴──────────────────────┴───────────────┴──────────┘
```

> ⚡ The **test-server** EC2 is not pre-created — it is **provisioned automatically by Terraform** every time the pipeline runs.

---

## ⚙️ Tools & Versions

```
Jenkins        2.462.3          CI/CD automation server
Java           OpenJDK 17       Amazon Corretto-17.0.12.7.1
Maven          3.9.9            Spring Boot build tool
Git            2.40.1           Source control
Docker         25.0.5           Containerization
Terraform      1.9.8            Infrastructure as Code (hashicorp/aws v5.73.0)
Ansible        2.15.3 (core)    Configuration management
Python         3.9.16           Ansible runtime dependency
Prometheus     2.53.2           Metrics collection
Node Exporter  1.8.2            System metrics agent
Grafana        v11.3.0          Monitoring dashboards
AWS EC2        us-east-1        Cloud infrastructure
```

---

## 🔄 How the Pipeline Works

Every `git push` to this repo triggers Jenkins automatically via **GitHub Webhook**.

```
git push
   ↓
GitHub Webhook
   ↓
Jenkins (50.17.98.208:8080)
   ↓
┌─────────────────────────────────────────────────────────┐
│  Stage 1 │ Checkout SCM              │  228 ms          │
│  Stage 2 │ Tool Install (Maven)      │   96 ms          │
│  Stage 3 │ Build (mvn clean package) │   12 s           │
│  Stage 4 │ Generate Test Reports     │  184 ms          │
│  Stage 5 │ Create Docker Image       │    1 s           │
│  Stage 6 │ Docker Login + Push       │  4 s + 4.8 s     │
│  Stage 7 │ Config & Deployment       │  3 min 3 s       │
└─────────────────────────────────────────────────────────┘
   ↓
Terraform provisions fresh EC2 (t2.micro) in 1 min 26 sec
   ↓
Ansible SSHes in → installs Docker → runs container
   ↓
🌐 App live at http://52.55.21.187:8084
```

---

## 📁 Repository Structure

```
Banking-CI-CD-Pipeline/
│
├── src/                          # Java Spring Boot source code
├── terraform-files/
│   ├── main.tf                   # EC2 provisioning + local-exec provisioners
│   ├── provider.tf               # AWS provider (us-east-1)
│   ├── ansible.cfg               # Ansible configuration
│   ├── ansibleplaybook.yml       # Docker install + container deploy
│   ├── inventory                 # Auto-filled by Terraform local-exec
│   └── mykey-project.pem         # SSH key for EC2 access
├── Dockerfile                    # Containerizes the Spring Boot JAR
├── jenkinsfile                   # Declarative pipeline — 7 stages
├── pom.xml                       # Maven project config
├── portfolio.html                # Project portfolio page
├── architecture.html             # DevOps architecture diagram
└── README.md
```

---

## 📋 Jenkinsfile Breakdown

```groovy
pipeline {
    agent any
    tools { maven "M2_HOME" }

    stages {

        // Stage 1+2: Clone repo and build JAR
        stage('Build') {
            steps {
                git 'https://github.com/Jhansi-112/Banking-CI-CD-Pipeline.git'
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }

        // Stage 3: Publish Maven Surefire HTML report
        stage('Generate Test Reports') {
            steps {
                publishHTML([reportDir: 'target/surefire-reports',
                             reportFiles: 'index.html',
                             reportName: 'HTML Report'])
            }
        }

        // Stage 4: Build Docker image
        stage('Create Docker Image') {
            steps {
                sh 'docker build -t jhansi977/banking-project-demo:1.0 .'
            }
        }

        // Stage 5: Login to DockerHub
        stage('Docker-Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-login',
                    passwordVariable: 'dockerpassword',
                    usernameVariable: 'dockerlogin')]) {
                    sh 'docker login -u ${dockerlogin} -p ${dockerpassword}'
                }
            }
        }

        // Stage 6: Push image to DockerHub
        stage('Push-Image') {
            steps {
                sh 'docker push jhansi977/banking-project-demo:1.0'
            }
        }

        // Stage 7: Terraform + Ansible — provision and deploy
        stage('Config & Deployment') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    credentialsId: 'AWS-ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
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

## 🏗️ Terraform — main.tf

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
    inline = ["echo 'wait to start the instance'"]
  }

  tags = { Name = "test-server" }

  # Write new EC2 IP into Ansible inventory
  provisioner "local-exec" {
    command = "echo ${aws_instance.test-server.public_ip} > inventory"
  }

  # Immediately run Ansible after EC2 is ready
  provisioner "local-exec" {
    command = "ansible-playbook /var/lib/jenkins/workspace/BankingProject/terraform-files/ansibleplaybook.yml"
  }
}
```

---

## 📜 Ansible Playbook

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

**Result:**
```
PLAY RECAP
52.55.21.187 : ok=5  changed=4  unreachable=0  failed=0  skipped=0
```

---

## 📊 Monitoring Stack

### Prometheus — `54.91.220.8:9090`

Scrape config in `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["54.91.220.8:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["52.55.21.187:9100"]
```

### Target Status

```
node_exporter  →  52.55.21.187:9100  →  UP ✅  →  scrape: 13.8s  →  16ms
prometheus     →  54.91.220.8:9090   →  UP ✅  →  scrape:  8.4s  →  5.78ms
```

### Grafana — `54.91.220.8:3000`

- Version: **v11.3.0 Enterprise**
- Data source: Prometheus `http://54.91.220.8:9090`
- Dashboard: **Server Metrics — CPU / Memory / Disk / Network**
- Host monitored: `52.55.21.187:9100`
- Status: *"Successfully queried the Prometheus API."* ✅

---

## 🐳 Docker

```bash
# Pull image from DockerHub
docker pull jhansi977/banking-project-demo:1.0

# Run locally
docker run -itd -p 8084:8081 jhansi977/banking-project-demo:1.0

# Container details from live project
CONTAINER ID   IMAGE                                COMMAND
0c8064116d89   jhansi977/banking-project-demo:1.0   "java -jar /app.jar"
STATUS: Up  |  PORTS: 0.0.0.0:8084->8081/tcp  |  NAME: exciting_morse
```

---

## 🔧 Jenkins Setup — One Time Commands

```bash
# Java + Jenkins
sudo yum install java-17-amazon-corretto-devel -y
yum install jenkins -y
systemctl enable jenkins && systemctl start jenkins

# Set environment variables in .bashrc
JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
M2_HOME=/opt/maven
PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin

# Maven 3.9.9
cd /opt
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
tar -xvzf apache-maven-3.9.9-bin.tar.gz && mv apache-maven-3.9.9 maven

# Docker
yum install docker -y
usermod -aG docker jenkins
systemctl enable docker && systemctl restart docker

# Ansible + Terraform + Git
yum install ansible git -y
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform

# Give Jenkins sudo access — add this line in /etc/sudoers
jenkins ALL=NOPASSWD: ALL
```

---

## ❌ Problems I Hit & How I Fixed Them

**1. Jenkins could not run Terraform or Docker**
- Cause: Jenkins user had no sudo rights
- Fix: Added `jenkins ALL=NOPASSWD: ALL` to `/etc/sudoers`

**2. Ansible playbook did not trigger after Terraform**
- Cause: The inventory file was not getting the new EC2 IP
- Fix: Used `local-exec` in Terraform — `echo ${public_ip} > inventory` — then immediately calls ansible-playbook

**3. Terraform SSH failed on new EC2**
- Cause: PEM file permissions were 644 which is too open
- Fix: `sudo chmod 600 mykey-project.pem` before `terraform apply`

**4. Docker push gave authentication required error**
- Cause: DockerHub credentials were not passed into the pipeline
- Fix: Wrapped with `withCredentials([usernamePassword(credentialsId: 'Docker-login'...)])`

**5. Grafana showed No Data on dashboard**
- Cause: Prometheus data source URL was wrong
- Fix: Set URL to `http://54.91.220.8:9090` in Grafana Connections Data Sources

**6. Node Exporter metrics not appearing in Prometheus**
- Cause: Wrong target IP in `prometheus.yml`
- Fix: Updated scrape target to `52.55.21.187:9100`

**7. App not reachable after container started**
- Cause: App runs internally on port 8081 not 8080
- Fix: Changed docker run to `-p 8084:8081` in the Ansible playbook

---

## 📸 Screenshots

All 40 screenshots are in the [`/screenshots`](./screenshots/) folder.

| # | File | What it shows |
|---|------|---------------|
| 01 | `file1.PNG` | Jenkins .bashrc — JAVA_HOME, M2_HOME, PATH |
| 02 | `file2.PNG` | Jenkins 2.462.3 + Java 17 + Maven 3.9.9 verified |
| 03 | `file3.PNG` | Jenkins unlock screen at 50.17.98.208:8080 |
| 04 | `file4.PNG` | git, Docker, Ansible all version checked |
| 05 | `file5.PNG` | Terraform 1.9.8 installed |
| 06 | `file6.PNG` | DockerHub — image pushed tags 1.0 + 3.0 |
| 07 | `file7.PNG` | Jenkins credentials — Docker, SSH, AWS |
| 08 | `file8.PNG` | ansible.cfg in GitHub repo |
| 09 | `file9.PNG` | ansibleplaybook.yml in GitHub repo |
| 10 | `file10.PNG` | main.tf in GitHub repo |
| 11 | `file11.PNG` | Jenkinsfile — Build, Test, Docker stages |
| 12 | `file12.PNG` | Jenkinsfile — Login, Push, Deploy stages |
| 13 | `file13.PNG` | Sudoers — jenkins NOPASSWD |
| 14 | `file14.PNG` | Jenkins Build #4 — all 7 stages green |
| 15-21 | `file15-21.PNG` | Full pipeline steps with timings |
| 22 | `file22.PNG` | Terraform init success |
| 23 | `file23.PNG` | Terraform apply + Ansible ok=5 changed=4 |
| 24 | `file24.PNG` | AWS EC2 test-server Running |
| 25 | `file25.PNG` | Banking app live at 52.55.21.187:8084 |
| 26 | `file26.PNG` | Prometheus installed on monitoring EC2 |
| 27 | `file27.PNG` | Node Exporter on test-server |
| 28-29 | `file28-29.PNG` | Node Exporter UI + raw /metrics |
| 30 | `file30.PNG` | prometheus.yml scrape config |
| 31 | `file31.PNG` | Prometheus targets — both UP |
| 32-33 | `file32-33.PNG` | Grafana installed + server logs |
| 34 | `file34.PNG` | Grafana login page |
| 35-36 | `file35-36.PNG` | Grafana datasource + API success |
| 37-39 | `file37-39.PNG` | Grafana CPU, Memory, Load, Disk dashboards |
| 40 | `file40.PNG` | Docker ps — container Up on Jenkins EC2 |

---

## 🚀 Run This Project Yourself

**Step 1** — Fork this repo

**Step 2** — Launch a t2.medium EC2 (Amazon Linux, us-east-1) and install all tools from the Jenkins Setup section above

**Step 3** — Add 3 credentials in Jenkins:

```
ID: Docker-login       → DockerHub username + password
ID: terraform-ansible  → SSH private key (PEM file content)
ID: AWS-ID             → AWS Access Key ID + Secret Key
```

**Step 4** — Create a Jenkins Pipeline job:

```
Name:        BankingProject
Type:        Pipeline
SCM:         Git
URL:         https://github.com/Jhansi-112/Banking-CI-CD-Pipeline.git
Script Path: jenkinsfile
```

**Step 5** — Add your `.pem` key as `mykey-project.pem` inside `terraform-files/`

**Step 6** — Click **Build Now** or push to GitHub — all 7 stages run automatically

**Step 7** — Set up a second EC2 for monitoring and install Prometheus + Node Exporter + Grafana using the commands in the Monitoring Stack section

---

## 🔗 Links

| Resource | Link |
|----------|------|
| 🌐 Portfolio | [View Portfolio](./portfolio.html) |
| 🏗️ Architecture | [View Diagram](./architecture.html) |
| 🐳 DockerHub | [jhansi977/banking-project-demo](https://hub.docker.com/r/jhansi977/banking-project-demo) |
| 💻 GitHub | [Jhansi-112/Banking-CI-CD-Pipeline](https://github.com/Jhansi-112/Banking-CI-CD-Pipeline) |

---

*Built by **Jhansi** — Star Agile DevOps Training · AWS us-east-1 · Oct 2024*
