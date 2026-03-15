# CI/CD Kubernetes Project

This project demonstrates a full **CI/CD pipeline** for a Python Flask application using **Docker**, **GitHub Actions**, and **AWS EKS (Kubernetes)**. The pipeline automatically builds, tests, pushes the Docker image to **AWS ECR**, and deploys it on an **EKS cluster**.

---

## **Table of Contents**
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Local Testing (Minikube)](#local-testing-minikube)
- [AWS Setup](#aws-setup)
- [GitHub Actions Workflow](#github-actions-workflow)
- [Deployment](#deployment)
  

---

## **Project Overview**

This project demonstrates:

1. **CI Pipeline:** Checkout, install dependencies, run tests.
2. **Docker Build & Push:** Build a Docker image and push it to AWS ECR.
3. **Kubernetes Deployment:** Deploy application to AWS EKS cluster.
4. **Automated Workflow:** Fully automated with GitHub Actions.

---

## **Prerequisites**

- AWS account (Free Tier recommended)
- IAM user with:
  - `AmazonEKSClusterPolicy`
  - `AmazonEKSWorkerNodePolicy`
  - `AmazonEC2ContainerRegistryReadOnly`
- GitHub account
- GitHub repository
- Minikube (optional, for local testing)
- Docker installed locally
- kubectl installed

---

## **Project Structure**


CICD-Kubernetes-Project/
├─ app/                        # Flask application code
│  ├─ main.py                  # Main Flask app
│  └─ requirements.txt         # Python dependencies
├─ docker/                     # Docker configuration
│  └─ Dockerfile               # Build Docker image
├─ k8s/                        # Kubernetes manifests
│  ├─ deployment.yaml          # Deployment manifest
│  └─ service.yaml             # Service manifest
└─ .github/
   └─ workflows/
      ├─ ci.yml                # CI workflow: tests & lint
      ├─ build-push.yml        # Build Docker image & push to ECR
      └─ deploy.yml            # Deploy to EKS


---

## **Local Testing (Minikube)**

1. Start Minikube:

```bash
minikube start --driver=docker

Build Docker image locally:

docker build -t devops-flask-app -f docker/Dockerfile .

Deploy to Minikube:

kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

Check pods and services:

kubectl get pods
kubectl get svc

Access the application:

kubectl port-forward svc/flask-service 5000:80

Open http://localhost:5000 in your browser.

AWS Setup
1. Create EKS Cluster

### **1. Create EKS Cluster using Terraform**
Instead of manually creating the cluster, this project uses **Terraform** to provision the AWS infrastructure, including:

- **EKS Cluster**
- **Node Group** (t3.micro for Free Tier)
- **IAM Roles and Policies** for cluster and nodes
- **VPC, Subnets, Security Groups** for Kubernetes networking

This ensures **Infrastructure as Code (IaC)** and makes it reproducible.

**Example Terraform steps:**

1. Initialize Terraform:

```bash
terraform init
Review plan:

terraform plan

Apply configuration:

terraform apply

After Terraform completes, you will have:

EKS cluster ready

Node group deployed

IAM roles and policies attached

VPC & networking configured

2. Configure kubeconfig

Once the cluster is created by Terraform:

aws eks --region eu-north-1 update-kubeconfig --name <cluster_name>
kubectl get nodes

This allows kubectl and GitHub Actions workflows to connect and deploy your application.

option 2 Use the AWS Console:

Cluster name: eks-cluster

Kubernetes version: latest stable

IAM Role: eks-cluster-role

Add Node Group:

Instance type: t3.micro (Free Tier)

Nodes: 1–2

Attach policies: AmazonEKSNodePolicy, AmazonEKSWorkerNodePolicy, AmazonEC2ContainerRegistryReadOnly

2. Configure kubeconfig
aws eks --region eu-north-1 update-kubeconfig --name devops-flask-cluster
kubectl get nodes

3. Push Docker Image to ECR
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin <account_id>.dkr.ecr.eu-north-1.amazonaws.com
docker tag devops-flask-app:latest <account_id>.dkr.ecr.eu-north-1.amazonaws.com/devops-flask-app:latest
docker push <account_id>.dkr.ecr.eu-north-1.amazonaws.com/devops-flask-app:latest
GitHub Actions Workflow
1. CI Workflow (ci.yml)

Checkout code

Install dependencies

Run tests

2. Build & Push Docker Image (build-push.yml)

Build Docker image

Log in to AWS ECR

Push image to ECR

3. Deploy Workflow (deploy.yml)

Update kubeconfig with EKS cluster

Deploy deployment.yaml and service.yaml to EKS

Workflows are chained using workflow_run events:

CI → Build & Push → Deploy

Deployment

After pushing code to GitHub main branch:

GitHub Actions will automatically:

Run tests

Build & push Docker image to ECR

Deploy application to EKS cluster

Verify pods and service on EKS:

kubectl get pods
kubectl get svc

Access the app using the NodePort service or LoadBalancer (if configured).

Conclusion

This project demonstrates a real-world CI/CD pipeline with:

GitHub Actions automation

Docker image management via ECR

Kubernetes deployment on AWS EKS

Full separation of CI, Build, and Deploy workflows

