# Tech Challenge1

## 🚀 Overview  
This repository contains a full-stack project with three main modules:  
1. **Frontend** — a web application (React)  
2. **Backend** — a service (Node.js)  
3. **Infrastructure** — Terraform configurations to provision AWS resources (ECS, ALB, networking, etc.)

Each application is Dockerized and is built/deployed via Jenkins pipelines. The resulting containers run as ECS tasks behind an Application Load Balancer (ALB) in AWS.

---

## 🧱 Architecture  
- The **frontend** and **backend** are built as two separate Docker containers.  
- Both containers are pushed to a container registry and then deployed into ECS clusters.  
- An ALB handles incoming HTTP requests and routes them appropriately (e.g., to frontend, or to backend via API).  
- Infrastructure (VPC, subnets, ECS cluster, ALB, IAM roles, etc.) is defined via Terraform so it is reproducible and version-controlled.  
- Jenkins pipelines orchestrate the build → test → dockerize → push → deploy flow.

---

## 📂 Repository Structure  
/
├── frontend/ # Front-end application source
│ ├── Dockerfile
│ ├── src/
│ ├── package.json
│ └── …
├── backend/ # Back-end service source
│ ├── Dockerfile
│ ├── src/
│ ├── pom.xml (or package.json / requirements.txt)
│ └── …
└── terraform/ # Infrastructure as Code
├── modules/ # Reusable Terraform modules (VPC, ECS, ALB, etc.)
├── envs/ # Environment-specific configs (dev/prod)
├── main.tf
├── variables.tf
└── outputs.tf

---

## 🛠️ Prerequisites  
Before you get started, ensure you have the following installed and configured:  
- Docker  
- AWS CLI, with credentials configured (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION)  
- Terraform (compatible version)  
- Jenkins configured with access to this repo and build agents (if working locally, you can skip Jenkins)  
- Container registry access (ECR, Docker Hub, or equivalent)  

---

## 🔧 Setup & Usage  

### 1. Local development  
**Frontend:**  
```bash
cd frontend
npm install
npm start

Access the UI at http://localhost:<PORT>

**Backend:** 
cd backend
# install deps (npm install / mvn clean install / pip install)
npm install
npm run start

API runs at http://localhost:<API_PORT>; ensure it’s configured to allow CORS (if your frontend calls it).

### 2. Build & Dockerize  
From project root:
cd frontend
docker build -t <your-registry>/frontend:<tag> .
docker push <your-registry>/frontend:<tag>

cd ../backend
docker build -t <your-registry>/backend:<tag> .
docker push <your-registry>/backend:<tag>

### 3. Deploy Infrastructure (Terraform)
cd terraform
terraform init
terraform plan -var-file="envs/dev.tfvars"
terraform apply -var-file="envs/dev.tfvars" -auto-approve

This will create the VPC, ECS cluster, ALB, and other resources.

### 4. Jenkins Pipeline
Jenkins monitors the GitHub repository for changes (push or PR).

On change, Jenkins triggers pipeline: build → test → dockerize → push image → update ECS service (via Terraform or ECS API).

After deployment, the application becomes available behind the ALB URL (output by Terraform).

🎯 Key Features
Micro-services split: Separate frontend and backend containers.
Infrastructure as Code: All AWS infra is versioned using Terraform.
CI/CD: Automated builds & deployments using Jenkins.
Docker + ECS: Containers are deployed as ECS tasks, enabling scalability.
Easy to extend: You can add more services/modules by following the existing pattern.

🧩 Environment Variables
Here are some typical environment variables you may need to configure:

| Variable               | Usage                                                       |
| ---------------------- | ----------------------------------------------------------- |
| `AWS_REGION`           | AWS region (e.g., us-west-2)                                |
| `ECR_REGISTRY`         | Container registry URI                                      |
| `FRONTEND_IMAGE_TAG`   | Docker tag for frontend image                               |
| `BACKEND_IMAGE_TAG`    | Docker tag for backend image                                |
| `ALB_DNS_NAME`         | DNS name of the Application Load Balancer (after Terraform) |
| `DB_CONNECTION_STRING` | (If applicable) Connection string for database              |

✅ Workflow Summary

Developer pushes code to GitHub → triggers Jenkins job
Jenkins builds, tests, dockerizes, and pushes images
Terraform (or an ECS update step) deploys/redeploys ECS services with new images
ALB routes traffic to updated containers → new version goes live

📌 Next Steps / Enhancements

Add automated tests (unit/integration) for backend and frontend apps.
Improve blue/green or canary deployment strategy for ECS tasks.
Introduce monitoring & alerting (CloudWatch, SNS, Prometheus).
Secure secrets with AWS Secrets Manager or Parameter Store.
Configure multi-environment (dev, stage, prod) via Terraform workspaces or separate tfvars.

📝 License
MIT License
Free to use, modify, and distribute.

---
