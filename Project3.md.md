# Flask Student Registration Application – Jenkins End‑to‑End Deployment

This repository documents a **complete real‑world DevOps project** where a **Flask web application** is deployed on **AWS EC2** using **Jenkins CI/CD**, **MariaDB**, and **systemd** — with **all automation handled entirely through Jenkinsfile**.

---

##  Project Overview

This project demonstrates how to:

- Deploy a Flask web application on AWS EC2
- Automate setup using Jenkins Pipeline
- Install and configure MariaDB automatically
- Create database, user, and tables via Jenkins
- Run Flask as a production‑ready service using systemd
- Ensure application survives server reboot
- Perform one‑click deployment using Jenkins

The application allows users to **register students** through a web form and stores the data in a MariaDB database.

---

##  Technology Stack

| Layer | Technology |
|------|------------|
| OS | Amazon Linux 2023 |
| Backend | Python Flask |
| Database | MariaDB 10.5 |
| CI/CD | Jenkins |
| Service Manager | systemd |
| Version Control | GitHub |
| Cloud | AWS EC2 |

---

##  Architecture Flow

```
Developer → GitHub → Jenkins Pipeline
                     ↓
            System & Package Setup
                     ↓
               Install MariaDB
                     ↓
          Create DB + User + Table
                     ↓
            Setup Python venv
                     ↓
          Create systemd service
                     ↓
              Run Flask App
```

---

##  Project Structure

```
INTERNSHIP_PROJECT_3/
│── app.py
│── requirements.txt
│── jenkinsfile
│── README.md
│── templates/
│   └── register.html
```

---

##  Jenkinsfile – Single Source of Automation

All setup and deployment steps are executed **only via Jenkinsfile**.

### Tasks handled by Jenkinsfile

- System update
- Install Git, Python, pip
- Install and start MariaDB
- Create database, user, privileges, table
- Clone GitHub repository
- Create Python virtual environment
- Install dependencies
- Create systemd service
- Start Flask application
- Verify deployment

---

##  Final Jenkinsfile Used

```groovy
pipeline {
    agent any

    environment {
        APP_DIR = "/var/lib/jenkins/workspace/Registration_Deployment"
        SERVICE_NAME = "flaskapp"
        DB_NAME = "studentsdb"
        DB_USER = "flaskuser"
        DB_PASS = "flaskpass"
    }

    stages {
        stage('System Update & Base Packages') {
            steps {
                sh '''
                sudo dnf update -y
                sudo dnf install -y git python3 python3-pip
                '''
            }
        }

        stage('Install MariaDB') {
            steps {
                sh '''
                sudo dnf install -y mariadb105-server mariadb105 || true
                sudo systemctl enable mariadb
                sudo systemctl start mariadb
                '''
            }
        }

        stage('Configure Database') {
            steps {
                sh """
                sudo mysql <<EOF
                CREATE DATABASE IF NOT EXISTS ${DB_NAME};
                CREATE USER IF NOT EXISTS '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASS}';
                GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';
                FLUSH PRIVILEGES;
                USE ${DB_NAME};
                CREATE TABLE IF NOT EXISTS students (
                    id INT AUTO_INCREMENT PRIMARY KEY,
                    name VARCHAR(100),
                    email VARCHAR(100),
                    phone VARCHAR(20),
                    course VARCHAR(100),
                    address TEXT
                );
                EOF
                """
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Jirage/INTERNSHIP_PROJECT_3.git'
            }
        }

        stage('Python Virtual Environment') {
            steps {
                sh '''
                python3 -m venv venv || true
                source venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Create systemd Service') {
            steps {
                sh """
                sudo tee /etc/systemd/system/${SERVICE_NAME}.service > /dev/null <<EOF
                [Unit]
                Description=Flask Student Registration App
                After=network.target mariadb.service

                [Service]
                User=jenkins
                WorkingDirectory=${APP_DIR}
                ExecStart=${APP_DIR}/venv/bin/python app.py
                Restart=always

                [Install]
                WantedBy=multi-user.target
                EOF
                """
            }
        }

        stage('Start Flask Service') {
            steps {
                sh '''
                sudo systemctl daemon-reload
                sudo systemctl enable flaskapp
                sudo systemctl restart flaskapp
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                sleep 5
                sudo ss -tulnp | grep 5000
                curl -I http://localhost:5000
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful — Flask App is LIVE on port 5000"
        }
        failure {
            echo "❌ Deployment Failed — Check logs"
        }
    }
}
```

---

##  Access Application

```
http://<EC2_PUBLIC_IP>:5000
```

---

##  Verification Commands

```bash
sudo systemctl status flaskapp
sudo ss -tulnp | grep 5000
curl http://localhost:5000
```

---

##  Common Issues & Fixes

### Port 5000 not accessible
- Open port **5000** in EC2 Security Group

### Database access denied
- Verify database user & password
- Ensure correct privileges

### Application not running
```bash
sudo journalctl -u flaskapp -n 50
```

##  Conclusion

This project demonstrates real‑world DevOps skills including:

- CI/CD pipeline automation
- Linux administration
- Database provisioning
- Application deployment
- Production service management

---

#  output

![](./img/Screenshot%202025-12-30%20151943.png)

![](./img/Screenshot%202025-12-30%20152153.png)

![](./img/Screenshot%202025-12-30%20152202.png)

![](./img/Screenshot%202025-12-30%20152217.png)

![](./img/Screenshot%202025-12-30%20151837.png)

![](./img/Screenshot%202025-12-30%20151815.png)

![](./img/Screenshot%202025-12-30%20151233.png)

![](./img/Screenshot%202025-12-30%20151547.png)