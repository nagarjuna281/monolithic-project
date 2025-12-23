---

# 📘 CI/CD Pipeline with Jenkins, Terraform, Ansible, and AWS S3

## 🚀 Overview
This project demonstrates a **cloud‑native CI/CD pipeline** that provisions AWS infrastructure using **Terraform**, configures EC2 instances with **Ansible**, and automates deployments via **Jenkins**.  
Dynamic inventory is powered by the **AWS EC2 plugin**, allowing Ansible to automatically discover EC2 instances launched from a specific **Launch Template**.  
The pipeline deploys a static website behind an **Elastic Load Balancer (ELB)**, scales with an **Auto Scaling Group (ASG)**, and uses **Amazon S3** for secure storage with ownership controls, ACLs, versioning, and encryption.

---

## 🛠 Prerequisites
- EC2 instance (`t2.medium`) running Amazon Linux 2023 for Jenkins/Ansible control node
- Installed tools:
  - Jenkins
  - Git
  - Terraform
  - Ansible
  - Python3 + pip
  - boto3
- AWS CLI configured with IAM credentials
- SSH key pair (`MY-KP.pem`) stored at `/etc/ansible/`

---

## ⚙️ Infrastructure with Terraform
Terraform provisions:
- **Security Group** → allows SSH (22), HTTP (80), HTTPS (443)
- **Launch Template** → defines EC2 instance configuration
- **Elastic Load Balancer (ELB)** → distributes traffic across instances
- **Auto Scaling Group (ASG)** → manages scaling and health replacement
- **S3 Bucket** → with ownership controls, ACL, versioning, and encryption

Run:
```bash
terraform init
terraform plan
terraform apply --auto-approve
```

---


## 🌐 Access the Website
After pipeline completion:
1. Get ELB DNS name:
   ```bash
   aws elb describe-load-balancers --query "LoadBalancerDescriptions[*].DNSName" --region us-east-1
   ```
2. Open in browser:
   ```
   http://web-server-lb-XXXXXXXX.us-east-1.elb.amazonaws.com
   ```
3. The static site is now live, resilient with ASG auto‑healing, and backed by secure storage in S3.

---

## 🖼️ Architecture Diagram

---

## ✅ Workflow Summary
1. Launch EC2 (`t2.medium`) and install Jenkins, Git, Terraform, Ansible, pip, boto3  
2. Write Terraform files (SG, LT, ELB, ASG, S3)  
3. Configure Ansible (`ansible.cfg`, `aws_ec2.yml`, `deployment.yml`)  
4. Setup Jenkins with Stage View plugin  
5. Create pipeline job with Jenkinsfile  
6. Build pipeline → Terraform provisions infra → Ansible deploys site  
7. Access site via ELB DNS with ASG scaling and S3 storage  
