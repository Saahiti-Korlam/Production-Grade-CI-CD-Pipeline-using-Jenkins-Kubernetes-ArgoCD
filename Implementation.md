# 🚀 Multi-Branch CI/CD Pipeline with Jenkins, Docker, EKS, ArgoCD & Monitoring

> A complete end-to-end DevOps project covering infrastructure setup, containerisation, Kubernetes deployment, GitOps with ArgoCD, and monitoring using Prometheus & Grafana.

---

## 📋 Table of Contents

1. [Phase 1 — EC2 Setup & Jenkins Installation](#phase-1--ec2-setup--jenkins-installation)
2. [Phase 2 — Docker Installation](#phase-2--docker-installation)
3. [Phase 3 — Install kubectl, eksctl & AWS CLI](#phase-3--install-kubectl-eksctl--aws-cli)
4. [Phase 4 — Create EKS Cluster](#phase-4--create-eks-cluster)
5. [Phase 5 — Configure Jenkins Plugins & Credentials](#phase-5--configure-jenkins-plugins--credentials)
6. [Phase 6 — Application Code & GitHub Push](#phase-6--application-code--github-push)
7. [Phase 7 — Update Kubeconfig & Jenkins Permissions](#phase-7--update-kubeconfig--jenkins-permissions)
8. [Phase 8 — ArgoCD Setup using Helm](#phase-8--argocd-setup-using-helm)
9. [Phase 9 — Create Multi-Branch Pipeline in Jenkins](#phase-9--create-multi-branch-pipeline-in-jenkins)
10. [Phase 10 — Feature Branch Workflow](#phase-10--feature-branch-workflow)
11. [Phase 11 — Monitoring with Prometheus & Grafana](#phase-11--monitoring-with-prometheus--grafana)

---

## Phase 1 — EC2 Setup & Jenkins Installation

> Launch an Ubuntu EC2 instance (instance type: **m7i-flex.large**). This will be the main server hosting Jenkins, Docker, kubectl, eksctl, and AWS CLI.

### Step 1.1 — Update the system packages

```bash
sudo apt update -y
```

### Step 1.2 — Create the Jenkins installation script

> Jenkins requires Java 17 (Temurin JDK). The script below installs Adoptium JDK 17 and then installs Jenkins from the official stable repository.

```bash
vi Jenkins.sh
```

Paste the following content into `Jenkins.sh`:

```bash
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

### Step 1.3 — Make the script executable and run it

```bash
sudo chmod +x Jenkins.sh
./Jenkins.sh
```

### Step 1.4 — Troubleshooting: If the above script gives errors

> Run these commands manually to re-add the Jenkins GPG key and repository, then install Jenkins.

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

### Step 1.5 — Access Jenkins in the browser

> Open port **8080** in the EC2 Security Group inbound rules.

```
http://<EC2-PUBLIC-IP>:8080
```

### Step 1.6 — Retrieve the initial admin password

> Jenkins auto-generates a one-time password stored on the server. Copy it to complete the setup wizard.

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

> Paste the output in the browser prompt, complete the setup wizard, and set your own credentials.

---

## Phase 2 — Docker Installation

> Docker is required for building and pushing container images from the Jenkins pipeline.

### Step 2.1 — Create the Docker installation script

```bash
vi docker.sh
```

Paste the following content into `docker.sh`:

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

docker --version
```

### Step 2.2 — Troubleshooting: If the script gives errors

> Run the following commands manually to add the Docker GPG key, repository, and install Docker packages.

```bash
# Add Docker's official GPG key
sudo install -m 755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker packages
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify installation
sudo docker run hello-world
```

---

## Phase 3 — Install kubectl, eksctl & AWS CLI

> These three tools are needed to interact with AWS EKS — `kubectl` to manage Kubernetes resources, `eksctl` to create/manage the cluster, and `awscli` to authenticate with AWS.

### Step 3.1 — Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 3.2 — Install AWS CLI

```bash
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### Step 3.3 — Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 3.4 — Attach IAM Role to EC2 Instance

> The EC2 instance needs an IAM role with sufficient permissions to create and manage EKS clusters, EC2 resources, CloudFormation stacks, and VPCs.

```
EC2 Console → Select Instance → Actions → Security → Modify IAM Role → Attach IAM Role → Save
```

---

## Phase 4 — Create EKS Cluster

> This command creates a managed EKS cluster with worker nodes. It uses CloudFormation under the hood and may take **15–20 minutes** to complete.

### Step 4.1 — Create the cluster

```bash
eksctl create cluster \
  --name Fortune500-cluster \
  --region ap-south-1 \
  --node-type t3.small \
  --zones ap-south-1a,ap-south-1b
```

---

## Phase 5 — Configure Jenkins Plugins & Credentials

> Jenkins needs additional plugins for Docker builds, Kubernetes deployments, and pipeline visualisation.

### Step 5.1 — Install required plugins

```
Jenkins Dashboard → Manage Jenkins → Plugins → Available Plugins

Install the following:
  - Pipeline Stage View
  - Docker (select all Docker-related plugins)
  - Kubernetes (select all Kubernetes-related plugins)
```

> Restart Jenkins after all plugins are installed.

### Step 5.2 — Configure Docker in Jenkins Tools

```
Jenkins Dashboard → Manage Jenkins → Tools → Docker Installations → Add Docker → Save
```

### Step 5.3 — Add DockerHub credentials

> Jenkins needs your DockerHub username and password/token to push images.

```
Jenkins Dashboard → Manage Jenkins → Credentials → System → Global Credentials
→ Add Credentials → Kind: Username with Password
  Username : <your-dockerhub-username>
  Password : <your-dockerhub-password-or-token>
  ID       : dockerhub_cred        ← must match exactly as used in Jenkinsfile
```

### Step 5.4 — Add GitHub credentials

> Jenkins needs a GitHub personal access token to clone the repo and push manifest updates.

```
Jenkins Dashboard → Manage Jenkins → Credentials → System → Global Credentials
→ Add Credentials → Kind: Username with Password
  Username : <your-github-username>
  Password : <your-github-personal-access-token>
  ID       : github_cred           ← must match exactly as used in Jenkinsfile
```

---

## Phase 6 — Application Code & GitHub Push

> Push the application source code, Dockerfile, Kubernetes manifests, and Jenkinsfile to the GitHub repository.

### Step 6.1 — Initialise the local Git repository and push

```bash
git init
git add .
git status
git commit -m "first version commit"
git remote add origin https://github.com/<your-username>/<your-repo>.git
git branch -m main
git push -u origin main
```

> Authenticate using your GitHub token when prompted.

---

## Phase 7 — Update Kubeconfig & Jenkins Permissions

> By default, Jenkins runs as the `jenkins` system user, which does not have access to the kubeconfig file stored under `/root`. We must copy the config and grant the correct permissions.

### Step 7.1 — Verify your AWS identity

```bash
aws sts get-caller-identity
```

### Step 7.2 — Update kubeconfig for the EKS cluster

```bash
aws eks update-kubeconfig --region ap-south-1 --name Fortune500-cluster
```

### Step 7.3 — Check which user Jenkins runs as

```bash
ps aux | grep jenkins
```

> Jenkins runs as the `jenkins` user by default. Since `kubectl` config is under `/root`, we must copy it to a path accessible by the `jenkins` user.

### Step 7.4 — Copy kubeconfig to the Jenkins user home directory

```bash
sudo mkdir -p /home/jenkins/.kube
sudo cp /root/.kube/config /home/jenkins/.kube/config
sudo chown -R jenkins:jenkins /home/jenkins/.kube
```

### Step 7.5 — Add Jenkins user to the Docker group

> This allows the Jenkins user to run Docker commands (build, push) without `sudo`.

```bash
sudo usermod -aG docker jenkins
```

### Step 7.6 — Restart Jenkins to apply group changes

```bash
systemctl restart jenkins
```

> Refresh the Jenkins browser page and log in again.

---

## Phase 8 — ArgoCD Setup using Helm

> ArgoCD is a GitOps continuous delivery tool. It watches the GitHub repository and automatically syncs any changes in Kubernetes manifests to the cluster.

### Step 8.1 — Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### Step 8.2 — Add the ArgoCD Helm repository

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### Step 8.3 — Install ArgoCD via Helm

```bash
kubectl create namespace argocd
helm install argocd argo/argo-cd -n argocd
```

### Step 8.4 — Verify ArgoCD pods and services are running

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
```

### Step 8.5 — Expose ArgoCD via LoadBalancer

> By default ArgoCD is only accessible within the cluster. Patching it to LoadBalancer type creates an external AWS ELB for browser access.

```bash
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'

kubectl get svc argocd-server -n argocd
```

> Copy the **EXTERNAL-IP** from the output and open it in your browser.

### Step 8.6 — Retrieve the ArgoCD admin password

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

> Copy the output (stop before `root@...`). Use this with username `admin` to log into ArgoCD.

### Step 8.7 — Create the application in ArgoCD UI

```
ArgoCD UI → New App

  Application Name : movie-app
  Project          : default
  Sync Policy      : Automatic  (enable Prune Resources)
  Source Repo URL  : https://github.com/<your-username>/<your-repo>.git
  Branch           : main
  Path             : kubernetes        ← folder containing deployment.yaml
  Destination      : https://kubernetes.default.svc
  Namespace        : default

→ Click Create
```

---
<img width="955" height="417" alt="argocd-application" src="https://github.com/user-attachments/assets/106aa9f6-5a0e-41be-878c-758124cdb21d" />


## Phase 9 — Create Multi-Branch Pipeline in Jenkins

> A multi-branch pipeline automatically discovers branches in your repository and creates a pipeline for each one that contains a Jenkinsfile.

```
Jenkins Dashboard → New Item
  Name : eMarket
  Type : Multibranch Pipeline

  Branch Sources    : Git
  Project URL       : https://github.com/<your-username>/<your-repo>.git
  Credentials       : github_cred
  Property Strategy : Named branches get different properties (recommended)

  Build Configuration : By Jenkinsfile
  Script Path         : Jenkinsfile     ← must match the filename in repo

→ Save
```

> After a successful build, verify all resources are running:

```bash
kubectl get all -n default
```
<img width="948" height="275" alt="movieapp-kubectl" src="https://github.com/user-attachments/assets/c937fb1c-bb94-474a-a247-a09d417069f6" />

> Copy the **EXTERNAL-IP** of the LoadBalancer service and open it in the browser to see **Version 1** of the application.

---
<img width="956" height="472" alt="Version1-app" src="https://github.com/user-attachments/assets/e121269b-e584-4c4f-b347-9fd8712f7304" />
<img width="955" height="448" alt="Version1-app-part2" src="https://github.com/user-attachments/assets/d3dfcac2-83de-4806-bc19-35bcf1204e37" />


## Phase 10 — Feature Branch Workflow

> This phase covers the GitOps workflow for pushing feature updates. Each feature branch triggers an isolated pipeline build; merging to `main` triggers deployment via ArgoCD.

### Step 10.1 — Create and work on Feature Branch A

```bash
# Create featureA branch from main (inherits all current code)
git checkout -b featureA

# List files
ls

# Update app.py with your new changes (open in your editor)
# ...

# Stage and commit the changes
git status
git add ./app.py
git status
git commit -m "featureA: updated app version"
git push origin featureA
```

### Step 10.2 — Merge featureA to main via Pull Request

```
GitHub → Compare & Pull Request (featureA → main) → Merge Pull Request
```
<img width="959" height="405" alt="merge-PR" src="https://github.com/user-attachments/assets/f204fdc8-ec04-4440-96e3-e2daa07ba5df" />

> After merge:
> - Jenkins detects the change on `main` and triggers the pipeline
> - Docker image is built with a new tag and pushed to DockerHub
> - `kubernetes/deployment.yaml` is updated with the new image tag and committed back to GitHub
> - ArgoCD detects the manifest change and re-creates the pods automatically
> - Open the LoadBalancer external IP to see **Version 2** of the application

<img width="959" height="442" alt="build2" src="https://github.com/user-attachments/assets/ee70a754-d753-4fc8-bf6f-3be078855d38" />
<img width="940" height="442" alt="image-tag-updated" src="https://github.com/user-attachments/assets/c5ce528c-d405-49d2-b00b-1f35789de3b6" />
<img width="949" height="476" alt="version2" src="https://github.com/user-attachments/assets/12c19eb4-e7c8-41db-83be-3e4448937446" />


---

### Step 10.3 — Create and work on Feature Branch B

> After featureA is merged, sync your local main with the latest remote changes before creating featureB.

```bash
# Switch to main and pull latest changes
git checkout main
git pull origin main

# Create featureB branch
git checkout -b featureB

# Update app.py with new changes
# ...

git status
git add ./app.py
git commit -m "featureB: updated app version"
git push origin featureB
```

### Step 10.4 — Merge featureB to main via Pull Request

```
GitHub → Compare & Pull Request (featureB → main) → Merge Pull Request
```
<img width="957" height="458" alt="featureB-PR" src="https://github.com/user-attachments/assets/033342e8-6749-45df-ab7e-91a65fc43191" />

> After merge:
> - Same automated pipeline runs — new image built, manifest updated, ArgoCD syncs
<img width="958" height="472" alt="build-3-sucess" src="https://github.com/user-attachments/assets/33bf14eb-e7b6-4706-9d35-fcb9ec0b2abc" />

> - Open the LoadBalancer external IP to see **Version 3** of the application
<img width="958" height="482" alt="Version3-changes" src="https://github.com/user-attachments/assets/d8a94d95-c991-4079-a884-4a68ef7b4ba6" />
<img width="958" height="485" alt="version3-contd" src="https://github.com/user-attachments/assets/4959dc6e-9122-4d4b-966f-e9a56ec55c1f" />

---

## Phase 11 — Monitoring with Prometheus & Grafana

> Deploy a full Kubernetes monitoring stack using the `kube-prometheus-stack` Helm chart. This single chart installs Prometheus, Grafana, Alertmanager, kube-state-metrics, and Node Exporter all together.

### Step 11.1 — Add the Prometheus community Helm repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Step 11.2 — Install the monitoring stack

```bash
kubectl create namespace monitoring

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring
```

> This installs: **Prometheus** (metrics collection) + **Grafana** (dashboards) + **Alertmanager** (alerts) + **kube-state-metrics** (cluster resource metrics) + **Node Exporter** (node-level metrics).

### Step 11.3 — Verify the installation

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

### Step 11.4 — Expose Grafana via LoadBalancer

```bash
kubectl patch svc monitoring-grafana \
  -n monitoring \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

### Step 11.5 — Get the Grafana LoadBalancer external IP

```bash
kubectl get svc monitoring-grafana -n monitoring
```

> Copy the **EXTERNAL-IP** and open it in your browser on port 80.

### Step 11.6 — Retrieve the Grafana admin password

```bash
kubectl get secret monitoring-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d
```

> Copy the output (stop before `root@...`).
> Log into Grafana with:
> - **Username**: `admin`
> - **Password**: *(output from above command)*

### Step 11.7 — Explore the pre-configured dashboards

> The `kube-prometheus-stack` comes with dashboards already configured. Navigate to:

```
Grafana UI → Dashboards → Browse

Recommended Dashboards:
  - Kubernetes / Nodes
  - Kubernetes / Pods
  - Kubernetes / Deployments
  - Kubernetes / Cluster
  - Node Exporter Full
```
<img width="955" height="482" alt="Grafana-monitoring" src="https://github.com/user-attachments/assets/ed93dad2-f2dd-416e-81e2-ef3568003d9d" />
<img width="959" height="497" alt="Grafana-monitoring2" src="https://github.com/user-attachments/assets/8712c599-7210-4bda-ae2e-db86ac3c2f93" />

---

## ✅ Project Summary

| Phase | Tool | Purpose |
|---|---|---|
| Infrastructure | EC2 (Ubuntu) | Host Jenkins, Docker, CLI tools |
| CI Server | Jenkins | Build, test, and deploy pipeline |
| Containerisation | Docker + DockerHub | Build and store container images |
| Orchestration | AWS EKS | Run containerised workloads |
| GitOps CD | ArgoCD | Sync Kubernetes manifests automatically |
| Package Manager | Helm | Install ArgoCD and monitoring stack |
| Monitoring | Prometheus + Grafana | Metrics collection and dashboards |
| Source Control | GitHub | Code, manifests, Jenkinsfile storage |
