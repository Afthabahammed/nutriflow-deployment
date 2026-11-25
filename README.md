# NutriFlow – Full-Stack Application with DevOps Automation

A full-stack nutrition and food-related application combined with a practical DevOps implementation. This project brings together application development, containerization, CI automation, cloud infrastructure and monitoring.

The main goal is to understand how an application moves from source code to a container image and towards deployment on AWS.

**Repository:** https://github.com/Afthabahammed/nutriflow-deployment

---

## 1. Project Overview

NutriFlow is built with a React frontend, Node.js backend and MongoDB database.

Alongside the application, this repository contains DevOps configurations for automating code checks, building Docker images, publishing images to Amazon Elastic Container Registry (ECR), and preparing AWS infrastructure using Terraform and Kubernetes manifests.

The project explores the following workflow:

```text
Developer
    |
    v
  GitHub
    |
    v
  Jenkins
    |
    v
Frontend & Backend Checks
    |
    v
  SonarQube
    |
    v
  Docker Build
    |
    v
  Amazon ECR
    |
    v
Amazon EKS / Kubernetes
    |
    v
Application Deployment
```

Monitoring with Prometheus and Grafana was also implemented in the earlier EC2-based deployment.

---

## 2. Technology Stack

| Category | Technologies |
|---|---|
| Frontend | React, Vite, Tailwind CSS |
| Backend | Node.js, Express.js |
| Database | MongoDB |
| Containerization | Docker |
| CI/CD | Jenkins |
| Code Quality | SonarQube |
| Container Registry | Amazon ECR |
| Cloud Platform | AWS |
| Infrastructure as Code | Terraform |
| Container Orchestration | Kubernetes, Amazon EKS |
| Monitoring | Prometheus, Grafana |
| Metrics Collection | Node Exporter |
| Version Control | Git, GitHub |
| Operating System | Ubuntu Linux |

---

## 3. Application Architecture

NutriFlow follows a frontend-backend-database architecture.

### Frontend

The frontend is developed using React and Vite, with Tailwind CSS for styling.

Responsibilities include:

- Providing the user interface.
- Handling user interactions.
- Communicating with backend APIs.
- Displaying application data.

The frontend is served using Nginx in its containerized deployment.

### Backend

The backend is developed using Node.js and Express.js.

Responsibilities include:

- Handling API requests.
- Processing application logic.
- Communicating with MongoDB.
- Managing backend operations.

The backend listens on port `8000`.

### Database

MongoDB is used to store application data.

Mongoose is used for interacting with MongoDB from the Node.js backend.

### Application flow

```text
User
 |
 v
React Frontend
 |
 | HTTP API Requests
 v
Node.js / Express Backend
 |
 | Database Operations
 v
MongoDB
```

---

## 4. Repository Structure

```text
nutriflow-deployment/
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   └── Application source files
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   └── React application source
│
├── infrastructure/
│   └── terraform-eks/
│       └── Terraform infrastructure files
│
├── k8s/
│   └── Kubernetes configuration files
│
├── Jenkinsfile
├── .gitignore
├── README.md
└── test.md
```

### Folder explanation

**backend/**

Contains the Node.js and Express.js application, its dependencies and Dockerfile.

**frontend/**

Contains the React application, frontend dependencies and Dockerfile.

**infrastructure/terraform-eks/**

Contains the Terraform configuration for preparing AWS infrastructure associated with Amazon EKS.

**k8s/**

Contains Kubernetes configuration files used to define application workloads and deployment resources.

**Jenkinsfile**

Defines the Jenkins CI pipeline, including source checkout, application checks, SonarQube analysis, Docker image building and publishing images to Amazon ECR.

---

## 5. Docker Containerization

Docker is used to package the frontend and backend applications into portable container images.

Each application has its own Dockerfile.

### Backend image

From the repository root:

```bash
docker build -t nutriflow-backend:v1 ./backend
```

Run the backend container:

```bash
docker run -d \
  --name nutriflow-backend \
  -p 8000:8000 \
  nutriflow-backend:v1
```

### Frontend image

```bash
docker build -t nutriflow-frontend:v1 ./frontend
```

Run the frontend container:

```bash
docker run -d \
  --name nutriflow-frontend \
  -p 8080:8080 \
  nutriflow-frontend:v1
```

### Verify containers

```bash
docker ps
```

View container logs:

```bash
docker logs nutriflow-backend
docker logs nutriflow-frontend
```

**Note:** The frontend and backend need to be configured with the appropriate API routing and runtime environment for the deployment environment.

---

## 6. Jenkins CI Pipeline

Jenkins is used to automate the continuous integration process.

The pipeline is defined in the root-level `Jenkinsfile`.

### Pipeline stages

**Stage 1 – Checkout**

Jenkins retrieves the latest source code from the configured Git repository.

It also checks that the frontend and backend package files and Dockerfiles exist.

**Stage 2 – Frontend Checks**

Jenkins installs frontend dependencies and builds the React application.

```bash
npm ci
npm run build
```

**Stage 3 – Backend Checks**

Jenkins installs backend dependencies and performs a Node.js syntax check.

```bash
npm ci
node --check index.js
```

**Stage 4 – SonarQube Analysis**

SonarQube scans the frontend and backend source code to identify code-quality issues and potential problems.

The Jenkins pipeline uses the configured SonarQube server and scanner installation.

**Stage 5 – Build and Push Docker Images**

Jenkins builds separate Docker images for the frontend and backend.

The images are tagged using the Jenkins build number and the `latest` tag.

**Stage 6 – Amazon ECR Push**

Jenkins authenticates with Amazon ECR using the AWS CLI and pushes the images to their respective repositories.

The ECR repositories used in the pipeline are:

- `nutriflow-frontend`
- `nutriflow-backend`

Images are published with build-specific and `latest` tags.

### Current CI workflow

```text
GitHub
   |
   v
Jenkins Checkout
   |
   v
Frontend Build Check
   |
   v
Backend Syntax Check
   |
   v
SonarQube Analysis
   |
   v
Docker Image Build
   |
   v
Amazon ECR Authentication
   |
   v
Push Frontend Image
   |
   v
Push Backend Image
```

The current Jenkinsfile implements the CI and image-publishing stages. Kubernetes deployment through EKS can be added as a subsequent CD stage.

---

## 7. Amazon ECR – Container Image Storage

Amazon Elastic Container Registry (ECR) is used to store the Docker images created by Jenkins.

ECR provides a private image registry integrated with AWS services.

The project uses separate repositories for the frontend and backend.

Example image naming:

```text
<account-id>.dkr.ecr.<region>.amazonaws.com/nutriflow-frontend:<tag>

<account-id>.dkr.ecr.<region>.amazonaws.com/nutriflow-backend:<tag>
```

The Jenkins pipeline authenticates using:

```bash
aws ecr get-login-password --region ap-south-1
```

The images are then pushed to the corresponding ECR repositories.

---

## 8. Infrastructure as Code – Terraform

Terraform is used to define AWS infrastructure through configuration files.

The project contains an infrastructure directory for the EKS-based setup:

```text
infrastructure/
└── terraform-eks/
```

Terraform helps make infrastructure:

- Repeatable.
- Version-controlled.
- Easier to review.
- Easier to manage.
- Less dependent on manual console operations.

Typical Terraform workflow:

```bash
terraform init

terraform fmt

terraform validate

terraform plan

terraform apply
```

Always review the Terraform plan before applying infrastructure changes.

**Cost consideration:** Amazon EKS has a control-plane charge. Worker nodes, EBS volumes, load balancers and networking resources can add further costs. Remove unused resources carefully when the environment is no longer needed.

---

## 9. Kubernetes and Amazon EKS

Amazon EKS is AWS's managed Kubernetes service.

Kubernetes is responsible for managing containerized application workloads.

The repository contains a `k8s/` directory for Kubernetes configuration.

Kubernetes provides features such as:

- Container scheduling.
- Application deployment management.
- Service discovery.
- Load balancing.
- Scaling.
- Self-healing and restart management.

### Basic Kubernetes workflow

```text
Docker Images
      |
      v
   Amazon ECR
      |
      v
   Amazon EKS
      |
      v
  Kubernetes Pods
      |
      v
Frontend and Backend
```

The EKS deployment stage can be connected to Jenkins after the infrastructure and Kubernetes deployment configurations are ready.

---

## 10. Monitoring and Observability

Monitoring was implemented in the earlier EC2-based deployment.

The monitoring stack included:

### Prometheus

Prometheus collects and stores time-series metrics.

It was configured to collect metrics from:

- Prometheus itself.
- Node Exporter.

### Node Exporter

Node Exporter collects system-level metrics from the Linux server, such as CPU, memory and filesystem information.

### Grafana

Grafana provides dashboards for visualizing metrics collected by Prometheus.

### Monitoring architecture

```text
Linux EC2 Instance
       |
       v
  Node Exporter
       |
       v
   Prometheus
       |
       v
    Grafana
       |
       v
 Monitoring Dashboards
```

The earlier EC2 monitoring setup was validated by checking that the Prometheus scrape targets were UP and that the Grafana interface was accessible.

The monitoring configuration for the previous EC2 deployment should not be confused with a completed monitoring deployment on the new EKS infrastructure.

---

## 11. Environment Variables and Security

The application requires environment-specific configuration.

Examples include:

```env
PORT=8000
MONGO_URI=<your-mongodb-connection-string>
JWT_SECRET=<your-secret>
GEMINI_API_KEY=<your-api-key>
GEMINI_MODEL=<your-model-name>
```

Other integrations may require additional configuration variables.

### Security practices

- Do not commit `.env` files.
- Never publish API keys, passwords or access tokens.
- Use Jenkins credentials for CI/CD secrets.
- Prefer IAM roles over hardcoded AWS access keys.
- Restrict SSH access to trusted IP addresses.
- Avoid exposing internal services such as MongoDB, Jenkins and SonarQube publicly.
- Use least-privilege IAM permissions.
- Protect Terraform state files because they may contain sensitive values.

---

## 12. Prerequisites

To work with this project, you should have:

- Git
- Node.js and npm
- Docker
- AWS CLI
- Terraform
- kubectl
- An AWS account
- A GitHub account
- Jenkins
- SonarQube

Some tools are needed only for the corresponding development or infrastructure stages.

---

## 13. Getting Started

Clone the repository:

```bash
git clone https://github.com/Afthabahammed/nutriflow-deployment.git
```

Navigate into the project:

```bash
cd nutriflow-deployment
```

Check the repository structure:

```bash
ls -la
```

### Frontend setup

```bash
cd frontend

npm ci

npm run build
```

### Backend setup

Open another terminal or return to the repository root:

```bash
cd backend

npm ci

node --check index.js
```

Configure the required environment variables before starting the application.

Refer to the individual application configuration and Dockerfiles for the appropriate runtime commands.

---

## 14. Skills and Learning Outcomes

This project provided practical experience in:

- Full-stack application deployment.
- Linux server administration.
- AWS EC2 management.
- Docker image creation and container management.
- Jenkins pipeline development.
- SonarQube code-quality analysis.
- Amazon ECR image management.
- Terraform infrastructure management.
- Kubernetes and Amazon EKS concepts.
- Prometheus metrics collection.
- Grafana monitoring dashboards.
- Git and GitHub version control.
- Environment configuration and secret management.
- Troubleshooting application and infrastructure issues.

---

## 15. Future Enhancements

The project can be extended with:

- Complete automated deployment from Jenkins to Amazon EKS.
- Kubernetes rollout verification and rollback.
- Argo CD for GitOps-based continuous delivery.
- Prometheus and Grafana monitoring for EKS workloads.
- Centralized log management using the Elastic Stack.
- Trivy container image and dependency vulnerability scanning.
- Improved IAM and Kubernetes RBAC configuration.
- Automated infrastructure provisioning and cleanup.

---

## 16. Conclusion

NutriFlow brings application development and DevOps practices together in one project.

It demonstrates how a full-stack application can be containerized, checked through an automated Jenkins pipeline, analyzed using SonarQube, and packaged for storage in Amazon ECR.

It also provides a foundation for infrastructure automation with Terraform and application orchestration with Kubernetes and Amazon EKS.

Working through this project helped develop practical skills in cloud computing, containerization, CI/CD, infrastructure as code and monitoring.

---

**Project Repository:** https://github.com/Afthabahammed/nutriflow-deployment

**Maintained by:** Afthabahammed
