
# 📘 CI/CD Pipeline with Jenkins, Terraform, Ansible, and AWS S3

## 🚀 Overview
This project demonstrates a **cloud‑native CI/CD pipeline** that provisions AWS infrastructure using **Terraform**, configures EC2 instances with **Ansible**, and automates deployments via **Jenkins**.  
Dynamic inventory is powered by the **AWS EC2 plugin**, allowing Ansible to automatically discover EC2 instances launched from a specific **Launch Template**.  
The pipeline deploys a static website behind an **Elastic Load Balancer (ELB)**, scales with an **Auto Scaling Group (ASG)**, and uses **Amazon S3** for secure storage with ownership controls, ACLs, versioning, and encryption.

---

## 🖥️ 1. Control Node Setup
- You start with a **t2.medium EC2 instance** (Amazon Linux 2023).  
- This acts as your **control node** where Jenkins, Terraform, Ansible, and supporting tools run.  
- Installed tools:
  - **Jenkins** → orchestrates the pipeline.  
  - **Git** → pulls source code from GitHub.  
  - **Terraform** → provisions AWS infrastructure.  
  - **Ansible** → configures EC2 instances and deploys the app.  
  - **pip + boto3** → Python package manager and AWS SDK, required for Ansible’s AWS EC2 dynamic inventory plugin.

---

## ⚙️ 2. Infrastructure with Terraform
Terraform defines and provisions AWS resources:

- **Security Group (SG)**  
  - Allows inbound SSH (22), HTTP (80), HTTPS (443).  
  - Allows all outbound traffic.  
  - Protects EC2 instances by controlling network access.

- **Launch Template (LT)**  
  - Blueprint for EC2 instances (AMI, instance type, key pair, SG).  
  - Ensures all instances launched by ASG have identical configuration.  
  - Supports versioning for updates.

- **Elastic Load Balancer (ELB)**  
  - Distributes traffic across multiple EC2 instances.  
  - Performs health checks and routes only to healthy nodes.  
  - Provides a single DNS endpoint for users.

- **Auto Scaling Group (ASG)**  
  - Manages a fleet of EC2 instances.  
  - Ensures desired capacity (e.g., 2 instances).  
  - Scales up when demand increases, scales down when demand decreases.  
  - Replaces unhealthy instances automatically.

- **S3 Bucket**  
  - Provides secure storage with ownership controls, ACL, versioning, and encryption.  
  - Can be used for logs, artifacts, backups, or static assets.  
  - Ensures compliance and durability.

👉 Running `terraform apply` provisions all these resources in AWS automatically.

---

## 📂 3. Configuration with Ansible
Once infrastructure is up, Ansible configures the EC2 instances:

- **ansible.cfg**  
  - Central config file.  
  - Points to dynamic inventory (`aws_ec2.yml`).  
  - Sets default SSH user (`ec2-user`).  
  - Enables privilege escalation (`sudo → root`).  
  - Disables host key checking for automation.

- **aws_ec2.yml**  
  - Dynamic inventory plugin for AWS.  
  - Queries AWS for EC2 instances that match filters (e.g., Launch Template ID).  
  - Ensures Ansible always targets the correct, running instances.  
  - Groups instances by tags for flexible targeting.

- **deployment.yml playbook**  
  - Installs Apache web server.  
  - Starts and enables Apache service.  
  - Installs Git.  
  - Clones static site code from GitHub into `/var/www/html`.  
  - Ensures every EC2 instance in the ASG is identically configured and ready to serve traffic.

👉 Running `ansible-playbook deployment.yml` applies this configuration across all EC2 instances discovered dynamically.

---

## 🔄 4. Jenkins Pipeline
Jenkins automates the entire workflow with a pipeline defined in a **Jenkinsfile**:

- **Stages**:
  1. **Checkout Code** → pulls repo from GitHub.  
  2. **Terraform Init** → initializes Terraform.  
  3. **Terraform Plan** → previews infrastructure changes.  
  4. **Terraform Apply** → provisions infrastructure.  
  5. **Deploy with Ansible** → configures EC2 instances and deploys the site.

- **Post Actions**:  
  - Success → logs success message.  
  - Failure → logs failure message.

👉 Jenkins ensures every build consistently provisions infra and deploys the app without manual steps.

---

## 🌐 5. Access via ELB
- After pipeline completion, the **Elastic Load Balancer** exposes the application.  
- You retrieve the ELB DNS name:
  ```bash
  aws elb describe-load-balancers --query "LoadBalancerDescriptions[*].DNSName" --region us-east-1
  ```
- Open the DNS name in a browser:
  ```
  http://web-server-lb-XXXXXXXX.us-east-1.elb.amazonaws.com
  ```
- The ELB routes traffic to healthy EC2 instances in the ASG.  
- ASG ensures resilience (replaces failed nodes) and scalability (adds/removes nodes based on demand).  
- S3 provides secure storage for logs, artifacts, or static assets.

---

## 🖼️ 6. Architecture Flow
```
Developer → GitHub Repo
       ↓
   Jenkins Pipeline
       ↓
   Terraform (Infra: SG, LT, ASG, ELB, S3)
       ↓
   AWS Cloud Resources
       ↓
   Ansible (Config + Deploy Site)
       ↓
   EC2 Instances (Apache + Site)
       ↓
   Elastic Load Balancer (DNS)
       ↓
   End User Browser
```

---

## ✅ Summary
- **Terraform** → builds infra (SG, LT, ASG, ELB, S3).  
- **Ansible** → configures EC2 instances and deploys the site.  
- **Jenkins** → automates pipeline from GitHub to deployment.  
- **ELB + ASG** → provide scalability and resilience.  
- **S3** → ensures secure, compliant storage.  

👉 In short: push code → Jenkins triggers pipeline → Terraform provisions infra → Ansible deploys → ELB serves site → ASG scales → S3 stores securely.
