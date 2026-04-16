# ⚡ TechFlow — Real-Time DevOps Project Management Platform

<div align="center">

![Docker](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Flask](https://img.shields.io/badge/Flask_3.0-000000?style=for-the-badge&logo=flask&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL_8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins_CI/CD-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Trivy](https://img.shields.io/badge/Trivy_Scan-1904DA?style=for-the-badge&logo=aqua&logoColor=white)
![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)

<br/>

**A production-grade Agile Project Management Dashboard — built with Flask, MySQL, and Docker.**
**Complete CI/CD pipeline using Jenkins + Trivy + SonarQube + Docker Scout.**

<br/>

*Designed & Built by **Akshay Sawant** — AWS DevOps Engineer | Hinjewadi, Pune*

---

```
🌐 Frontend:  http://localhost:3000   (Nginx + HTML/CSS/JS)
⚙️  Backend:   http://localhost:5000   (Flask REST API)
🗄️  Database:  localhost:3306          (MySQL techflow_db)
```

</div>

---

## 📌 Table of Contents
- [What is TechFlow?](#-what-is-techflow)
- [Application Features](#-application-features)
- [Project Structure](#-project-structure)
- [Architecture](#-architecture)
- [Run Locally](#-run-locally-step-by-step)
- [Jenkins Pipeline](#-jenkins-cicd-pipeline)
- [Jenkins Setup Steps](#-jenkins-setup-steps)
- [API Reference](#-api-reference)
- [AWS EC2 Deployment](#-aws-ec2-deployment)
- [Accessing MySQL](#-accessing-mysql-database)

---

## 🎯 What is TechFlow?

TechFlow is a **full-stack DevOps project management platform** that lets teams:

```
📁 Manage Projects    → Create, track, update software projects
✅ Manage Tasks       → Kanban board, sprint tracking, story points
🐛 Track Bugs         → Report and track bugs by severity
🚀 Log Deployments    → Track all deployments by version & environment
👥 Manage Team        → Add and view team members and their roles
📊 View Dashboard     → Live stats from all 4 database tables
```

---

## ✨ Application Features

| Feature | Description |
|---------|-------------|
| 📊 **Live Dashboard** | Real-time stats, activity feed, deployment history |
| 📁 **Project Management** | Create projects with tech stack, priority, progress |
| ✅ **Task Board** | Full Kanban view (Todo → In Progress → Review → Done) |
| 🐛 **Bug Tracker** | Report bugs with severity, environment, project link |
| 🚀 **Deployment Log** | Track every deployment with version and status |
| 👥 **Team Management** | Add team members with role and emoji avatar |
| 🔄 **Auto Refresh** | Dashboard auto-refreshes every 30 seconds |
| 🔍 **Search** | Global search across projects and tasks |
| 🌑 **Dark Theme** | Glassmorphism dark UI with animated particles |

---

## 📁 Project Structure

```
techflow-app/
│
├── 📄 docker-compose.yml         ← Orchestrates all 3 containers
├── 📄 Jenkinsfile                ← Complete CI/CD pipeline (11 stages)
├── 📄 .gitignore
├── 📄 README.md
│
├── 📁 backend/                   ← Flask REST API
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py                    ← All API endpoints
│
├── 📁 frontend/                  ← Nginx + Static HTML/CSS/JS
│   ├── Dockerfile
│   ├── nginx.conf                ← Proxies /api → backend:5000
│   └── index.html                ← Full SPA dashboard
│
└── 📁 mysql-init/
    └── 01-init.sql               ← Creates schema + seeds 40+ records
```

---

## 🏗️ Architecture

```
Browser
  │
  ▼ Port 3000
┌───────────────────────────────────┐
│   Frontend Container (Nginx)      │
│   - Serves index.html             │
│   - /api/* → proxy_pass          │
│              → backend:5000       │
└────────────────┬──────────────────┘
                 │ HTTP (internal docker network)
                 ▼
┌───────────────────────────────────┐
│   Backend Container (Flask:5000)  │
│   - REST API (20+ endpoints)      │
│   - MySQL connection pool         │
└────────────────┬──────────────────┘
                 │ MySQL connector
                 ▼
┌───────────────────────────────────┐
│   MySQL Container (:3306)         │
│   - techflow_db                   │
│   - 6 tables: users, projects,    │
│     tasks, bugs, deployments,     │
│     activity_log                  │
│   - Auto-seeded with sample data  │
└───────────────────────────────────┘

All on: techflow-net (Docker bridge)
```

---

## 🚀 Run Locally — Step by Step

### Prerequisites
```bash
docker --version          # Docker 24+
docker compose version    # Docker Compose v2+
```

### Step 1 — Clone the Repo
```bash
git clone https://github.com/social9009/techflow-app.git
cd techflow-app
```

### Step 2 — Build and Start
```bash
# First run (builds images + seeds DB — ~3 min)
docker compose up --build

# Background mode
docker compose up --build -d
```

### Step 3 — Check All Containers
```bash
docker compose ps
# Expected:
# techflow-mysql     Up (healthy)   0.0.0.0:3306->3306/tcp
# techflow-backend   Up (healthy)   0.0.0.0:5000->5000/tcp
# techflow-frontend  Up             0.0.0.0:3000->80/tcp
```

### Step 4 — Open the Application
```
🌐 Dashboard:  http://localhost:3000
⚙️  API Health: http://localhost:5000/api/health
```

### Step 5 — Stop
```bash
docker compose down        # Keep database data
docker compose down -v     # Remove data (fresh start)
```

---

## ⚙️ Jenkins CI/CD Pipeline

The Jenkinsfile contains **11 automated stages** — everything runs without manual intervention:

```
Stage 1:  Git Checkout          → Pull latest code from GitHub
Stage 2:  Trivy FS Scan         → Scan filesystem for CVEs & misconfigs
Stage 3:  SonarQube Analysis    → Static code quality analysis (Python)
Stage 4:  Quality Gate          → Block pipeline if code quality fails
Stage 5:  Verify Docker Compose → Validate docker-compose.yml
Stage 6:  Build Backend Image   → docker build + tag + push to DockerHub
Stage 7:  Build Frontend Image  → docker build + tag + push to DockerHub
Stage 8:  Trivy Image Scans     → Scan both Docker images (parallel)
Stage 9:  Docker Scout Analysis → CVE analysis & recommendations
Stage 10: Deploy with Compose   → docker compose up -d --build
Stage 11: Verify Deployment     → Health checks + API smoke tests
```

### Why Jenkins? What It Automates:
```
Without Jenkins (Manual):
  1. SSH to server
  2. git pull
  3. Run trivy scan manually
  4. Open SonarQube, run scan manually
  5. docker build, docker push (backend)
  6. docker build, docker push (frontend)
  7. docker scout run manually
  8. docker compose down && up
  9. Test manually
  10. Check logs manually
  → 30+ minutes every deployment

With Jenkins Pipeline:
  One git push → EVERYTHING above runs automatically
  → 8-12 minutes, zero manual steps, full audit trail
```

---

## 🔧 Jenkins Setup Steps

### 1. Launch EC2 & Install Tools
```bash
# Launch Ubuntu 24.04 t2.large (30GB) on AWS
# Open ports: 22, 80, 443, 3000, 5000, 8080, 9000, 6443

sudo su
sudo apt update

# Install Jenkins
chmod +x jenkins.sh && ./jenkins.sh

# Install Docker
chmod +x docker.sh && ./docker.sh
sudo chmod 666 /var/run/docker.sock
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

# Install Trivy
chmod +x trivy.sh && ./trivy.sh

# Install Docker Scout
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin
sudo chmod 777 /var/run/docker.sock
```

### 2. Start SonarQube
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
# Access: http://<EC2-IP>:9000 (admin/admin)
```

### 3. Jenkins Configuration
```
# Install Plugins:
SonarQube Scanner, Docker, Docker Commons, Docker Pipeline, 
Docker API, docker-build-step, Pipeline stage view,
Kubernetes (optional), Prometheus metrics

# Configure SonarQube:
Manage Jenkins → System → SonarQube servers → Add sonar
  Name: sonar
  URL: http://localhost:9000
  Token: (create in SonarQube → Security → Users → Tokens)

# Configure Docker credentials:
Manage Jenkins → Credentials → Global → Add Credentials
  Kind: Username/Password
  Username: <DockerHub username>
  Password: <DockerHub password>
  ID: docker

# Configure SonarQube Scanner tool:
Manage Jenkins → Tools → SonarQube Scanner
  Name: sonar-scanner (auto-install latest)

# Create SonarQube Webhook:
SonarQube → Administration → Webhooks → Create
  Name: jenkins
  URL: http://<JENKINS-IP>:8080/sonarqube-webhook/
```

### 4. Create Jenkins Pipeline Job
```
Jenkins → New Item → Pipeline
→ Pipeline script from SCM
→ SCM: Git
→ URL: https://github.com/social9009/techflow-app.git
→ Branch: main
→ Script Path: Jenkinsfile
→ Save → Build Now
```

---

## 📡 API Reference

```
GET  /api/health              Service health + DB status
GET  /api/dashboard           All stats, activity, recent data

GET  /api/projects            All projects (filter: status, priority, search)
POST /api/projects            Create project
GET  /api/projects/{id}       Get project + tasks + bugs
DELETE /api/projects/{id}     Delete project

GET  /api/tasks               All tasks (filter: project_id, status, assigned_to)
POST /api/tasks               Create task
PUT  /api/tasks/{id}          Update task (status, priority, etc.)
DELETE /api/tasks/{id}        Delete task

GET  /api/bugs                All bugs (filter: status, severity)
POST /api/bugs                Report new bug

GET  /api/deployments         All deployment logs
POST /api/deployments         Log a deployment

GET  /api/users               All active team members
POST /api/users               Add team member
```

---

## ☁️ AWS EC2 Deployment

```bash
# Launch t2.large, Ubuntu 22.04, ports: 22, 80, 3000, 5000, 8080, 9000

# SSH in
ssh -i key.pem ubuntu@<EC2-IP>

# Install Docker
sudo apt update
sudo apt install -y docker.io docker-compose-plugin git
sudo systemctl start docker
sudo usermod -aG docker ubuntu && newgrp docker

# Clone and deploy
git clone https://github.com/social9009/techflow-app.git
cd techflow-app
docker compose up --build -d

# Access
# Frontend: http://<EC2-IP>:3000
# Backend:  http://<EC2-IP>:5000/api/health
```

---

## 🗄️ Accessing MySQL Database

```bash
# Connect to MySQL container
docker exec -it techflow-mysql mysql -u root -ptechflow@2025

# Common queries
SHOW DATABASES;
USE techflow_db;
SHOW TABLES;

-- View all projects
SELECT id, name, status, priority, progress FROM projects;

-- View task distribution
SELECT status, COUNT(*) as count FROM tasks GROUP BY status;

-- View open bugs
SELECT title, severity, environment FROM bugs WHERE status = 'open';

-- View recent deployments
SELECT p.name, d.version, d.environment, d.status
FROM deployments d
JOIN projects p ON d.project_id = p.id
ORDER BY d.created_at DESC LIMIT 5;
```

---

## 🔗 Docker Project Series

[![Docker-6b](https://img.shields.io/badge/Docker--6b-IIIU_Python_Microservices-2496ED?style=for-the-badge&logo=docker)](https://github.com/social9009/indian-university-portal)
[![Docker-5](https://img.shields.io/badge/Docker--5-Casa_Royale-FF6600?style=for-the-badge)](https://github.com/social9009/casa-royale)
[![Docker-3](https://img.shields.io/badge/Docker--3-Go_Multi--Stage-00ADD8?style=for-the-badge&logo=go)](https://github.com/social9009/Go-App)

---

## 👨‍💻 Author

**Akshay Sawant** — AWS DevOps Engineer | AWS Solutions Architect Associate

[![Email](https://img.shields.io/badge/Email-akshaysawant9009@gmail.com-D14836?style=flat-square&logo=gmail)](mailto:akshaysawant9009@gmail.com)
[![GitHub](https://img.shields.io/badge/GitHub-social9009-181717?style=flat-square&logo=github)](https://github.com/social9009)
[![Phone](https://img.shields.io/badge/Phone-+91_9096505065-25D366?style=flat-square&logo=whatsapp)](tel:+919096505065)

---

<div align="center">

⭐ **Star this repo if it helped you understand Jenkins CI/CD + Docker + DevSecOps!** ⭐

*Full DevOps Cycle: Code → Trivy Scan → SonarQube → Docker Build → Scout → Deploy → Verify*

</div>
