# 🚀 GitHub Actions with Azure OIDC Authentication

This repository demonstrates how to configure passwordless authentication between **GitHub Actions** and **Microsoft Azure** using OpenID Connect (OIDC) and Microsoft Entra ID Federated Credentials.

With OIDC, GitHub Actions can securely access Azure resources without storing Client Secrets.

---

## Architecture

```text
GitHub Actions
      │
      ▼
GitHub OIDC Token
      │
      ▼
Microsoft Entra ID
(Federated Credential)
      │
      ▼
Azure Service Principal
      │
      ▼
Azure Subscription
```

---


## Step 1: Create an App Registration into Microsoft Entra ID

Navigate to:

```text
Azure Portal
→ Microsoft Entra ID
→ App Registrations
→ New Registration
```

Example:

```text
Name: oidc-demo-001
```

After creation, note the following values:

- Application (Client) ID
- Directory (Tenant) ID
- Subscription ID

---

## Step 2: Assign Azure Permissions

Navigate to:

```text
Subscriptions
→ Access Control (IAM)
→ Add Role Assignment
```

Assign:

```text
Role: Contributor
Member: oidc-demo-001
```

---

## Step 3: Create a Federated Credential

Navigate to:

```text
App Registration
→ Certificates & Secrets
→ Federated Credentials
→ Add Credential
```

Configure:

| Setting | Value |
|----------|----------|
| Scenario | GitHub Actions deploying Azure resources |
| Organization | satyammaurya-cloud |
| Repository | GitHub-Actions-Azure-OIDC-passwordless |
| Entity Type | Branch |
| Branch | main |
| Name | auth |

### Configuration Screenshot

<img width="726" height="571" alt="image" src="https://github.com/user-attachments/assets/483b8d40-7535-4d42-a0fe-9373a9cf7c09" />

---

## Step 4: Create GitHub Secrets

Navigate to:

```text
Repository
→ Settings
→ Secrets and Variables
→ Actions
```

Create the following secrets:

| Secret | Value |
|----------|----------|
| AZURE_CLIENT_ID | Application (Client) ID |
| AZURE_TENANT_ID | Directory (Tenant) ID |
| AZURE_SUBSCRIPTION_ID | Azure Subscription ID |

---

## Step 5: Create GitHub Actions Workflow

Create:

```text
.github/workflows/main.yml
```

```yaml
name: Azure OIDC Authentication

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  azure-login:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Azure Login using OIDC
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Verify Login
        run: az account show

      - name: List Resource Groups
        run: az group list --output table
```
## Verify Authentication

Navigate to:

```text
GitHub Repository
→ Actions
→ Azure OIDC Authentication
```
Successful output:

```bash
az account show
```

---

## How It Works

1. GitHub Actions requests an OIDC token.
2. GitHub issues a signed JWT token.
3. Microsoft Entra ID validates the Organization, Repository, and Branch.
4. Azure issues a temporary access token.
5. GitHub Actions authenticates to Azure without using secrets.

---

## Security Benefits

- No Client Secrets
- No Secret Rotation
- Short-Lived Tokens
- Improved Security
- Microsoft Recommended Authentication Method

---
## Architechture
<img width="829" height="420" alt="arch" src="https://github.com/user-attachments/assets/f84c1e70-caa0-48ce-b571-44a80175fdd6" />


## References

- https://github.com/Azure/login
- https://docs.github.com/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect
