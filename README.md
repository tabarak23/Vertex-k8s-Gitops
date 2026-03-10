# Vertex Kubernetes GitOps Repository

## Overview

This repository contains the **Kubernetes deployment configurations for the Vertex platform** using a GitOps-based approach.

The platform is designed as a **complete cloud-native architecture** with separate repositories for application code, infrastructure provisioning, and Kubernetes deployment.

This repository focuses on **deploying microservices to Kubernetes using Helm charts** and managing environment-specific configurations.

Currently the Kubernetes setup is being **tested locally using Kind (Kubernetes in Docker)** to understand networking and service communication.  
The deployment will soon be moved to **Amazon EKS** as part of the full production architecture.

---

# Vertex Platform Repositories

The Vertex platform is structured using **three independent repositories**, each responsible for a specific layer of the system.

---

## Application Code & CI/CD

Repository  
https://github.com/tabarak23/Vertex-Microservices-code

Contains the **microservices source code and CI/CD pipelines**.  
The pipelines build applications, perform security scans, create Docker images, and push them to **Amazon ECR** using GitHub Actions with **OIDC authentication**.

---

## Infrastructure as Code

Repository  
https://github.com/tabarak23/Vertex-infra-terraform

This repository provisions the **complete AWS infrastructure** using Terraform, including the VPC, Amazon EKS cluster, node groups, databases, IAM roles, bastion host, and supporting services.

---

## Kubernetes GitOps Deployment (This Repository)

Repository  
https://github.com/tabarak23/Vertex-k8s-Gitops

This repository contains **Helm charts and Kubernetes manifests** used to deploy the microservices to the Kubernetes cluster using a GitOps workflow.

---

# GitOps Architecture

This repository represents the **deployment layer of the platform**.

Application container images are built by the CI pipeline and stored in **Amazon ECR**, and this repository defines how those images are deployed to Kubernetes.

```
Developer
   │
   │ Push Code
   ▼
Vertex-Microservices-code
   │
   │ CI/CD Pipeline
   ▼
Build • Test • Security Scans
   │
   ▼
Docker Images
   │
   ▼
Amazon ECR
   │
   ▼
Vertex-k8s-Gitops
   │
   │ Helm Charts
   ▼
Kubernetes Deployment
   │
   ▼
Cluster (Kind / EKS)
```

---

# Helm Chart Architecture

Each microservice is deployed using a **Helm chart** that provides reusable Kubernetes templates.

Example structure:

```
helm/
└── user-service
    ├── Chart.yaml
    ├── values.yaml
    ├── values-local.yaml
    ├── values-dev.yaml
    ├── values-staging.yaml
    ├── values-prod.yaml
    └── templates
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        ├── hpa.yaml
        ├── pdb.yaml
        ├── serviceaccount.yaml
        ├── externalsecret.yaml
        ├── secretstore.yaml
        ├── db-init-job.yaml
        ├── mysql.yaml
        ├── servicemonitor.yaml
        └── schema-configmap.yaml
```

This allows:

- reusable deployments
- environment specific configuration
- scalable Kubernetes workloads

---

# Kubernetes Components

## Deployment

Defines the application pods and container configuration.

Features include:

- configurable replicas
- resource requests and limits
- health probes
- environment variables from secrets

```
Deployment
   │
   ▼
Pods
   │
   ▼
Containers (Application)
```

---

## Service

Each microservice exposes an internal **ClusterIP service**.

```
Service
   │
   ▼
Load Balancing
   │
   ▼
Application Pods
```

---

## Ingress

Ingress resources expose services externally through an **Ingress Controller**.

```
Internet
   │
   ▼
Ingress Controller
   │
   ▼
Service
   │
   ▼
Pods
```

---

# Autoscaling

The platform uses **Horizontal Pod Autoscaler (HPA)** to scale applications automatically.

```
Metrics Server
   │
   ▼
CPU Metrics
   │
   ▼
Horizontal Pod Autoscaler
   │
   ▼
Scale Pods Automatically
```

Example configuration:

```
minReplicas: 2
maxReplicas: 10
targetCPU: 60%
```

---

# Secrets Management

Secrets are managed using **External Secrets Operator** integrated with **AWS Secrets Manager**.

```
AWS Secrets Manager
       │
       ▼
External Secrets Operator
       │
       ▼
Kubernetes Secret
       │
       ▼
Application Pods
```

This provides:

- centralized secret management
- automatic synchronization
- improved security

---

# IAM Roles for Service Accounts (IRSA)

Applications access AWS services securely using **IRSA**.

```
Kubernetes Service Account
       │
       ▼
IAM Role
       │
       ▼
AWS Services
```

This removes the need for static AWS credentials inside containers.

---

# Database Initialization (Local Development)

For local environments, the chart includes a **database initialization job**.

```
MySQL Pod
   │
   ▼
Init Job
   │
   ▼
Create Database
Create User
Apply Schema
```

This allows developers to run the service locally without external database dependencies.

---

# Observability

Monitoring is enabled using **Prometheus ServiceMonitor**.

```
Application Metrics
       │
       ▼
ServiceMonitor
       │
       ▼
Prometheus
       │
       ▼
Grafana Dashboards
```

Metrics endpoint:

```
/metrics
```

---

# Local Development Environment

The Kubernetes configuration is currently tested locally using **Kind (Kubernetes in Docker)**.

```
Developer Machine
      │
      ▼
Docker
      │
      ▼
Kind Cluster
      │
      ▼
Deploy Helm Charts
```

This setup helps validate:

- Kubernetes networking
- service communication
- Helm deployments

---

# Repository Structure

```
.
├── helm
│   └── user-service
│        ├── Chart.yaml
│        ├── values.yaml
│        ├── values-local.yaml
│        ├── values-dev.yaml
│        ├── values-staging.yaml
│        ├── values-prod.yaml
│        └── templates
│             ├── deployment.yaml
│             ├── service.yaml
│             ├── ingress.yaml
│             ├── hpa.yaml
│             ├── pdb.yaml
│             ├── serviceaccount.yaml
│             ├── externalsecret.yaml
│             ├── secretstore.yaml
│             ├── db-init-job.yaml
│             ├── mysql.yaml
│             └── servicemonitor.yaml
```

---

# End-to-End Platform Architecture

The complete platform architecture connects all repositories.

```
Developer
   │
   ▼
Vertex-Microservices-code
   │
   │ CI/CD Pipeline
   ▼
Build • Test • Security Scans
   │
   ▼
Docker Images
   │
   ▼
Amazon ECR
   │
   ▼
Vertex-k8s-Gitops
   │
   │ Helm Charts
   ▼
Kubernetes Cluster (Kind / EKS)
   │
   ▼
Microservices Running
   │
   ▼
RDS Databases
   │
   ▼
AWS Secrets Manager
```

---

# Future Improvements

Planned improvements include:

- deployment to Amazon EKS
- full GitOps automation using ArgoCD
- AWS Load Balancer Controller
- ExternalDNS integration
- centralized logging
- distributed tracing
- production-grade monitoring stack
