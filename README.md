
# 🐳 Three-Tier Application Deployment Using Docker Compose

![Docker](https://img.shields.io/badge/Docker-29.1.3-blue?logo=docker)
![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws)
![Ubuntu](https://img.shields.io/badge/Ubuntu-26.04_LTS-purple?logo=ubuntu)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql)
![Nginx](https://img.shields.io/badge/Nginx-Alpine-green?logo=nginx)
![Status](https://img.shields.io/badge/Status-Live-brightgreen)

---

## 📌 Project Overview

Deployed a **production-grade three-tier application** on AWS EC2 using Docker Compose.

| Layer | Technology | Port |
|-------|-----------|------|
| Frontend | Nginx Alpine | 80 |
| Backend | Apache Tomcat | 8080 |
| Database | MySQL 8.0 | 3306 |

> 🔗 **Live URL:** http://65.0.179.143 (Frontend) | http://65.0.179.143:8080 (Backend)

---

## 🏗️ Architecture Diagram

```
                        ┌─────────────────────────────────────┐
                        │         AWS EC2 Instance             │
                        │    (Ubuntu 26.04 LTS - Mumbai)       │
                        │                                      │
   🌍 Internet ──────► │  ┌─────────────────────────────────┐ │
                        │  │      Docker Engine 29.1.3       │ │
                        │  │                                 │ │
                        │  │  ┌──────────┐  Port 80         │ │
                        │  │  │  Nginx   │◄─────────────────┼─┼── User Browser
                        │  │  │ Frontend │                  │ │
                        │  │  └────┬─────┘                  │ │
                        │  │       │ app-network             │ │
                        │  │  ┌────▼─────┐  Port 8080       │ │
                        │  │  │  Tomcat  │◄─────────────────┼─┼── API Calls
                        │  │  │ Backend  │                  │ │
                        │  │  └────┬─────┘                  │ │
                        │  │       │ app-network             │ │
                        │  │  ┌────▼─────┐  Port 3306       │ │
                        │  │  │  MySQL   │                  │ │
                        │  │  │ Database │                  │ │
                        │  │  └────┬─────┘                  │ │
                        │  │       │                         │ │
                        │  │  ┌────▼─────┐                  │ │
                        │  │  │ db-data  │ (Docker Volume)  │ │
                        │  │  └──────────┘                  │ │
                        │  └─────────────────────────────────┘ │
                        └─────────────────────────────────────┘
```

---

## 📁 Project Structure

```
three-tier-docker-app/
│
├── docker-compose.yml          # Main orchestration file
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD Pipeline
├── screenshots/
│   ├── containers-running.png  # Docker compose ps output
│   ├── nginx-frontend.png      # Nginx page in browser
│   ├── tomcat-backend.png      # Tomcat page in browser
│   └── cicd-success.png        # GitHub Actions green tick
└── README.md                   # This file
```

---

## 🛠️ Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| Docker | 29.1.3 | Containerization platform |
| Docker Compose | Built-in (v2) | Multi-container orchestration |
| Nginx | Alpine | Frontend web server |
| Apache Tomcat | Latest | Backend application server |
| MySQL | 8.0 | Relational database |
| AWS EC2 | m7i-flex.large | Cloud compute instance |
| Ubuntu | 26.04 LTS | Server operating system |
| Git | Latest | Version control |

---

## ✅ Prerequisites

```bash
# Required:
✅ AWS Account
✅ EC2 instance running Ubuntu 26.04 LTS
✅ Security Group with these ports open:
   - 22  (SSH)
   - 80  (HTTP - Frontend)
   - 8080 (Tomcat - Backend)
   - 3306 (MySQL - Database)
✅ Internet access on EC2
```

---

## 🚀 Deployment Steps

### Step 1: Launch EC2 Instance

```
AWS Console → EC2 → Launch Instance
├── Name: three-tier-project
├── AMI: Ubuntu Server 26.04 LTS (HVM)
├── Instance Type: m7i-flex.large (Free Tier Eligible)
├── Key Pair: Create new → download .pem file
├── Security Group: Allow ports 22, 80, 8080, 3306
└── Storage: 20 GiB gp3
```

### Step 2: Connect to EC2

```bash
# Via AWS Console:
EC2 Dashboard → Select Instance → Connect → EC2 Instance Connect → Connect

# Via SSH (from local machine):
ssh -i your-key.pem ubuntu@YOUR-PUBLIC-IP
```

### Step 3: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 4: Install Docker

```bash
# Install Docker
sudo apt install docker.io -y

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (no sudo needed)
sudo usermod -aG docker ubuntu
newgrp docker

# Verify installation
docker --version
# Output: Docker version 29.1.3
```

### Step 5: Verify Docker Compose

```bash
docker compose --version
# Note: Docker 29.x has Compose built-in
# Use 'docker compose' (space) NOT 'docker-compose' (hyphen)
```

### Step 6: Create Project Directory

```bash
mkdir three-tier-app && cd three-tier-app
```

### Step 7: Create docker-compose.yml

```bash
cat > docker-compose.yml << 'EOF'
services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    image: tomcat:latest
    ports:
      - "8080:8080"
    depends_on:
      - database
    environment:
      - DB_HOST=database
      - DB_PORT=3306
      - DB_NAME=appdb
      - DB_USER=appuser
      - DB_PASSWORD=apppass123
    networks:
      - app-network

  database:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass123
      - MYSQL_DATABASE=appdb
      - MYSQL_USER=appuser
      - MYSQL_PASSWORD=apppass123
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
EOF
```

### Step 8: Deploy Application

```bash
docker compose up -d
```

Expected Output:
```
✔ database Pulled     12.8s
✔ frontend Pulled      5.6s
✔ backend Pulled       8.7s
✔ Network three-tier-app_app-network  Created
✔ Volume three-tier-app_db-data       Created
✔ Container three-tier-app-database-1 Started
✔ Container three-tier-app-backend-1  Started
✔ Container three-tier-app-frontend-1 Started
[+] Running 5/5
```

### Step 9: Verify Running Containers

```bash
docker compose ps
```

Expected Output:
```
NAME                          IMAGE           STATUS          PORTS
three-tier-app-backend-1      tomcat:latest   Up 44 seconds   0.0.0.0:8080->8080/tcp
three-tier-app-database-1     mysql:8.0       Up 45 seconds   0.0.0.0:3306->3306/tcp
three-tier-app-frontend-1     nginx:alpine    Up 44 seconds   0.0.0.0:80->80/tcp
```

### Step 10: Access Application

```
Frontend (Nginx):  http://YOUR-EC2-PUBLIC-IP
Backend (Tomcat):  http://YOUR-EC2-PUBLIC-IP:8080
Database (MySQL):  Internal - Port 3306
```

---

## 📄 Source Code

### docker-compose.yml

```yaml
services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    image: tomcat:latest
    ports:
      - "8080:8080"
    depends_on:
      - database
    environment:
      - DB_HOST=database
      - DB_PORT=3306
      - DB_NAME=appdb
      - DB_USER=appuser
      - DB_PASSWORD=apppass123
    networks:
      - app-network

  database:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass123
      - MYSQL_DATABASE=appdb
      - MYSQL_USER=appuser
      - MYSQL_PASSWORD=apppass123
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
```

---

## 🔧 Useful Commands

```bash
# Start all containers (detached mode)
docker compose up -d

# Stop all containers
docker compose down

# View running containers
docker compose ps

# View logs of all containers
docker compose logs

# View logs of specific container
docker compose logs frontend
docker compose logs backend
docker compose logs database

# Restart all containers
docker compose restart

# Pull latest images
docker compose pull

# View resource usage
docker stats

# Enter container shell
docker exec -it three-tier-app-frontend-1 sh
docker exec -it three-tier-app-backend-1 bash
docker exec -it three-tier-app-database-1 bash
```

---

## 📊 Results

| Component | Status | Access URL |
|-----------|--------|------------|
| Frontend (Nginx) | ✅ Running | http://65.0.179.143 |
| Backend (Tomcat) | ✅ Running | http://65.0.179.143:8080 |
| Database (MySQL) | ✅ Running | Internal port 3306 |
| Docker Network | ✅ Created | app-network (bridge) |
| Docker Volume | ✅ Created | db-data (persistent) |

---

## 📸 Screenshots

> Add your screenshots in the `screenshots/` folder:
> - `screenshots/containers-running.png` → Output of `docker compose ps`
> - `screenshots/nginx-frontend.png` → Nginx page in browser
> - `screenshots/tomcat-backend.png` → Tomcat page in browser

---

## ⚠️ Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Port already in use | Another process using port | `docker compose down` then `up -d` |
| Container not starting | Image pull failed | Check internet connection |
| App not accessible | Security group issue | Check inbound rules in AWS |
| WARN: version obsolete | Old docker-compose.yml format | Remove `version: '3.8'` line |
| Database not connecting | Wrong credentials | Check environment variables |

---

## 🎯 Key Learnings

- Docker containerization and image management
- Multi-container orchestration with Docker Compose
- Docker networking (bridge networks)
- Docker volumes for persistent storage
- Three-tier application architecture
- AWS EC2 instance management
- Security group configuration

---

## 👩‍💻 Author

**Sayali Magdum**
- 🔗 GitHub: [@sayalimagdum0712](https://github.com/sayalimagdum0712)
- 💼 LinkedIn: [Sayali Magdum](https://linkedin.com/in/sayalimagdum)
- 🎯 Role: Aspiring Cloud & DevOps Engineer

---

## 📅 Project Date
June 27, 2026
