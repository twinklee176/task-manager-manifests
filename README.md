#  Task Manager Kubernetes Manifests

This repository contains the Kubernetes manifest files for deploying the **Task Manager** application on **Amazon EKS** using a **GitOps CI/CD pipeline**.

The repository is monitored by **Argo CD**, which continuously watches this repository and automatically synchronizes any changes to the Kubernetes cluster.

---

# Overview

This project demonstrates a complete GitOps-based Continuous Integration and Continuous Deployment (CI/CD) pipeline using AWS services and Kubernetes.

The application source code is maintained in a separate GitHub repository, while this repository stores only the Kubernetes deployment manifests.

Whenever Jenkins builds and pushes a new Docker image to Amazon ECR, it automatically updates the image tag inside `deployment.yaml` and pushes the changes to this repository. Argo CD detects the new commit and deploys the latest version to the Amazon EKS cluster.

---

#  Architecture

```text
Developer
      │
      ▼
GitHub (Application Repository)
      │
      ▼
Jenkins Pipeline
      │
      ▼
Docker Build
      │
      ▼
Amazon ECR
      │
      ▼
GitHub (Manifest Repository)
      │
      ▼
Argo CD
      │
      ▼
Amazon EKS
      │
      ▼
AWS Application Load Balancer
      │
      ▼
Users
```

---

#  Repository Structure

```text
task-manager-manifests/
│
├── deployment.yaml
├── service.yaml
├── ingress.yaml
└── README.md
```

---

#  Manifest Files

## deployment.yaml

Creates the Kubernetes Deployment.

- Deploys the Task Manager application
- Pulls Docker image from Amazon ECR
- Defines replicas
- Exposes container port 80

---

## service.yaml

Creates a Kubernetes ClusterIP Service.

- Exposes the Deployment internally
- Routes traffic to application Pods

---

## ingress.yaml

Creates an AWS Application Load Balancer.

Features:

- Internet-facing ALB
- HTTPS Listener
- AWS ACM SSL Certificate
- Route53 DNS Integration
- AWS Load Balancer Controller

---

#  AWS Services Used

- Amazon EC2
- Amazon EKS
- Amazon ECR
- AWS IAM
- AWS Load Balancer Controller
- AWS Certificate Manager (ACM)
- Amazon Route53

---

#  Kubernetes Components

- Namespace
- Deployment
- Service
- Ingress
- Pods
- ReplicaSets

---

#  Environment Setup & Commands Used

## Install AWS CLI

```bash
sudo dnf install -y unzip

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws --version
```

---

## Install eksctl

```bash
ARCH=amd64 && PLATFORM=$(uname -s)_$ARCH && curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_${PLATFORM}.tar.gz" && tar -xzf eksctl_${PLATFORM}.tar.gz && sudo mv eksctl /usr/local/bin && eksctl version
```

---

## Configure AWS

```bash
aws configure

aws sts get-caller-identity
```

---

## Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client
```

---

## Create Amazon EKS Cluster

```bash
eksctl create cluster \
--name dev-cluster \
--region ap-south-1 \
--version 1.36 \
--nodes 2 \
--node-type t3.small

kubectl get nodes
```

---

## Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## Configure IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
--cluster dev-cluster \
--approve
```

---

## Create IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.14.1/docs/install/iam_policy.json

aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json
```

---

## Create IAM Service Account

```bash
eksctl create iamserviceaccount \
--cluster=dev-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::796093524638:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region=ap-south-1 \
--approve
```

---

## Install AWS Load Balancer Controller

```bash
helm repo add eks https://aws.github.io/eks-charts

helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=dev-cluster \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller \
--version 1.14.0
```

Verify:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

---

## Install Argo CD

```bash
kubectl create namespace argocd

kubectl apply -n argocd \
--server-side \
--force-conflicts \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Get Argo CD Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}"
```

---

## Create Application Namespace

```bash
kubectl create namespace task-manager
```

---

## Deploy Kubernetes Resources

```bash
kubectl apply -f deployment.yaml

kubectl apply -f service.yaml

kubectl apply -f ingress.yaml
```

---

# GitOps Workflow

```text
GitHub (Application Code)
        │
        ▼
Jenkins Pipeline
        │
        ▼
Docker Build
        │
        ▼
Amazon ECR
        │
        ▼
Update deployment.yaml
        │
        ▼
Push Changes to GitHub
        │
        ▼
Argo CD detects changes
        │
        ▼
Deploy to Amazon EKS
```

---

#  Technologies Used

- Kubernetes
- Docker
- Jenkins
- GitHub
- Git
- Amazon EKS
- Amazon ECR
- Amazon EC2
- AWS IAM
- Route53
- AWS ACM
- Helm
- Argo CD

---
