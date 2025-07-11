# 🧠 End-to-End DevOps Workflow: GenAI Mental Health Assistant

A production-grade, AI-powered Mental Health Assistant Chatbot built using Flask, Google Gemini, and LangChain — deployed on AWS EKS with an automated CI/CD pipeline. This project showcases DevOps best practices including Kubernetes orchestration, Docker-based containerization, and Jenkins-powered deployment automation.

---

## 🚀 Project Objective

- Build a GenAI chatbot that empathetically assists users with mental health-related queries.
- Showcase complete DevOps workflow using containerization, Kubernetes, CI/CD, and AWS-managed services.
- Deploy in a production-ready cloud-native environment using AWS EKS and ELB.

---

## ⚙️ Tech Stack

### 💬 Application Stack
- **Flask** – Python web framework
- **LangChain** – Framework for orchestrating LLM interactions
- **Google Gemini** – Natural Language Generation (NLG)
- **PostgreSQL** – Stores chat history and user sessions

### 🛠️ DevOps & Deployment Tools
- **eksctl** – Bootstrap and manage AWS EKS cluster
- **Docker** – Containerize Flask app and dependencies
- **Kubernetes** – Orchestrates app and database deployment
- **Helm** – Manages Kubernetes configurations and releases
- **Jenkins** – CI/CD pipeline triggered on GitHub push
- **GitHub** – Version control and webhook integration
- **Docker Hub** – Public image registry
- **AWS ELB** – Exposes the application securely to the internet

---

## 🧰 DevOps Workflow

### 1. 🌐 EKS Provisioning with eksctl
- EKS cluster and node groups are provisioned using `eksctl` configuration files.
- ELB is automatically created and attached via Kubernetes `Service` of type `LoadBalancer`.

### 2. ⚒️ CI/CD Pipeline (Jenkins)
- GitHub webhook triggers Jenkins pipeline automatically.
- Pipeline steps:
  - Clone repository
  - Owasp Dependency Check
  - Sonarqube Scan
  - Build Docker image
  - Push to Docker Hub
  - Connect to EKS
  - Deploy updated image using Helm

### 3. 📦 Kubernetes Deployment
- Helm manages deployment of:
  - Flask backend app
- Application is exposed externally via **AWS ELB** provisioned by a `LoadBalancer` service in Kubernetes.

---

## 🎮 Chatbot Features

- Empathetic and multilingual responses using Google Gemini
- Real-time typing animation in UI
- Secure user login with persistent chat history stored in PostgreSQL
- Fully containerized and scalable Kubernetes architecture

---

## 📦 Docker Hub

👉 [Docker Image on Docker Hub](https://hub.docker.com/r/jayesh9999/mental_health_assistant)

---

## 🚀 Future Enhancements

- Integrate Prometheus + Grafana for monitoring
- Enable HTTPS using AWS ACM or Cert-Manager
- Add Horizontal Pod Autoscaler (HPA) for dynamic scaling

---

## 👨‍💻 Author

**Jayesh Thorat**  
AWS Certified Solutions Architect  
[LinkedIn](https://www.linkedin.com/in/jayesh9999) • [Docker Hub](https://hub.docker.com/u/jayesh9999)

---

> ⭐ Star this repository if you found it helpful or inspiring!