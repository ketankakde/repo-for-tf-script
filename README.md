# 🚀 EKS Cluster — Terraform + Jenkins CI/CD

<p align="center">
  <img src="https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS_EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" />
  <img src="https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white" />
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" />
  <img src="https://img.shields.io/badge/IAM-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" />
</p>

> Provision a production-ready **Amazon EKS cluster** on AWS using **Terraform**, automated through a **Jenkins CI/CD pipeline** with a manual approval gate before any infrastructure changes are applied.

---

## 📋 Table of Contents

- [Architecture Overview](#-architecture-overview)
- [What Gets Provisioned](#-what-gets-provisioned)
- [Prerequisites](#-prerequisites)
- [Project Structure](#-project-structure)
- [Configuration](#️-configuration)
- [Jenkins Pipeline Stages](#-jenkins-pipeline-stages)
- [Getting Started](#-getting-started)
- [Security Notes](#-security-notes)

---

## 🏗️ Architecture Overview

```
                        ┌──────────────────────────────────────────────────────────────┐
                        │                      AWS  ( us-east-1 )                      │
                        │                                                              │
  ┌───────────────┐     │   ┌──────────────────────────────────────────────────────┐   │
  │    Jenkins    │     │   │                  VPC: default-vpc                    │   │
  │  CI/CD Server │     │   │                                                      │   │
  └──────┬────────┘     │   │   ┌──────────────────────────────────────────────┐   │   │
         │ terraform    │   │   │            EKS Cluster: my-eks               │   │   │
         │ init/plan/   │   │   │                                              │   │   │
         │ apply        │   │   │   Add-ons:  vpc-cni │ kube-proxy │ coredns   │   │   │
         │              │   │   │                                              │   │   │
         ▼              │   │   │   ┌──────────────────────────────────────┐   │   │   │
  ┌──────────────┐      │   │   │   │     Node Group: my-eks-node-group    │   │   │   │
  │  S3  Backend │      │   │   │   │                                      │   │   │   │
  │  (tfstate)   │      │   │   │   │     min: 1  │ desired: 1 │ max: 2    │   │   │   │
  └──────────────┘      │   │   │   └──────────────────────────────────────┘   │   │   │
                        │   │   │                                              │   │   │
                        │   │   │   IAM Roles:  eks_cluster_role               │   │   │
                        │   │   │               eks_node_role                  │   │   │
                        │   │   └──────────────────────────────────────────────┘   │   │
                        │   │                                                      │   │
                        │   └──────────────────────────────────────────────────────┘   │
                        │                                                              │
                        └──────────────────────────────────────────────────────────────┘
```

---

## ✅ What Gets Provisioned

| Resource | Details |
|---|---|
| 🔐 **IAM Role** — Cluster | `eks_cluster_role` with `AmazonEKSClusterPolicy` |
| 🔐 **IAM Role** — Nodes | `eks_node_role` with Worker, CNI & ECR policies |
| ☸️ **EKS Cluster** | `my-eks` in `us-east-1` using existing VPC |
| 🔌 **Add-on: vpc-cni** | `v1.20.4-eksbuild.2` — pod networking |
| 🔌 **Add-on: kube-proxy** | `v1.30.9-eksbuild.2` — network rules |
| 🔌 **Add-on: coredns** | `v1.11.4-eksbuild.2` — in-cluster DNS |
| 🖥️ **Node Group** | `my-eks-node-group`, configurable instance type, scales 1–2 |
| 🪣 **S3 Backend** | Remote state in a user-defined S3 bucket |

---

## 🔧 Prerequisites

Before running this project, make sure you have:

- ✅ [Terraform](https://developer.hashicorp.com/terraform/install) `>= 1.3`
- ✅ [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) configured with appropriate permissions
- ✅ [Jenkins](https://www.jenkins.io/doc/book/installing/) with the following plugins:
  - Pipeline
  - Git
- ✅ An existing AWS VPC tagged `Name = "default-vpc"` with subnets
- ✅ An S3 bucket of your choice for Terraform remote state
- ✅ AWS IAM permissions to create: EKS clusters, IAM roles, node groups

---

## 📁 Project Structure

```
eks-terraform-jenkins/
├── main.tf          # All Terraform resources (EKS, IAM, add-ons, node group)
├── jenkinsfile      # Jenkins CI/CD pipeline definition
└── README.md        # You are here
```

---

## ⚙️ Configuration

The key values you may want to change before deploying:

| Parameter | Location | Default | Description |
|---|---|---|---|
| `region` | `main.tf` | `us-east-1` | AWS region to deploy into |
| `bucket` | `main.tf` | — | Set your S3 bucket name for Terraform state |
| Cluster name | `main.tf` | `my-eks` | EKS cluster name |
| Instance type | `main.tf` | — | Set your preferred EC2 instance type for the node group |
| VPC tag | `main.tf` | `default-vpc` | Tag name of the existing VPC |
| Git repo URL | `jenkinsfile` | `<your-repo>` | **Update this to your GitHub repo** |

---

## 🔄 Jenkins Pipeline Stages

```
Checkout ──► Terraform Init ──► Terraform Plan ──► 🔔 Approve ──► Terraform Apply ──► Deploy
```

| Stage | Description |
|---|---|
| 🔁 **Checkout** | Clones the repo from GitHub (`main` branch) |
| 🔧 **Terraform Init** | Initialises Terraform and configures the S3 backend |
| 📋 **Terraform Plan** | Generates and saves an execution plan (`tfplan`) |
| 🔔 **Approve** | **Manual gate** — a human must approve before any changes apply |
| 🚀 **Terraform Apply** | Applies the saved plan to provision infrastructure |
| ✅ **Deploy** | Confirms successful deployment |

---

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 2. Update the Jenkinsfile

Edit `jenkinsfile` and replace the placeholder repo URL:

```groovy
git branch: 'main', url: 'https://github.com/<your-username>/<your-repo>.git'
```

### 3. Ensure the S3 bucket exists

```bash
aws s3 create-bucket --bucket <your-bucket-name> --region us-east-1
```

### 4. Create a Jenkins Pipeline job

1. Open Jenkins → **New Item** → **Pipeline**
2. Under **Pipeline**, set **Definition** to `Pipeline script from SCM`
3. Set **SCM** to `Git` and point it to this repo
4. Set **Script Path** to `jenkinsfile`
5. Save and click **Build Now**

### 5. Approve the plan

When the pipeline reaches the **Approve** stage, review the Terraform plan output in Jenkins and click **Approve** to proceed.

---

## 🔒 Security Notes

- 🛡️ **Never commit AWS credentials** to this repo. Use Jenkins credentials or IAM instance profiles.
- 🔑 The S3 backend bucket should have **versioning enabled** and **server-side encryption** (`SSE-S3` or `SSE-KMS`).
- 🔐 Consider enabling **S3 state locking** via a DynamoDB table to prevent concurrent state modifications:
  ```hcl
  backend "s3" {
    bucket         = "your-bucket-name"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
  ```
- 📦 Node groups use `AmazonEC2ContainerRegistryReadOnly` — nodes can pull from ECR but cannot push.

---

