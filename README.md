# Azure GitHub Actions DevSecOps

> End-to-end DevSecOps pipeline for a containerized Python application using GitHub Actions, Docker, Trivy, Azure Container Registry, and Azure Container Apps.

<p align="center">
  <img src="https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" alt="GitHub Actions">
  <img src="https://img.shields.io/badge/Python-3.13-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.13">
  <img src="https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/Azure-Container%20Apps-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Azure Container Apps">
  <img src="https://img.shields.io/badge/DevSecOps-Trivy%20%7C%20pip--audit-DC3545?style=for-the-badge" alt="DevSecOps">
</p>

---

## Overview

This project demonstrates a practical **DevSecOps implementation** for a Python Flask application.

The solution integrates automated testing, dependency security auditing, container vulnerability scanning, secure Azure authentication, container image publishing, and deployment to Azure Container Apps.

### What This Project Demonstrates

- Building a CI pipeline with GitHub Actions
- Integrating security into the CI workflow
- Containerizing a Python application with Docker
- Scanning application dependencies with `pip-audit`
- Scanning container images with Trivy
- Using GitHub OIDC for passwordless Azure authentication
- Using Azure RBAC for least-privilege access
- Using Managed Identity for secure ACR access
- Using Git commit SHA tags for container traceability
- Running containerized workloads on Azure Container Apps

---

## Architecture

           Developer
              │
              │ git push
              ▼
    ┌─────────────────────┐
    │  GitHub Repository  │
    └──────────┬──────────┘
               │
               ▼
    ┌─────────────────────┐
    │   GitHub Actions    │
    └──────────┬──────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
      pytest       pip-audit
        │             │
        └──────┬──────┘
               │
               ▼
         Docker Build
               │
               ▼
       Trivy Security Scan
               │
          Security Gate
               │
               ▼
        GitHub OIDC → Azure
               │
               ▼
    ┌─────────────────────┐
    │ Azure Container     │
    │ Registry (ACR)      │
    └──────────┬──────────┘
               │
            AcrPull
               │
               ▼
    ┌─────────────────────┐
    │ Azure Container      │
    │ Apps                 │
    │ Consumption          │
    └──────────┬──────────┘
               │
               ▼
       Flask Application
          │          │
          ▼          ▼
         `/`      `/health`

---

## CI/CD Pipeline

The GitHub Actions workflow runs on pushes to the `main` branch.

| Stage | Tool / Service | Purpose |
|:---|:---|:---|
| **Testing** | `pytest` | Validate application functionality |
| **Dependency Audit** | `pip-audit` | Detect vulnerable Python dependencies |
| **Container Build** | Docker | Package the application |
| **Security Scan** | Trivy | Detect container vulnerabilities |
| **Authentication** | GitHub OIDC | Authenticate securely to Azure |
| **Registry** | Azure Container Registry | Store the validated image |
| **Authorization** | Azure RBAC | Provide scoped `AcrPull` access |
| **Runtime** | Azure Container Apps | Run the application |

### Pipeline Flow

    Code Push
        ↓
    Python Tests
        ↓
    Dependency Audit
        ↓
    Docker Build
        ↓
    Trivy Security Scan
        ↓
    GitHub OIDC Authentication
        ↓
    Push Image to ACR
        ↓
    Azure Container Apps
        ↓
    Application Verification

The container security job executes only after the test job succeeds.

---

## Security

Security is integrated throughout the software delivery lifecycle.

### Dependency Security

`pip-audit` checks Python dependencies for known vulnerabilities before the container image is published.

### Container Security

Trivy scans the Docker image for:

- Operating system vulnerabilities
- Application/library vulnerabilities
- HIGH severity vulnerabilities
- CRITICAL severity vulnerabilities

The workflow is configured to fail when applicable HIGH or CRITICAL vulnerabilities are detected.

### GitHub OIDC

GitHub Actions authenticates to Azure using **OpenID Connect (OIDC)** instead of storing a long-lived Azure client secret.

    GitHub Actions
          │
          │ OIDC Token
          ▼
      Azure Entra ID
          │
          ▼
    Federated Identity
          │
          ▼
      Azure Resources

### Managed Identity + RBAC

The Azure Container App uses a **system-assigned Managed Identity**.

The identity is granted the:

    AcrPull

role at the Azure Container Registry scope.

This allows the Container App to pull the private image without storing registry credentials.

### Immutable Container Images

Images are tagged using the Git commit SHA:

    pvgithubdevsecopsacr2026.azurecr.io/azure-github-actions-devsecops:<commit-sha>

This creates a traceable relationship between:

    Git Commit
        ↓
    Docker Image
        ↓
    Azure Container Registry
        ↓
    Azure Container Apps

---

## Azure Infrastructure

The completed implementation uses three primary Azure resources.

| Resource | Purpose |
|:---|:---|
| **Azure Container Registry** | Stores the container image |
| **Container Apps Environment** | Provides the hosting environment |
| **Azure Container App** | Runs the Flask application |

### Azure Container Registry

| Property | Value |
|:---|:---|
| **Name** | `pvgithubdevsecopsacr2026` |
| **Login Server** | `pvgithubdevsecopsacr2026.azurecr.io` |
| **SKU** | Basic |
| **Admin Authentication** | Disabled |

### Container Apps Environment

| Property | Value |
|:---|:---|
| **Name** | `cae-azure-github-actions-devsecops` |
| **Region** | Southeast Asia |
| **Workload Profile** | Consumption |
| **Application Logs** | None |

### Container App

| Property | Value |
|:---|:---|
| **Name** | `ca-gha-devsecops` |
| **Region** | Southeast Asia |
| **Revision Mode** | Single |
| **Minimum Replicas** | 0 |
| **Maximum Replicas** | 1 |
| **Target Port** | 5000 |
| **Ingress** | External |

---

## Application

The project contains a lightweight **Python Flask** application.

### `GET /`

    {
      "application": "Azure GitHub Actions DevSecOps",
      "status": "running",
      "message": "Application deployed successfully!"
    }

### `GET /health`

    {
      "status": "healthy"
    }

Both endpoints were successfully verified against the deployed Azure Container App.

---

## Deployment Evidence

Screenshots captured during implementation provide proof of the completed workflow.

### GitHub Actions

- Successful CI workflow
- Python test execution
- Dependency security audit
- Docker image build
- Successful Trivy security scan
- Azure authentication
- Successful ACR image push

### Azure

- Azure Container Registry configuration
- Container Apps Environment
- Container App configuration
- System-assigned Managed Identity
- `AcrPull` role assignment
- Running container revision
- Deployed container image

### Application

- Successful `GET /` response
- Successful `GET /health` response

---

## Project Structure

    azure-github-actions-devsecops/
    │
    ├── .github/
    │   └── workflows/
    │       └── ci.yml
    │
    ├── app/
    │   ├── app.py
    │   ├── requirements.txt
    │   └── tests/
    │
    ├── Dockerfile
    └── README.md

---

## Technology Stack

| Category | Technologies |
|:---|:---|
| **Application** | Python 3.13, Flask |
| **CI/CD** | GitHub Actions |
| **Authentication** | GitHub OIDC |
| **Testing** | pytest |
| **Dependency Security** | pip-audit |
| **Container Security** | Trivy |
| **Containerization** | Docker |
| **Container Registry** | Azure Container Registry |
| **Runtime** | Azure Container Apps |
| **Identity** | Azure Managed Identity |
| **Authorization** | Azure RBAC |

---

## Key DevSecOps Practices

| Practice | Implementation |
|:---|:---|
| **Shift-Left Security** | Dependency and container scanning in CI |
| **Automated Testing** | pytest before container publication |
| **Dependency Security** | pip-audit |
| **Container Security** | Trivy |
| **Security Gates** | HIGH/CRITICAL vulnerability checks |
| **Passwordless Authentication** | GitHub OIDC |
| **Least Privilege** | Scoped Azure RBAC |
| **Secretless ACR Access** | Managed Identity |
| **Artifact Traceability** | Git commit SHA image tags |
| **Cloud-Native Runtime** | Azure Container Apps |

---

## Project Result

The completed workflow demonstrates:

    Source Code
         ↓
    Automated Testing
         ↓
    Dependency Security Audit
         ↓
    Docker Build
         ↓
    Trivy Security Scan
         ↓
    GitHub OIDC Authentication
         ↓
    Azure Container Registry
         ↓
    Managed Identity + AcrPull
         ↓
    Azure Container Apps
         ↓
    Application + Health Verification

### Outcome

The Python application was successfully:

- Tested through GitHub Actions
- Audited for dependency vulnerabilities
- Containerized with Docker
- Scanned with Trivy
- Published to Azure Container Registry
- Deployed to Azure Container Apps
- Verified through application and health endpoints
  
---
