# 🚀 Amazon EKS Demo  
### Terraform + AWS CloudShell + Kubernetes App

**Repository:** eks-far-2-cel-demo-30-12  
**Region:** us-east-1 (N. Virginia)

---

## 🎯 מטרת התרגיל

הקמה מלאה של **Amazon EKS** באמצעות **Terraform**,  
והרצה של **אפליקציה אמיתית** בתוך Kubernetes.

זהו תרגיל **End-to-End** שמדגים:
- עבודה מ־AWS CloudShell בלבד  
- יצירת IAM User ייעודי  
- הקמת VPC + EKS  
- חיבור IAM ↔ Kubernetes (RBAC)  
- Build של Docker Image  
- Push ל־ECR  
- Deployment + Service ב־Kubernetes  
- גישה חיצונית לאפליקציה

---

## 🧱 ארכיטקטורה (High Level)

GitHub (app code) → Docker Image → Amazon ECR → Amazon EKS → Service (LoadBalancer) → Public URL

---

## ✅ דרישות מקדימות

**נדרש:** חשבון AWS פעיל, הרשאות Admin (לשלב ההקמה), חשבון GitHub  
**לא נדרש:** AWS CLI מקומי, Terraform מקומי, Docker מקומי

---

## 1️⃣ בחירת Region
בחר Region: **us-east-1 (N. Virginia)**

---

## 2️⃣ פתיחת AWS CloudShell
בדיקה:
```
aws sts get-caller-identity
```

---

## 3️⃣ יצירת IAM User ייעודי
User name: `eks-far-2-cel-demo-user`  
Attach policy: `AdministratorAccess`

---

## 4️⃣ התחברות עם ה-IAM החדש
```
aws configure
aws sts get-caller-identity
```

---

## 5️⃣ התקנת Terraform
```
curl -sLo terraform.zip https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip
unzip terraform.zip
sudo mv terraform /usr/local/bin/
rm terraform.zip
terraform -version
```

---

## 6️⃣ טיפול במגבלת דיסק
```
export TF_PLUGIN_CACHE_DIR=/tmp/terraform-plugin-cache
mkdir -p /tmp/terraform-plugin-cache
```

---

## 7️⃣ Clone Repository
```
git clone https://github.com/agorbach/eks-far-2-cel-demo-30-12.git
cd eks-far-2-cel-demo-30-12
```

---

## 8️⃣ Terraform Configuration
קבצים: versions.tf, provider.tf, main.tf (כפי שהוגדרו במדריך)

---

## 9️⃣ יצירת ה-EKS
```
terraform init -upgrade
terraform apply
```

---

## 🔟 חיבור kubectl
```
aws eks update-kubeconfig --region us-east-1 --name eks-far-2-cel-demo-30-12
kubectl get nodes
```

---

## 1️⃣1️⃣ הרצת אפליקציית far-2-cel
כולל Dockerfile, ECR, Deployment, Service

---

## 1️⃣2️⃣ ניקוי משאבים
```
terraform destroy
```

---

## ✅ סיכום
EKS פעיל, אפליקציה רצה, מוכן לדמו בכיתה.

