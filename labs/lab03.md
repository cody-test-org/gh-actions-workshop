# 3 - Environments, Secrets & Security 🔐

In this lab you will implement secure workflows with environment protection, secret management, and security best practices.

> **Duration:** 10-15 minutes

## Learning Objectives

By the end of this lab, you will be able to:
- ✅ Create and configure deployment environments with protection rules
- ✅ Manage secrets at repository and environment levels
- ✅ Implement least-privilege permissions for GITHUB_TOKEN
- ✅ Use environment URLs for deployment tracking
- ✅ Configure manual approval workflows
- ✅ Understand OIDC for cloud provider authentication (concept)

## References
- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Encrypted secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Permissions for GITHUB_TOKEN](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)
- [Configuring OpenID Connect in cloud providers](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

---

## 3.1 Create environments with protection rules

Let's set up a multi-environment deployment pipeline with proper controls.

### Step 1: Create Development Environment

1. Navigate to your repository **Settings** → **Environments**
2. Click **New environment**
3. Name it `DEV`
4. Click **Configure environment**
5. Leave protection rules empty (DEV deploys automatically)
6. Add an environment variable:
   - Name: `ENVIRONMENT_URL`
   - Value: `https://dev.example.com`
7. Click **Save protection rules**

### Step 2: Create UAT Environment with Approvals

1. Create another environment named `UAT`
2. Enable **Required reviewers**
3. Add yourself as a required reviewer
4. Set **Wait timer** to `0` minutes (optional: use 5 for production)
5. Add an environment secret:
   - Name: `MY_ENV_SECRET`
   - Value: `uat-secret-value-123`
6. Add environment variable:
   - Name: `ENVIRONMENT_URL`
   - Value: `https://uat.example.com`
7. Save the configuration

### Step 3: Create Production Environment

1. Create an environment named `PROD`
2. Enable **Required reviewers** (add yourself)
3. Optional: **Enable deployment branches** → Select `main` only
4. Add environment secret:
   - Name: `MY_ENV_SECRET`
   - Value: `prod-secret-value-456`
5. Add environment variable:
   - Name: `ENVIRONMENT_URL`
   - Value: `https://prod.example.com`

> 🔒 **Security Best Practice:** Always require manual approval for production deployments. Consider using wait timers to allow rollback windows.

---

## 3.2 Create repository-level secrets

1. Navigate to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Create a secret:
   - Name: `MY_REPO_SECRET`
   - Value: `my-super-secret-repo-value`
4. Click **Add secret**

> 💡 **Secret Hierarchy:** 
> - **Organization secrets** - Shared across repos
> - **Repository secrets** - Available to all workflows
> - **Environment secrets** - Only available when deploying to that environment

---

## 3.3 Implement secure workflow with least-privilege permissions

1. Open the workflow file [environments-secrets.yml](/.github/workflows/environments-secrets.yml)
2. Update the workflow with modern security practices:

```yaml
name: 03-1. Environments and Secrets

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

# Implement least-privilege permissions
permissions:
  contents: read      # Read repository code
  deployments: write  # Create deployment statuses
  id-token: write     # Required for OIDC (future use)

env:
  PROD_URL: 'https://prod.example.com'
  DOCS_URL: 'https://docs.example.com'
```

> 🔐 **Permission Best Practice:** Start with minimal permissions and only add what's needed. The principle of least privilege reduces security risks.

---

## 3.4 Add secret usage and masking demonstration

Add a job to demonstrate secure secret handling:

```yaml
jobs:
  demonstrate-secrets:
    name: Secret Handling Demo
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - name: Using secrets securely
        env:
          REPO_SECRET: ${{ secrets.MY_REPO_SECRET }}
        run: |
          echo "✅ Secret loaded into environment variable"
          # Secrets are automatically masked in logs
          echo "Secret value: $REPO_SECRET"
          
      - name: Demonstrate secret masking
        run: |
          echo "⚠️ Warning: GitHub automatically masks secrets in logs"
          echo "But you should avoid printing them intentionally!"
          echo "Secret: ${{ secrets.MY_REPO_SECRET }}"
          
      - name: Using secrets with actions
        uses: ./.github/actions/hello-world-js
        with:
          who-to-greet: ${{ secrets.MY_REPO_SECRET }}
```

> ⚠️ **Critical Warning:** Never use secrets in:
> - URLs or query parameters (they appear in logs)
> - Conditional statements that might fail
> - Step names or job names
> - Anything that gets stored in artifacts

---

## 3.5 Create multi-environment deployment pipeline

Add the deployment pipeline with environment protection:

```yaml
  deploy-dev:
    name: Deploy to DEV
    runs-on: ubuntu-latest
    needs: demonstrate-secrets
    environment:
      name: DEV
      url: ${{ vars.ENVIRONMENT_URL }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to DEV environment
        run: |
          echo "🚀 Deploying to DEV: ${{ vars.ENVIRONMENT_URL }}"
          echo "Deployment: Automatic (no approval required)"
          
      - name: Create deployment summary
        run: |
          echo "## DEV Deployment Summary 🚀" >> $GITHUB_STEP_SUMMARY
          echo "- **Environment:** DEV" >> $GITHUB_STEP_SUMMARY
          echo "- **URL:** ${{ vars.ENVIRONMENT_URL }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Status:** ✅ Deployed" >> $GITHUB_STEP_SUMMARY

  deploy-uat:
    name: Deploy to UAT
    runs-on: ubuntu-latest
    needs: deploy-dev
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment:
      name: UAT
      url: ${{ vars.ENVIRONMENT_URL }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to UAT environment
        env:
          ENV_SECRET: ${{ secrets.MY_ENV_SECRET }}
        run: |
          echo "🚀 Deploying to UAT: ${{ vars.ENVIRONMENT_URL }}"
          echo "Deployment requires manual approval ✋"
          echo "Environment secret is available: ${ENV_SECRET:0:5}***"
          
      - name: Run smoke tests
        run: |
          echo "🧪 Running smoke tests..."
          echo "✅ Health check passed"
          echo "✅ API endpoints responding"

  deploy-prod:
    name: Deploy to PROD
    runs-on: ubuntu-latest
    needs: deploy-uat
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment:
      name: PROD
      url: ${{ vars.ENVIRONMENT_URL }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Production
        env:
          ENV_SECRET: ${{ secrets.MY_ENV_SECRET }}
        run: |
          echo "🚀 Deploying to PROD: ${{ vars.ENVIRONMENT_URL }}"
          echo "Critical deployment - requires approval ✋"
          echo "Build: ${{ github.run_number }}"
          echo "SHA: ${{ github.sha }}"
          
      - name: Post-deployment verification
        run: |
          echo "🔍 Verifying deployment..."
          echo "✅ Application health: OK"
          echo "✅ Database connections: OK"
          echo "✅ External integrations: OK"
          
      - name: Notify deployment
        run: |
          echo "## 🎉 Production Deployment Complete" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "- **Version:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Deployed by:** @${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Environment:** ${{ vars.ENVIRONMENT_URL }}" >> $GITHUB_STEP_SUMMARY
```

---

## 3.6 Test the workflow

1. Commit all changes to the `main` branch
2. Navigate to **Actions** and observe the workflow execution
3. The workflow will:
   - Run `demonstrate-secrets` automatically
   - Deploy to `DEV` automatically
   - **Pause at UAT** - requiring your approval
4. Approve the UAT deployment:
   - Click on the waiting workflow run
   - Click **Review deployments**
   - Select **UAT**
   - Add a comment (optional)
   - Click **Approve and deploy**
5. After UAT completes, approve PROD deployment

> ✅ **Success Check:** You should see the deployment URL badges in the Actions UI after approval!

---

## 3.7 (Advanced) Understanding OIDC for Cloud Authentication

Instead of storing long-lived cloud credentials as secrets, use OpenID Connect (OIDC):

### Concept Overview

```yaml
jobs:
  deploy-to-aws:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # Required for OIDC
      contents: read
    
    steps:
      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: us-east-1
          # No access keys needed! 🎉
      
      - name: Deploy to AWS
        run: |
          aws s3 sync ./dist s3://my-bucket/
```

> 🔐 **OIDC Benefits:**
> - No long-lived credentials stored in secrets
> - Automatic credential rotation
> - Scoped to specific repositories and branches
> - Audit trail in cloud provider logs

**Setup Requirements:**
1. Configure trust relationship in AWS IAM (or Azure, GCP)
2. Grant `id-token: write` permission
3. Use cloud-specific auth action with `role-to-assume`

**Learn more:** [About security hardening with OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

---

## 3.8 Complete Workflow Example

<details>
<summary>Click to see the complete workflow</summary>

```yaml
name: 03-1. Environments and Secrets

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

# Implement least-privilege permissions
permissions:
  contents: read
  deployments: write
  id-token: write

env:
  PROD_URL: 'https://prod.example.com'

jobs:
  demonstrate-secrets:
    name: Secret Handling Demo
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - name: Using secrets securely
        env:
          REPO_SECRET: ${{ secrets.MY_REPO_SECRET }}
        run: |
          echo "✅ Secret loaded into environment variable"
          echo "Secret value: $REPO_SECRET"
          
      - name: Demonstrate secret masking
        run: |
          echo "⚠️ Warning: GitHub automatically masks secrets in logs"
          echo "Secret: ${{ secrets.MY_REPO_SECRET }}"

  deploy-dev:
    name: Deploy to DEV
    runs-on: ubuntu-latest
    needs: demonstrate-secrets
    environment:
      name: DEV
      url: ${{ vars.ENVIRONMENT_URL }}
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to DEV environment
        run: echo "🚀 Deploying to DEV: ${{ vars.ENVIRONMENT_URL }}"

  deploy-uat:
    name: Deploy to UAT
    runs-on: ubuntu-latest
    needs: deploy-dev
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment:
      name: UAT
      url: ${{ vars.ENVIRONMENT_URL }}
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to UAT environment
        env:
          ENV_SECRET: ${{ secrets.MY_ENV_SECRET }}
        run: |
          echo "🚀 Deploying to UAT: ${{ vars.ENVIRONMENT_URL }}"
          echo "Environment secret available: ${ENV_SECRET:0:5}***"

  deploy-prod:
    name: Deploy to PROD
    runs-on: ubuntu-latest
    needs: deploy-uat
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment:
      name: PROD
      url: ${{ vars.ENVIRONMENT_URL }}
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Production
        env:
          ENV_SECRET: ${{ secrets.MY_ENV_SECRET }}
        run: |
          echo "🚀 Deploying to PROD: ${{ vars.ENVIRONMENT_URL }}"
          echo "Build: ${{ github.run_number }}"
      - name: Create deployment summary
        run: |
          echo "## 🎉 Production Deployment Complete" >> $GITHUB_STEP_SUMMARY
          echo "- **Version:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Deployed by:** @${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
```
</details>

---

## 🎓 Key Takeaways

- **Environment protection:** Use manual approvals for critical deployments
- **Secret hierarchy:** Repository secrets for shared values, environment secrets for deployment-specific
- **Least privilege:** Grant minimal GITHUB_TOKEN permissions needed
- **Secret masking:** GitHub masks secrets automatically, but avoid printing them
- **OIDC:** Modern authentication eliminates long-lived credentials
- **Deployment tracking:** Use environment URLs for visibility

## 🔒 Security Checklist

- ✅ Environments configured with appropriate protection rules
- ✅ Production requires manual approval
- ✅ Secrets never exposed in logs, URLs, or artifacts
- ✅ GITHUB_TOKEN uses least-privilege permissions
- ✅ Environment URLs configured for deployment tracking
- ✅ Consider OIDC for cloud deployments

---

## ✅ Lab Complete!

You've mastered secure deployment workflows! You now understand:
- Environment protection and manual approvals
- Secret management at multiple levels
- Least-privilege permissions
- Secure secret handling practices
- Modern OIDC authentication patterns

**Next:** Proceed to [Lab 4: Reusable Workflows](/labs/lab04.md)
