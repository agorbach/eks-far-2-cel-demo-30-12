# 🚀 Amazon EKS Demo – Full End‑to‑End Guide
Terraform • AWS CloudShell • Kubernetes Application

Repository: eks-far-2-cel-demo-30-12  
Region: us-east-1 (N. Virginia)

---

## 🎯 Goal of This Project

This guide demonstrates a **complete, classroom‑ready workflow** for:

• Creating an Amazon EKS cluster using Terraform  
• Working **only** from AWS CloudShell (no local setup)  
• Creating and using a **dedicated IAM user**  
• Connecting IAM permissions to Kubernetes RBAC  
• Building a Docker image from real application code  
• Pushing the image to Amazon ECR  
• Deploying the application to Kubernetes  
• Exposing it publicly via LoadBalancer  

This is **not** a hello‑world demo — this is a real, end‑to‑end cloud flow.

---

## 🧱 High‑Level Architecture

GitHub (application code)  
→ Docker image  
→ Amazon ECR  
→ Amazon EKS  
→ Kubernetes Deployment  
→ Kubernetes Service (LoadBalancer)  
→ Public URL

---

## ✅ Prerequisites

Required:
• AWS account  
• Administrator permissions (for demo/classroom use)  
• GitHub account  

Not required:
• Local AWS CLI  
• Local Terraform  
• Local Docker  

All actions are performed inside **AWS CloudShell**.

---

## 1️⃣ Select AWS Region

In the AWS Console (top‑right):

Region must be set to:

us‑east‑1 (N. Virginia)

⚠️ All steps in this guide assume **us‑east‑1**.

---

## 2️⃣ Open AWS CloudShell

1. Log in to AWS Console  
2. Click the CloudShell icon (>_)  
3. Verify the region is us‑east‑1  

Check identity:

```bash
aws sts get-caller-identity
```

---

## 3️⃣ Create a Dedicated IAM User (MANDATORY)

❗ Do **not** work with the root user in class.  
We create a clear, dedicated IAM user.

### 3.1 Create the User

AWS Console → IAM → Users → Create user

User name:
```
eks-far-2-cel-demo-user
```

Enable:
• AWS Management Console access  
• Programmatic access  

---

### 3.2 Permissions

Attach policies directly:
• AdministratorAccess

⚠️ This is acceptable for demos and classrooms only.

---

### 3.3 Access Keys

Create an access key and **save securely**:
• Access Key ID  
• Secret Access Key  

---

## 4️⃣ Configure AWS CLI in CloudShell

In CloudShell:

```bash
aws configure
```

Enter:
• Access Key ID  
• Secret Access Key  
• Default region: us‑east‑1  
• Output format: json  

Verify:

```bash
aws sts get-caller-identity
```

Expected format:
```
arn:aws:iam::ACCOUNT_ID:user/eks-far-2-cel-demo-user
```

---

## 5️⃣ Install Terraform in CloudShell

Terraform is NOT preinstalled.

```bash
cd ~
curl -sLo terraform.zip https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip
unzip terraform.zip
sudo mv terraform /usr/local/bin/
rm terraform.zip
terraform -version
```

---

## 6️⃣ CloudShell Disk Space Fix (CRITICAL)

CloudShell storage is limited.

Without this step, Terraform may fail.

```bash
export TF_PLUGIN_CACHE_DIR=/tmp/terraform-plugin-cache
mkdir -p /tmp/terraform-plugin-cache
```

---

## 7️⃣ Clone the Project Repository

```bash
cd ~
git clone https://github.com/agorbach/eks-far-2-cel-demo-30-12.git
cd eks-far-2-cel-demo-30-12
```

Expected structure:
```
infra/
k8s/
app/
```

---

## 8️⃣ Terraform Configuration Files

### infra/versions.tf

```hcl
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.30"
    }
  }
}
```

⚠️ IMPORTANT:
AWS provider **6.x is NOT compatible** with the EKS module used here.

---

### infra/provider.tf

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

### infra/main.tf  (FULL FILE)

⚠️ IMPORTANT:
You **MUST** replace `ACCOUNT_ID` with your real AWS account ID.

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.2"

  name = "eks-far-2-cel-demo-30-12-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.24.3"

  cluster_name    = "eks-far-2-cel-demo-30-12"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  access_entries = {
    admin = {
      principal_arn = "arn:aws:iam::ACCOUNT_ID:user/eks-far-2-cel-demo-user"

      policy_associations = {
        admin = {
          policy_arn = "arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy"
          access_scope = {
            type = "cluster"
          }
        }
      }
    }
  }

  eks_managed_node_groups = {
    default = {
      instance_types = ["t3.medium"]
      desired_size   = 2
      min_size       = 1
      max_size       = 3
    }
  }
}
```

---

## 9️⃣ Create the EKS Cluster

```bash
cd infra
rm -rf .terraform .terraform.lock.hcl
terraform init -upgrade
terraform apply
```

Confirm with:
```
yes
```

⏱️ Provisioning time: ~10–15 minutes

---

## 🔟 Connect kubectl to the Cluster

```bash
aws eks update-kubeconfig   --region us-east-1   --name eks-far-2-cel-demo-30-12
```

Verify:
```bash
kubectl get nodes
```

---

## 1️⃣1️⃣ Deploy the far‑2‑cel Application

### 11.1 Clone Application Source

```bash
cd ~/eks-far-2-cel-demo-30-12
mkdir -p app
cd app
git clone https://github.com/agorbach/test2025.git
cd test2025/far-2-cel
```

---

### 11.2 Dockerfile

If missing, create:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

---

### 11.3 Create ECR Repository

```bash
aws ecr create-repository   --repository-name far-2-cel   --region us-east-1
```

---

### 11.4 Login to ECR

⚠️ Replace ACCOUNT_ID below.

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
```

---

### 11.5 Build & Push Image

```bash
docker build -t far-2-cel:1.0 .
docker tag far-2-cel:1.0 ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/far-2-cel:1.0
docker push ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/far-2-cel:1.0
```

---

### 11.6 Kubernetes Deployment & Service

deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: far-2-cel
spec:
  replicas: 2
  selector:
    matchLabels:
      app: far-2-cel
  template:
    metadata:
      labels:
        app: far-2-cel
    spec:
      containers:
        - name: far-2-cel
          image: ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/far-2-cel:1.0
          ports:
            - containerPort: 5000
```

service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: far-2-cel
spec:
  type: LoadBalancer
  selector:
    app: far-2-cel
  ports:
    - port: 80
      targetPort: 5000
```

Apply:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get svc far-2-cel
```

Open in browser:
```
http://<EXTERNAL-IP>
```

---

## 1️⃣2️⃣ Clean Up Resources

```bash
cd infra
terraform destroy
```

---

## ✅ Final Result

✔ EKS cluster created  
✔ IAM correctly connected to Kubernetes  
✔ Real application running inside Kubernetes  
✔ Public access via LoadBalancer  
✔ Fully classroom‑ready

---

End of guide.
