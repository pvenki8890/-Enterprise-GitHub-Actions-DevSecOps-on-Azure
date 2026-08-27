# Azure GitHub Actions DevSecOps

> Secure and cost-conscious CI/CD pipeline for a containerized Python application using GitHub Actions, Docker, Trivy, Azure Container Registry, and Azure Container Apps.

<p align="center">
  <img src="https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" alt="GitHub Actions">
  <img src="https://img.shields.io/badge/Python-3.13-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.13">
  <img src="https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/Azure-Container%20Apps-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Azure Container Apps">
  <img src="https://img.shields.io/badge/DevSecOps-Trivy%20%7C%20pip--audit-DC3545?style=for-the-badge" alt="DevSecOps">
</p>

---

## Overview

This project implements an end-to-end **DevSecOps workflow** for a Python Flask application.

The solution combines automated testing, dependency security auditing, container vulnerability scanning, secure Azure authentication, container image publishing, and Azure Container Apps deployment.

### Key Highlights

- Automated CI with GitHub Actions
- Python testing with `pytest`
- Dependency security auditing with `pip-audit`
- Docker containerization
- Trivy vulnerability scanning
- GitHub OIDC authentication to Azure
- Azure Container Registry with admin authentication disabled
- System-assigned Managed Identity
- Least-privilege `AcrPull` RBAC access
- Immutable Git commit SHA image tagging
- Azure Container Apps deployment
- Application and health endpoint verification
- Cost-conscious Consumption-based architecture

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
    pytest        pip-audit
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
    │ Azure Container     │
    │ Apps                │
    │ Consumption         │
    └──────────┬──────────┘
               │
               ▼
       Flask Application
          │          │
          ▼          ▼
         `/`      `/health`

---

## CI/CD Pipeline

The workflow is triggered by a push to the `main` branch.

| Stage | Technology | Purpose |
|:---|:---|:---|
| **Testing** | pytest | Validate application functionality |
| **Dependency Audit** | pip-audit | Detect vulnerable Python dependencies |
| **Container Build** | Docker | Package the application |
| **Security Scan** | Trivy | Scan the image for vulnerabilities |
| **Authentication** | GitHub OIDC | Secure passwordless Azure authentication |
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

The `container-security` job runs only after the `test` job succeeds.

---

## Security

Security is integrated into the CI/CD workflow rather than being treated as a separate final step.

### Dependency Security

`pip-audit` checks the Python dependencies for known vulnerabilities before the container image is published.

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

### Managed Identity

The Azure Container App uses a **system-assigned Managed Identity**.

The identity is granted the:

    AcrPull

role at the Azure Container Registry scope.

This allows the Container App to pull the private image without storing registry credentials.

### Immutable Image Tags

Container images are tagged using the Git commit SHA.

    pvgithubdevsecopsacr2026.azurecr.io/azure-github-actions-devsecops:<commit-sha>

This provides traceability:

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

The project contains a lightweight Python Flask application.

### Application Endpoint

**GET `/`**

    {
      "application": "Azure GitHub Actions DevSecOps",
      "status": "running",
      "message": "Application deployed successfully!"
    }

### Health Endpoint

**GET `/health`**

    {
      "status": "healthy"
    }

Both endpoints were successfully verified against the deployed Azure Container App.

---

## Deployment Verification

The final deployment was verified through the public Azure Container Apps endpoint.

### Application Verification

    GET /

Response:

    {
      "application": "Azure GitHub Actions DevSecOps",
      "status": "running",
      "message": "Application deployed successfully!"
    }

### Health Verification

    GET /health

Response:

    {
      "status": "healthy"
    }

These successful responses confirm that the application container was running correctly on Azure Container Apps.

---

## Evidence

Screenshots captured during implementation provide visual proof of the major project stages.

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

- `/` application response
- `/health` health response

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
| **CI/CD** | GitHub Actions, GitHub OIDC |
| **Testing** | pytest |
| **Dependency Security** | pip-audit |
| **Container Security** | Trivy |
| **Containerization** | Docker |
| **Azure Registry** | Azure Container Registry |
| **Azure Runtime** | Azure Container Apps |
| **Identity** | Azure Managed Identity |
| **Authorization** | Azure RBAC |

---

## DevSecOps Practices

| Practice | Implementation |
|:---|:---|
| **Automated Testing** | pytest in GitHub Actions |
| **Dependency Security** | pip-audit |
| **Container Security** | Trivy |
| **Security Gates** | HIGH/CRITICAL vulnerability checks |
| **Cloud Authentication** | GitHub OIDC |
| **Least Privilege** | Scoped Azure RBAC |
| **Secretless ACR Access** | Managed Identity |
| **Artifact Traceability** | Git SHA image tags |
| **Cost Control** | Consumption + 0–1 replicas |

---

## Project Result

The completed workflow is:

    Source
      ↓
    Test
      ↓
    Dependency Audit
      ↓
    Docker Build
      ↓
    Trivy Scan
      ↓
    GitHub OIDC
      ↓
    Azure Container Registry
      ↓
    Managed Identity + AcrPull
      ↓
    Azure Container Apps
      ↓
    Application Verification

**Result:** The Python application was successfully tested, security-scanned, published to Azure Container Registry, deployed to Azure Container Apps, and verified through both the application and health endpoints.

---
