---
layout: post
title: Deploying Amazon EKS on AWS Using Terraform
date: 2025-11-04 17:32:20 +0500
description: Learn how to provision a fully managed Amazon Elastic Kubernetes Service (EKS) cluster on AWS using Terraform, including VPC setup, IAM roles, and worker node configuration.
img: i-image.jpg
fig-caption: Terraform automating EKS deployment on AWS
tags: [AWS, Terraform, EKS, DevOps, Kubernetes]
---

Amazon Elastic Kubernetes Service (EKS) simplifies running Kubernetes on AWS without needing to install or manage your own control plane. Using Terraform, we can automate the entire infrastructure setup for an EKS cluster, ensuring reproducibility and scalability.

---

## 🧱 Prerequisites

Before starting, ensure you have:

- An **AWS account** with sufficient permissions  
- **Terraform** (v1.6.0 or higher) installed  
- **kubectl** CLI tool  
- **AWS CLI** configured (`aws configure`)  

---

## ⚙️ Step 1: Define Provider and Backend

Create a file named `provider.tf`:

```hcl
provider "aws" {
  region = "us-east-1"
}
````

If you’re using remote state storage (recommended), configure an S3 backend for Terraform state:

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state-bucket"
    key    = "eks/terraform.tfstate"
    region = "us-east-1"
  }
}
```

---

## 🌐 Step 2: Create VPC and Networking

EKS requires a properly configured VPC with public and private subnets.

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.2"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    "kubernetes.io/cluster/eks-cluster" = "shared"
  }
}
```

---

## 🛠 Step 3: Create EKS Cluster

```hcl
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  version         = "20.8.4"

  cluster_name    = "eks-cluster"
  cluster_version = "1.30"
  cluster_endpoint_public_access = true

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    default = {
      instance_types = ["t3.medium"]
      desired_size   = 2
      max_size       = 3
      min_size       = 1
    }
  }

  tags = {
    Environment = "Dev"
    Project     = "EKS-Terraform"
  }
}
```

---

## 🚀 Step 4: Initialize and Apply

Run the following commands:

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

After successful provisioning, update your local kubeconfig:

```bash
aws eks update-kubeconfig --name eks-cluster --region us-east-1
```

Verify connection:

```bash
kubectl get nodes
```

---

## 📦 Step 5: Deploy a Sample App

Let’s deploy a simple Nginx application:

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl get svc
```

Once the `EXTERNAL-IP` is assigned, open it in your browser to confirm it’s running.

---

## 🧩 Cleanup

To destroy all resources:

```bash
terraform destroy -auto-approve
```

---

## ✅ Conclusion

With Terraform, deploying Amazon EKS becomes repeatable, scalable, and version-controlled. This approach not only saves time but also enforces infrastructure best practices through code.

> Automate your cloud. Codify your clusters.

```

Would you like me to include **an architecture diagram (in Markdown or image)** showing the EKS setup with Terraform (VPC, nodes, ALB, etc.) for this post?
```
