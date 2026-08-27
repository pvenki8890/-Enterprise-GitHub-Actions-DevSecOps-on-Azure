# Azure GitHub Actions DevSecOps

Enterprise-style CI/CD and DevSecOps pipeline using GitHub Actions, Docker, and Microsoft Azure.

This project demonstrates a secure and cost-conscious container delivery workflow for a Python Flask application. The application is tested, dependency-audited, containerized, security-scanned, pushed to Azure Container Registry, and deployed to Azure Container Apps.

---

## Project Overview

The goal of this project is to implement a practical DevSecOps pipeline using GitHub Actions and Microsoft Azure.

The project demonstrates:

- Automated Python testing with pytest
- Python dependency vulnerability auditing with pip-audit
- Docker image creation
- Container vulnerability scanning with Trivy
- Secure GitHub Actions authentication to Azure using OpenID Connect (OIDC)
- Azure Container Registry (ACR)
- Azure Managed Identity
- Azure RBAC with least-privilege AcrPull access
- Immutable Docker image tagging using the Git commit SHA
- Azure Container Apps deployment
- Application and health endpoint verification
- Cost-conscious Azure resource configuration

---

## Architecture

```text
                         Developer
                             |
                             | git push
                             v
                  +-----------------------+
                  |   GitHub Repository   |
                  +-----------+-----------+
                              |
                              v
                  +-----------------------+
                  |    GitHub Actions     |
                  +-----------+-----------+
                              |
                 +------------+------------+
                 |                         |
                 v                         v
          Python Tests               pip-audit
                 |                         |
                 +------------+------------+
                              |
                              v
                       Docker Build
                              |
                              v
                     Trivy Security Scan
                              |
                       Scan Successful
                              |
                              v
                    GitHub OIDC → Azure
                              |
                              v
                  +-----------------------+
                  | Azure Container       |
                  | Registry (ACR)        |
                  +-----------+-----------+
                              |
                         AcrPull
                              |
                              v
                  +-----------------------+
                  | Azure Container Apps  |
                  | Consumption           |
                  +-----------+-----------+
                              |
                              v
                    Flask Application
                              |
                    +---------+---------+
                    |                   |
                    v                   v
                   /                 /health
                Application           Healthy
                 Response             Status


CI/CD Pipeline

The GitHub Actions workflow is triggered when code is pushed to the main branch.

The pipeline performs the following stages:

1. Python Testing

The workflow sets up Python 3.13 and installs the application dependencies.

Tests are executed using:

python -m pytest app/tests

The container security job depends on the successful completion of the test job.

2. Dependency Security Audit

Python dependencies are audited using:

python -m pip_audit -r app/requirements.txt

This provides an additional security gate before the container image is published.

3. Docker Build

The application is packaged into a Docker image using the project's Dockerfile.

The image is tagged using the GitHub commit SHA:

azure-github-actions-devsecops:${{ github.sha }}

This provides traceability between the source commit and the resulting container image.

4. Container Security Scan

The Docker image is scanned using Trivy.

The scan checks:

Operating system vulnerabilities
Python/library vulnerabilities
HIGH severity vulnerabilities
CRITICAL severity vulnerabilities

The workflow is configured to fail when applicable HIGH or CRITICAL vulnerabilities are detected.

5. Secure Azure Authentication

GitHub Actions authenticates to Azure using OpenID Connect (OIDC).

The workflow uses:

permissions:
  contents: read
  id-token: write

No long-lived Azure client secret is stored in the GitHub Actions workflow.

6. Push Image to Azure Container Registry

After the security scan succeeds, the Docker image is authenticated against Azure Container Registry, tagged with the ACR login server, and pushed to the registry.

The image uses the Git commit SHA as its tag.

7. Azure Container Apps Deployment

The validated container image is deployed to Azure Container Apps.

The application runs using the Consumption workload profile with:

Minimum replicas: 0
Maximum replicas: 1

This allows the application to scale down when idle and keeps the project suitable for a small portfolio/lab workload.

DevSecOps Security Controls

Security is incorporated throughout the software delivery lifecycle.

GitHub Actions OIDC

GitHub Actions uses workload identity federation to authenticate to Azure.

GitHub Actions
      |
      | OIDC Token
      v
Azure Entra ID
      |
      v
Federated Identity Credential
      |
      v
Azure Resources

This avoids the need for long-lived Azure credentials in GitHub Actions.

Dependency Security

pip-audit checks Python dependencies for known vulnerabilities before the container image is published.

Container Security

Trivy scans the built Docker image for HIGH and CRITICAL vulnerabilities before the image is pushed to ACR.

Azure Container Registry Security

Azure Container Registry admin authentication is disabled.

The project uses identity-based authentication instead of storing an ACR username and password.

Managed Identity

The Azure Container App uses a system-assigned managed identity.

The identity has the:

AcrPull

role scoped to the Azure Container Registry.

This allows Azure Container Apps to securely pull the private container image without storing registry credentials.

Application

The project contains a lightweight Python Flask application.

Root Endpoint
/

Successful response:

{
  "application": "Azure GitHub Actions DevSecOps",
  "status": "running",
  "message": "Application deployed successfully!"
}
Health Endpoint
/health

Successful response:

{
  "status": "healthy"
}

The deployed application and health endpoint were successfully verified against the Azure Container Apps public endpoint.

Docker Configuration

The application uses Python 3.13 slim as its base image.

The Dockerfile:

Uses a lightweight Python base image
Updates the underlying OS packages
Installs application dependencies
Upgrades setuptools
Uses Python unbuffered output
Exposes port 5000
Runs the Flask application on 0.0.0.0:5000

Application startup command:

python app.py
Repository Structure
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
GitHub Actions Workflow

The workflow is located at:

.github/workflows/ci.yml

The workflow is triggered by pushes to the main branch.

The workflow contains two jobs.

Test Job
test
 |
 +-- Checkout repository
 |
 +-- Set up Python 3.13
 |
 +-- Install dependencies
 |
 +-- Run pytest
 |
 +-- Install pip-audit
 |
 +-- Audit Python dependencies
Container Security Job

The container security job runs only after the test job succeeds.

container-security
 |
 +-- Checkout repository
 |
 +-- Build Docker image
 |
 +-- Verify container dependencies
 |
 +-- Trivy vulnerability scan
 |
 +-- Azure OIDC authentication
 |
 +-- ACR authentication
 |
 +-- Tag Docker image
 |
 +-- Push image to ACR
Azure Infrastructure

The completed project uses three intentional Azure resources.

Azure Resource	Purpose
Azure Container Registry	Stores the Docker image
Azure Container Apps Environment	Provides the Container Apps hosting environment
Azure Container App	Runs the Flask application
Azure Container Registry
Name:
pvgithubdevsecopsacr2026

Login server:

pvgithubdevsecopsacr2026.azurecr.io

Configuration:

SKU: Basic
Admin authentication: Disabled
Container Apps Environment
Name:
cae-azure-github-actions-devsecops

Region:

Southeast Asia

Workload profile:

Consumption

Application logs destination:

None
Container App
Name:
ca-gha-devsecops

Region:

Southeast Asia

Configuration:

Revision mode: Single
Minimum replicas: 0
Maximum replicas: 1
Target port: 5000
External ingress: Enabled
Immutable Container Image

The container image is tagged using the Git commit SHA.

Example:

7bc7e14d05c57bb6ffc5cfb2508a80e4af05d960

The resulting image reference is:

pvgithubdevsecopsacr2026.azurecr.io/azure-github-actions-devsecops:7bc7e14d05c57bb6ffc5cfb2508a80e4af05d960

This provides traceability across the deployment chain:

Git Commit
    |
    v
Docker Image
    |
    v
Azure Container Registry
    |
    v
Azure Container Apps

Using commit-based image tags avoids relying on a mutable latest tag for the deployed application.

Deployment Verification

The deployed application was verified using the Azure Container Apps public endpoint.

Application Verification
curl.exe https://<container-app-fqdn>/

Successful response:

{
  "application": "Azure GitHub Actions DevSecOps",
  "message": "Application deployed successfully!",
  "status": "running"
}
Health Verification
curl.exe https://<container-app-fqdn>/health

Successful response:

{
  "status": "healthy"
}

Both endpoints were successfully tested against the deployed application.

Cost-Conscious Design

This project was intentionally designed to avoid unnecessary Azure infrastructure and excessive ongoing costs.

The final architecture uses:

Azure Container Registry Basic SKU
Azure Container Apps Consumption workload profile
Minimum replicas set to 0
Maximum replicas set to 1
No dedicated virtual machines
No unnecessary always-on compute
No unnecessary Log Analytics workspace
No unnecessary supporting infrastructure

The resource group contains only the three intentional resources required for the project:

Azure Container Registry
Azure Container Apps Environment
Azure Container App

This configuration is intended for a portfolio/lab project rather than a high-traffic production workload.

Azure pricing can change. Actual costs should always be checked in the Azure Portal and current Azure pricing documentation.

Local Development
Create a Python Virtual Environment

Windows PowerShell:

python -m venv .venv

Activate it:

.\.venv\Scripts\Activate.ps1
Install Dependencies
python -m pip install -r app\requirements.txt
Run Tests
python -m pytest app/tests
Run the Application
python app\app.py

The application will be available at:

http://localhost:5000

Health endpoint:

http://localhost:5000/health
Docker
Build the Docker Image
docker build -t azure-github-actions-devsecops .
Run the Container
docker run --rm -p 5000:5000 azure-github-actions-devsecops

Test the application:

curl.exe http://localhost:5000/

Test the health endpoint:

curl.exe http://localhost:5000/health
Evidence and Screenshots

Screenshots were captured during the implementation and verification of the project.

The evidence covers the major stages of the solution, including:

GitHub Actions
Successful GitHub Actions workflow
Python test execution
Successful dependency audit
Docker image build
Successful Trivy security scan
Azure OIDC authentication
Successful ACR image push
Azure
Azure Container Registry configuration
ACR repository and image tag
Container Apps environment
Container App configuration
System-assigned Managed Identity
AcrPull role assignment
Running Container App revision
Deployed container image
Application
Application root endpoint returning the expected JSON response
/health endpoint returning a healthy status

These screenshots provide visual proof of the implemented CI/CD pipeline, security controls, Azure infrastructure, and successful application deployment.

Technologies Used
Python 3.13
Flask
pytest
pip-audit
Docker
Trivy
GitHub Actions
GitHub OIDC
Microsoft Azure
Azure Container Registry
Azure Container Apps
Azure Managed Identity
Azure RBAC
DevSecOps Principles Demonstrated

This project demonstrates several practical DevSecOps principles:

Shift-left security through dependency and container scanning
Automated testing before container publication
Immutable artifacts using Git commit SHA image tags
Secure authentication using GitHub OIDC
Least-privilege access using Azure RBAC
Managed Identity for Container Apps to ACR authentication
Security gates before publishing container images
Traceability from source commit to container image
Cost-aware cloud architecture
Health verification of the deployed application
Project Outcome

The completed project demonstrates a practical DevSecOps workflow for a containerized Python application.

The final implementation provides:

Source Code
    |
    v
Automated Python Tests
    |
    v
Dependency Security Audit
    |
    v
Docker Build
    |
    v
Trivy Container Scan
    |
    v
GitHub OIDC Authentication
    |
    v
Azure Container Registry
    |
    v
Managed Identity + AcrPull
    |
    v
Azure Container Apps
    |
    v
Running Flask Application
    |
    v
Application + Health Verification

The application was successfully deployed to Azure Container Apps and verified through both the application and health endpoints.
