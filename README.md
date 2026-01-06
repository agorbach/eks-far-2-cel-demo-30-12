# 🚀 הדגמת Amazon EKS  
### Terraform • AWS CloudShell • Kubernetes

**Region:** us-east-1 (N. Virginia)  
**שם ה-Repository:** eks-far-2-cel-demo-30-12  

---

## 🎯 מטרת התרגיל

מסמך זה מציג תהליך מלא, מסודר וברור להקמת **Amazon EKS** באמצעות **Terraform**,  
והרצה של **אפליקציה אמיתית (far-2-cel)** בתוך Kubernetes – משלב אפס ועד אפליקציה זמינה בדפדפן.

התרגיל מיועד לדמו בכיתה / קורס DevOps, ונבנה כך שניתן לבצע אותו **מהתחלה ועד הסוף**  
כולל תרגול מעשי של כשעתיים.

---

## 🧱 ארכיטקטורה כללית (High Level)

GitHub (קוד האפליקציה)  
→ Docker Image  
→ Amazon ECR  
→ Amazon EKS  
→ Deployment  
→ Service (LoadBalancer)  
→ כתובת ציבורית בדפדפן  

---

## ✅ דרישות מקדימות

### נדרש
- חשבון AWS פעיל  
- הרשאות Administrator (לצורכי קורס / דמו)  
- חשבון GitHub  

### לא נדרש
- התקנות מקומיות  
- Docker מקומי  
- Terraform מקומי  

> 💡 כל העבודה מתבצעת בענן – באמצעות **AWS CloudShell בלבד**.

---

## 1️⃣ כניסה ל-AWS Console ובחירת Region

1. פתח דפדפן  
2. היכנס לכתובת: https://console.aws.amazon.com  
3. התחבר לחשבון ה-AWS שלך  
4. בפינה הימנית העליונה בחר Region:  

**N. Virginia (us-east-1)**

---

## 2️⃣ פתיחת AWS CloudShell

1. מתוך ה-Console לחץ על אייקון **CloudShell (>_)**  
2. ודא שאתה ב-Region `us-east-1`

בדיקה:
```bash
aws sts get-caller-identity
```

---

## 3️⃣ יצירת IAM User ייעודי (שלב חובה)

❗ לא עובדים עם root בכיתה.

### 3.1 יצירת המשתמש
IAM → Users → Create user  

**שם:**
```
eks-far-2-cel-demo-user
```

סמן:
- AWS Management Console access  
- Programmatic access  

---

### 3.2 הרשאות למשתמש

יש לצרף את ההרשאות הבאות (Attach policies directly):

- AdministratorAccess  
- AdministratorAccess-Amplify  
- AdministratorAccess-AWSElasticBeanstalk  
- AWSAuditManagerAdministratorAccess  
- AWSManagementConsoleAdministratorAccess  
- IAMUserChangePassword  

> ⚠️ הרשאות אלו מיועדות **לקורס בלבד**.

---

### 3.3 יצירת Access Keys

בסיום יצירת המשתמש:
- שמור Access Key ID  
- שמור Secret Access Key  

---

## 4️⃣ הגדרת AWS CLI ב-CloudShell

```bash
aws configure
```

הכנס:
- Access Key  
- Secret Key  
- Region: us-east-1  
- Output format: json  

בדיקה:
```bash
aws sts get-caller-identity
```

---

## 5️⃣ התקנת Terraform ב-CloudShell

```bash
cd ~
curl -sLo terraform.zip https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip
unzip terraform.zip
sudo mv terraform /usr/local/bin/
terraform -version
```

---

## 6️⃣ טיפול במגבלת דיסק של CloudShell

```bash
export TF_PLUGIN_CACHE_DIR=/tmp/terraform-plugin-cache
mkdir -p /tmp/terraform-plugin-cache
```

---

## 7️⃣ Clone של ה-Repository

```bash
git clone https://github.com/agorbach/eks-far-2-cel-demo-30-12.git
cd eks-far-2-cel-demo-30-12
```

---

## 8️⃣ יצירת EKS באמצעות Terraform

```bash
cd infra
terraform init -upgrade
terraform apply
```

אשר עם:
```
yes
```

⏱️ זמן הקמה: כ-10–15 דקות.

---

## 9️⃣ חיבור kubectl ל-EKS

```bash
aws eks update-kubeconfig --region us-east-1 --name eks-far-2-cel-demo-30-12
kubectl get nodes
```

---

## 🔟 בניית והרצת אפליקציית far-2-cel

### 10.1 הורדת קוד האפליקציה

```bash
cd ~
git clone https://github.com/agorbach/test2025.git
cd test2025/far-2-cel
```

---

### 10.2 Dockerfile (בדוק ועובד)

```dockerfile
FROM python:3.7
RUN mkdir /app
WORKDIR /app
ADD . /app/
RUN pip install Flask
EXPOSE 8080
CMD ["python", "/app/main.py"]
```

---

### 10.3 Build ו-Push ל-ECR

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
aws ecr create-repository --repository-name far-2-cel
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com

docker build -t far-2-cel:1.0 .
docker tag far-2-cel:1.0 $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/far-2-cel:1.0
docker push $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/far-2-cel:1.0
```

---

## 1️⃣1️⃣ Deployment ו-Service ב-Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get svc far-2-cel
```

פתח בדפדפן:
```
http://<EXTERNAL-IP>
```

---

## 1️⃣2️⃣ תרגול מעשי (שעת תרגול)

### תרגיל 1 – Pods ו-Nodes
```bash
kubectl get pods -o wide
kubectl get nodes -o wide
```

### תרגיל 2 – מחיקת Pod
```bash
kubectl delete pod <POD_NAME>
```

### תרגיל 3 – Scale
```bash
kubectl scale deployment far-2-cel --replicas=5
```

### תרגיל 4 – מחיקת Service
```bash
kubectl delete svc far-2-cel
```

### תרגיל 5 – החזרת Service
```bash
kubectl apply -f k8s/service.yaml
```

---

## 1️⃣3️⃣ ניקוי משאבים

```bash
cd infra
terraform destroy
```

---

## ✅ סיכום

- הקמנו EKS מלא  
- חיברנו IAM ל-Kubernetes  
- הרצנו אפליקציה אמיתית  
- תרגלנו Nodes, Pods ו-LoadBalancer  

📌 מסמך זה נועד לשמש כתרגיל מלא לכיתה / קורס DevOps.
