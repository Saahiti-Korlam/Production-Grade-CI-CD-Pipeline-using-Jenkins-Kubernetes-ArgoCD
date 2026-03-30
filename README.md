# 🚀 Production-Grade CI/CD Pipeline using Jenkins, Kubernetes & ArgoCD

## 📌 Project Overview
This project demonstrates a complete CI/CD pipeline for a containerized application using Jenkins for Continuous Integration and ArgoCD for GitOps-based Continuous Delivery on AWS EKS.

---

## 🏗️ Architecture Flow
- Developers push code to feature branches
- Pull Requests → reviewed → merged into main branch
- Jenkins builds Docker image & pushes to DockerHub
- ArgoCD automatically deploys to Kubernetes (EKS)

---

## 📁 Repository Structure
ecommerce-app-repo/
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── k8s/
│ ├── deployment.yaml
│ └── service.yaml
└── argocd/
└── application.yaml


---

## ⚙️ Phase 1: Infrastructure Setup
**Summary:** Provisioned cloud infrastructure and prepared the environment.

- Launched EC2 instance (Ubuntu)
- Updated system packages
- Installed required dependencies

---

## 🔧 Phase 2: Jenkins Installation & Setup
**Summary:** Set up Jenkins as the CI tool.

- Installed Java & Jenkins
- Configured Jenkins UI
- Created admin user and accessed dashboard

---

## 🐳 Phase 3: Docker Installation
**Summary:** Enabled containerization capability.

- Installed Docker engine
- Verified Docker setup
- Allowed Jenkins to use Docker

---

## ☸️ Phase 4: Kubernetes & AWS Setup
**Summary:** Configured Kubernetes cluster on AWS.

- Installed kubectl, AWS CLI, eksctl
- Created EKS cluster
- Configured kubeconfig access for Jenkins

---

## 🔐 Phase 5: Jenkins Configuration
**Summary:** Integrated external systems with Jenkins.

- Installed required plugins (Pipeline, Docker, Kubernetes)
- Configured DockerHub credentials
- Configured GitHub credentials
- Set up Docker tool in Jenkins

---

## 📦 Phase 6: Application Containerization
**Summary:** Built Docker image for the application.

- Created Dockerfile
- Built Python-based container image
- Exposed application port

---

## 🔄 Phase 7: CI Pipeline (Jenkins)
**Summary:** Automated build and image push process.

- Trigger on code changes
- Build Docker image with version tag
- Push image to DockerHub

---

## 🚀 Phase 8: Kubernetes Deployment (GitOps)
**Summary:** Automated deployment using GitOps approach.

- Created Kubernetes manifests (Deployment, Service)
- Jenkins updates image tag in manifests
- Changes pushed to GitHub

---

## 🔁 Phase 9: ArgoCD Setup & CD Pipeline
**Summary:** Enabled automated Kubernetes deployment.

- Installed ArgoCD via Helm
- Created ArgoCD application
- Enabled auto-sync & pruning
- Deployed application to EKS

---

## 🌿 Phase 10: Multi-Branch Strategy
**Summary:** Implemented scalable development workflow.

- Feature branches (featureA, featureB)
- Pull request-based merging
- Automated pipeline trigger on merge
- Versioned deployments (v1, v2, v3)

---

## 📊 Phase 11: Monitoring Setup
**Summary:** Implemented observability stack.

- Installed Prometheus & Grafana using Helm
- Configured dashboards:
  - Kubernetes Nodes
  - Pods
  - Deployments
  - Cluster metrics

---

## ✅ Key Features
- Multi-branch CI/CD pipeline
- GitOps-based deployment (ArgoCD)
- Automated Docker image versioning
- Kubernetes auto-deployment
- Real-time monitoring with Grafana
- Scalable and production-ready architecture
