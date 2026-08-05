# cicd-pipeline-automation
Reusable CI/CD pipeline templates for automated build, test, and deployment workflows. Covers Jenkins pipelines and Azure DevOps YAML pipelines with Docker and ECR integration.
[README.md](https://github.com/user-attachments/files/30732357/README.md)
# CI/CD Pipeline Automation

Reusable CI/CD pipeline templates for automated **build, test, and deploy** workflows on AWS (EKS + ECR) and Azure (AKS + ACR).

## What's included

| File | Purpose |
|---|---|
| `jenkins/Jenkinsfile` | Multi-stage Jenkins pipeline (lint → test → build → push → deploy → verify) |
| `azure-devops/azure-pipelines.yml` | Azure DevOps YAML pipeline with dev/prod stage gates |
| `scripts/deploy.sh` | Generic deploy script supporting both EKS and AKS |

## Pipeline stages

```
Checkout → Lint → Test → Build (Docker) → Push (ECR / ACR) → Deploy → Verify
```

- **Checkout** — checks out source, captures short Git SHA for image tagging
- **Lint** — static analysis (flake8 / eslint / hadolint)
- **Test** — unit tests with JUnit + coverage report publishing
- **Build** — Docker image tagged with `<build_number>-<git_sha>`
- **Push** — pushes to ECR (AWS) or ACR (Azure)
- **Deploy** — rolling update via `kubectl`; conditional on branch (`main` → prod, `develop` → dev)
- **Verify** — confirms pods are running; auto-rollback on failure

## Jenkins setup

1. Install plugins: `Pipeline`, `Docker Pipeline`, `AWS Steps`, `Azure CLI`
2. Add credentials: `aws-credentials`, `ECR_REPO_URL` (AWS) **or** `AZURE_SERVICE_CONNECTION` (Azure)
3. Create a pipeline job pointing to `jenkins/Jenkinsfile`
4. Set environment variables: `AWS_REGION`, `EKS_CLUSTER`, `AWS_ACCOUNT_ID`

## Azure DevOps setup

1. Create a new pipeline → select `azure-devops/azure-pipelines.yml`
2. Add pipeline variables:
   - `AZURE_SERVICE_CONNECTION` — your Azure service connection name
   - `ACR_NAME` — Azure Container Registry name (without `.azurecr.io`)
   - `AKS_CLUSTER_DEV` / `AKS_CLUSTER_PROD`
   - `RESOURCE_GROUP_DEV` / `RESOURCE_GROUP_PROD`
3. Create environments (`dev`, `production`) in Azure DevOps → Pipelines → Environments

## deploy.sh usage

```bash
chmod +x scripts/deploy.sh

# Deploy to AWS EKS
export AWS_REGION=us-east-1
export EKS_CLUSTER=eks-demo
./scripts/deploy.sh --provider aws --env prod --image 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:42

# Deploy to Azure AKS
export AZURE_RG=my-rg
export AKS_CLUSTER=my-aks
./scripts/deploy.sh --provider azure --env dev --image myacr.azurecr.io/my-app:42
```
