# 🛡️ Azure GitHub Actions DevSecOps

[![CI/CD Pipeline](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![Docker](https://img.shields.io/badge/Container-Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Microsoft Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/)
[![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Security: Trivy](https://img.shields.io/badge/Security-Trivy%20Scan-1904DA?style=for-the-badge&logo=aqua&logoColor=white)](https://aquasecurity.github.io/trivy/)
[![Security: OIDC](https://img.shields.io/badge/Auth-OIDC%20Federated-success?style=for-the-badge&logo=openid&logoColor=white)](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation)

Enterprise-grade, cost-conscious CI/CD and DevSecOps container delivery workflow for a Python Flask application deployed to **Azure Container Apps** using **GitHub Actions**, **OIDC Workload Identity Federation**, and automated security scanning gates.

---

## 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [DevSecOps Security Controls](#-devsecops-security-controls)
- [CI/CD Pipeline Stages](#-cicd-pipeline-stages)
- [Azure Infrastructure](#-azure-infrastructure)
- [Application Endpoints](#-application-endpoints)
- [Docker Configuration](#-docker-configuration)
- [Repository Structure](#-repository-structure)
- [Local Development & Testing](#-local-development--testing)
- [Docker Execution](#-docker-execution)
- [Evidence & Verification](#-evidence--verification)
- [Cost-Conscious Design](#-cost-conscious-design)
- [Technologies Used](#-technologies-used)
- [Project Outcome](#-project-outcome)

---

## 🎯 Project Overview

The objective of this project is to implement a practical, production-ready DevSecOps pipeline using GitHub Actions and Microsoft Azure.

### 🌟 Key Highlights
- 🧪 **Automated Testing**: Unit and functional testing via `pytest`.
- 🔍 **Dependency Vulnerability Auditing**: Software Composition Analysis (SCA) with `pip-audit`.
- 🐳 **Immutable Container Packaging**: Commit SHA tagging strategy (`${{ github.sha }}`) preventing mutable `latest` tag risks.
- 🛡️ **Container Security Scanning**: Shift-left CVE scanning via `Trivy` configured to block high/critical vulnerabilities.
- 🔑 **Passwordless Authentication**: GitHub Actions OIDC integration with Azure Entra ID (zero persistent secrets).
- 📦 **Private Registry Hardening**: Azure Container Registry (ACR) with admin credentials disabled.
- 🔐 **Least-Privilege Authorization**: System-Assigned Managed Identity with scoped `AcrPull` role.
- 🚀 **Serverless Microservices**: Azure Container Apps (Consumption profile) with automated scale-to-zero capability.
- 💰 **Cost-Conscious Cloud Footprint**: Minimal resource footprint without idle compute overhead.

---

## 🏛️ Architecture

```text
 ┌─────────────────────────────────────────────────────────┐
 │                   Developer / Git Push                  │
 └────────────────────────────┬────────────────────────────┘
                              │
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │               GitHub Actions CI/CD Pipeline             │
 │                                                         │
 │  ┌─────────────────────┐       ┌─────────────────────┐  │
 │  │    Python Tests     │  and  │   pip-audit (SCA)   │  │
 │  │      (pytest)       │       │  Dependency Check   │  │
 │  └──────────┬──────────┘       └──────────┬──────────┘  │
 │             └──────────────┬──────────────┘             │
 │                            ▼                            │
 │              ┌───────────────────────────┐              │
 │              │   Docker Container Build  │              │
 │              └─────────────┬─────────────┘              │
 │                            ▼                            │
 │              ┌───────────────────────────┐              │
 │              │  Trivy Security Scanning  │              │
 │              │  (Fail on HIGH/CRITICAL)  │              │
 │              └─────────────┬─────────────┘              │
 │                            ▼                            │
 │              ┌───────────────────────────┐              │
 │              │ GitHub OIDC Authentication│              │
 │              │     (Azure Entra ID)      │              │
 │              └─────────────┬─────────────┘              │
 └────────────────────────────┼────────────────────────────┘
                              │
                              ▼ (Push immutable image: ${{ github.sha }})
 ┌─────────────────────────────────────────────────────────┐
 │             Azure Container Registry (ACR)              │
 │             (Admin authentication disabled)             │
 └────────────────────────────┬────────────────────────────┘
                              │
                              │ AcrPull (Managed Identity)
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │             Azure Container Apps (Consumption)          │
 │                                                         │
 │    ┌──────────────────────────────────────────────┐     │
 │    │           Flask Python Application           │     │
 │    │       GET /               GET /health        │     │
 │    │   (Status: Running)    (Status: Healthy)     │     │
 │    └──────────────────────────────────────────────┘     │
 └─────────────────────────────────────────────────────────┘

🔒 DevSecOps Security ControlsSecurity ControlImplementation MechanismSecurity ImpactShift-Left Dependency SCApip-audit -r app/requirements.txtIdentifies vulnerable Python packages before container build.Container CVE ScanningAqua Security TrivyScans OS and Python runtime packages for HIGH & CRITICAL vulnerabilities.Passwordless AuthOpenID Connect (OIDC)Eliminates long-lived Azure service principal secrets in GitHub Actions.Artifact ImmutabilityGit Commit SHA TaggingPrevents supply-chain drift and ensures complete build traceability.Registry HardeningACR RBAC (Admin Disabled)Blocks local account access and enforces Azure Entra ID authentication.Workload IdentitySystem-Assigned Managed IdentityGrants Azure Container Apps least-privilege AcrPull permissions.⚙️ CI/CD Pipeline StagesThe workflow executes automatically on every push to the main branch via .github/workflows/ci.yml:Plaintextci.yml
 ├── 🧪 Job 1: test
 │    ├── 📥 Checkout repository
 │    ├── 🐍 Set up Python 3.13
 │    ├── 📦 Install application dependencies
 │    ├── 🔬 Execute automated tests (pytest app/tests)
 │    └── 🛡️ Audit Python dependencies (pip-audit -r app/requirements.txt)
 │
 └── 🛡️ Job 2: container-security (depends on: test)
      ├── 📥 Checkout repository
      ├── 🐳 Build Docker image (azure-github-actions-devsecops:${{ github.sha }})
      ├── 🔍 Execute Trivy vulnerability scan
      ├── 🔑 Authenticate to Azure using OIDC (id-token: write)
      ├── 📦 Authenticate against Azure Container Registry (ACR)
      ├── 🏷️ Tag Docker image with ACR login server
      └── 🚀 Push immutable image to ACR
☁️ Azure InfrastructureResourceResource NameSKU / Workload ProfilePurpose & ConfigurationAzure Container Registrypvgithubdevsecopsacr2026BasicSecure container store (Admin user: Disabled).Container Apps Environmentcae-azure-github-actions-devsecopsConsumptionServerless execution boundary in Southeast Asia.Azure Container Appca-gha-devsecopsConsumptionHosts the containerized Flask app (0 to 1 auto-scaling).🏷️ Immutable Image Tagging & TraceabilityImages are tagged strictly with the Git commit SHA rather than mutable latest tags:Plaintextpvgithubdevsecopsacr2026.azurecr.io/azure-github-actions-devsecops:7bc7e14d05c57bb6ffc5cfb2508a80e4af05d960
This guarantees end-to-end traceability across the entire deployment lifecycle:$$\text{Git Commit SHA} \longrightarrow \text{Docker Image} \longrightarrow \text{Azure Container Registry} \longrightarrow \text{Azure Container Apps}$$🌐 Application Endpoints1. Root Application Endpoint (/)Bashcurl.exe https://<container-app-fqdn>/
JSON{
  "application": "Azure GitHub Actions DevSecOps",
  "message": "Application deployed successfully!",
  "status": "running"
}
2. Service Health Endpoint (/health)Bashcurl.exe https://<container-app-fqdn>/health
JSON{
  "status": "healthy"
}
🐳 Docker ConfigurationThe application uses an optimized python:3.13-slim base image:🪶 Minimal Footprint: Uses slim OS image to minimize attack surface.🔄 OS Patching: Updates core OS packages on build.⚡ Optimized Runtime: Configured with PYTHONUNBUFFERED=1.🚪 Port Binding: Exposes and serves traffic on 0.0.0.0:5000.📁 Repository StructurePlaintextazure-github-actions-devsecops/
│
├── .github/
│   └── workflows/
│       └── ci.yml               # GitHub Actions CI/CD & DevSecOps workflow
│
├── app/
│   ├── app.py                  # Python Flask application
│   ├── requirements.txt        # Python dependency manifest
│   └── tests/                  # Automated pytest test suites
│
├── .dockerignore               # Docker build exclusions
├── .gitignore                  # Git tracking exclusions
├── Dockerfile                  # Lean Python 3.13 slim container definition
└── README.md                   # Project documentation
💻 Local Development & Testing1. Set Up Virtual EnvironmentBash# Create virtual environment
python -m venv .venv

# Activate on Windows PowerShell
.\.venv\Scripts\Activate.ps1

# Activate on macOS / Linux
source .venv/bin/activate
2. Install & Audit DependenciesBashpython -m pip install --upgrade pip
python -m pip install -r app/requirements.txt
python -m pip_audit -r app/requirements.txt
3. Run Test SuiteBashpython -m pytest app/tests
4. Run Application LocallyBashpython app/app.py
# Application accessible at: http://localhost:5000 and http://localhost:5000/health
🐳 Docker ExecutionBuild Docker ImageBashdocker build -t azure-github-actions-devsecops:local .
Run Docker ContainerBashdocker run --rm -p 5000:5000 azure-github-actions-devsecops:local
Verify Local ContainerBashcurl.exe http://localhost:5000/
curl.exe http://localhost:5000/health
📸 Evidence & VerificationVisual proof and audit logs were captured across the end-to-end implementation:GitHub Actions: Successful workflow runs, test executions, pip-audit checks, and Trivy scan logs.Azure Security & Registry: Passwordless OIDC tokens, ACR repository tags, and disabled admin credentials.Azure Identity & Compute: System-assigned Managed Identity, AcrPull RBAC assignment, and active Container App revisions.Application Validation: Public curl verification of JSON payloads on / and /health.💡 Cost-Conscious DesignDesigned to avoid unnecessary infrastructure expenses during portfolio demonstrations:📉 ACR Basic SKU: Minimal registry storage overhead.⏱️ Scale-to-Zero (minReplicas: 0): Azure Container Apps shuts down compute when idle.🚫 Zero Dedicated VMs: No continuous compute cluster overhead.🧹 No Idle Analytics: Avoided unnecessary dedicated Log Analytics workspaces.🛠️ Technologies UsedRuntime & Framework: Python 3.13, FlaskTesting & Quality: pytest, pip-auditContainer Engine: DockerSecurity & Vulnerability Analysis: Aqua Security TrivyCI/CD Automation & Identity: GitHub Actions, OpenID Connect (OIDC)Cloud Infrastructure: Microsoft Azure, Azure Container Apps, Azure Container Registry (ACR), Azure Managed Identity, Azure RBAC🏆 Project OutcomePlaintext Source Code 
      │
      ▼
 Automated Python Tests (pytest)
      │
      ▼
 Dependency SCA Audit (pip-audit)
      │
      ▼
 Container Build & Trivy CVE Gate
      │
      ▼
 Passwordless GitHub OIDC Auth
      │
      ▼
 Azure Container Registry (Immutable SHA Tag)
      │
      ▼
 Managed Identity + AcrPull RBAC
      │
      ▼
 Azure Container Apps (Consumption)
      │
      ▼
 🚀 Validated Live Flask Application
