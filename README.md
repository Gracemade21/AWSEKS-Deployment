# **AWSEKS-Deployment**
This guide provides a step-by-step approach to deploying the Node.js Musician App on Amazon Elastic Kubernetes Service (EKS) from an EC2 Ubuntu instance. Push the application to Amazon ECR and deploy it to EKS.

# **Deploying a Scalable, Highly Available, and Secure Node.js Application on AWS EKS from an EC2 Instance**

## **Introduction**
This guide provides a **step-by-step approach** to deploying the **Node.js Musician App** on **Amazon Elastic Kubernetes Service (EKS) from an EC2 Ubuntu instance**. The deployment ensures **scalability, high availability, fault tolerance, and security**. Additionally, we will configure the required **IAM roles and permissions** to push the application to **Amazon ECR** and deploy it to **EKS**.

---

# **🚀 Part 1: Setting Up the EC2 Instance**

## **Step 1: Launch an Ubuntu EC2 Instance**
1. Go to the **AWS Management Console** → Navigate to **EC2**.
2. Click **Launch Instance**.
3. Select **Ubuntu 22.04 LTS** (or latest version).
4. Choose instance type (**t3.medium** recommended for better performance).
5. Configure Security Group:
   - **Allow SSH (port 22)** for remote access.
   - **Allow HTTP (port 80)** for web access.
   - **Allow Kubernetes traffic (port 6443)** if needed.
6. Attach an **IAM Role** with the required permissions (see next step).
7. Launch and connect using SSH:
```sh
ssh -i <your-key.pem> ubuntu@<EC2-Public-IP>
```

---

## **Step 2: Create IAM Role and Attach to EC2 Instance**
To allow the EC2 instance to push images to **ECR** and interact with **EKS**, create an IAM Role with the following permissions.

### **2.1 Create a Trust Policy for EC2 Role**
Create a JSON file `ec2-trust-policy.json`:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
Run this command to create the IAM role:
```sh
aws iam create-role --role-name EC2EKSRole --assume-role-policy-document file://ec2-trust-policy.json
```

### **2.2 Attach Required Policies to the IAM Role**
```sh
aws iam attach-role-policy --role-name EC2EKSRole --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
aws iam attach-role-policy --role-name EC2EKSRole --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
aws iam attach-role-policy --role-name EC2EKSRole --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam attach-role-policy --role-name EC2EKSRole --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
```

### **2.3 Attach the IAM Role to EC2**
1. Go to **EC2 Dashboard** → **Instances**.
2. Select your **EC2 Instance**.
3. Click **Actions** → **Security** → **Modify IAM Role**.
4. Attach the **EC2EKSRole** IAM Role.

✅ Now, your EC2 instance has the required permissions.

---

# **🚀 Part 2: Uploading and Deploying the Application**

## **Step 3: Upload the Application Files to EC2**
```sh
scp -i <your-key.pem> -r /path/to/local/project ubuntu@<EC2-Public-IP>:/home/ubuntu/musician-app
```
Or clone from GitHub:
```sh
git clone https://github.com/your-repo/musician-app.git
cd musician-app
```

Verify the file structure:
```sh
ls -l /home/ubuntu/musician-app
```
✅ You should see `Dockerfile`, `app.js`, `deployment.yaml`, and `service.yaml`.

---

## **Step 4: Install Required Tools on EC2**
```sh
sudo apt update && sudo apt install -y docker.io unzip nodejs npm awscli
sudo usermod -aG docker $USER && newgrp docker
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
curl -sS https://webinstall.dev/eksctl | bash
source ~/.bashrc
```
Verify installations:
```sh
docker --version
kubectl version --client
eksctl version
aws --version
```

---

## **Step 5: Create an EKS Cluster from EC2**
```sh
eksctl create cluster --name musician-cluster --region us-east-1 --nodes 3 --node-type t3.medium --managed
```
Verify the cluster:
```sh
aws eks list-clusters
kubectl get nodes
```

---

## **Step 6: Push Docker Image to Amazon ECR**
```sh
aws ecr create-repository --repository-name musician-app
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
docker build -t musician-app .
docker tag musician-app:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/musician-app:latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/musician-app:latest
```

---

## **Step 7: Deploy the Application to EKS from EC2**
```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
Verify deployment:
```sh
kubectl get pods
kubectl get svc musician-service
```
✅ Open the **EXTERNAL-IP** in a browser.

---

# **🚀 Summary**
✅ **Configured an IAM Role** with required permissions for EC2 to interact with **ECR & EKS**.
✅ **Uploaded the application** to the EC2 instance.
✅ **Installed AWS CLI, Docker, Kubernetes tools, and Node.js**.
✅ **Created an EKS Cluster** from EC2.
✅ **Pushed the Docker image** to Amazon ECR.
✅ **Deployed the Node.js app** on EKS from EC2.

Your application is now **scalable, highly available, fault-tolerant, and secure**! 🚀 Let me know if you need further refinements. 🎯

