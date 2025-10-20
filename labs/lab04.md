# 4 - Reusable Workflows ♻️

In this lab you will create and consume reusable workflows to reduce duplication and standardize CI/CD processes.

> **Duration:** 10-15 minutes

## Learning Objectives

By the end of this lab, you will be able to:
- ✅ Create reusable workflows with workflow_call trigger
- ✅ Pass inputs and secrets to reusable workflows
- ✅ Return outputs from reusable workflows
- ✅ Call reusable workflows from other workflows
- ✅ Understand when to use reusable vs composite actions
- ✅ Share workflows across repositories

## References
- [Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Workflow syntax for reusable workflows](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_call)
- [Sharing workflows with your organization](https://docs.github.com/en/actions/using-workflows/sharing-workflows-secrets-and-runners-with-your-organization)

---

## 4.1 Make an existing workflow reusable

Let's convert the job-dependencies workflow to be reusable.

1. Open the workflow file [job-dependencies.yml](/.github/workflows/job-dependencies.yml)
2. Update the `on:` section to include `workflow_call`:

```yaml
name: 02-2. Job Dependencies (Reusable)

on:
  workflow_call:
    inputs:
      environment:
        description: 'Target environment for deployment'
        required: false
        type: string
        default: 'development'
      enable-tests:
        description: 'Run test suite'
        required: false
        type: boolean
        default: true
    outputs:
      build-version:
        description: 'The version number of the build'
        value: ${{ jobs.prepare.outputs.build-id }}
      deployment-url:
        description: 'URL of the deployment'
        value: ${{ jobs.deploy-staging.outputs.url }}
  workflow_dispatch:
  push:
    branches:
      - main
    paths:
      - '.github/workflows/job-dependencies.yml'
```

3. Update the `prepare` job to set the output:

```yaml
  prepare:
    runs-on: ubuntu-latest
    outputs:
      build-id: ${{ steps.build-info.outputs.build-id }}
    steps:
      - name: Generate build information
        id: build-info
        run: |
          echo "build-id=v1.${{ github.run_number }}" >> $GITHUB_OUTPUT
          echo "📦 Build ID: v1.${{ github.run_number }}"
```

4. Update the deploy-staging job to output a URL:

```yaml
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [test, prepare]
    outputs:
      url: ${{ steps.deploy.outputs.url }}
    environment:
      name: ${{ inputs.environment }}
    steps:
      - name: Deploy application
        id: deploy
        run: |
          echo "url=https://${{ inputs.environment }}.example.com" >> $GITHUB_OUTPUT
          echo "🚀 Deployed to https://${{ inputs.environment }}.example.com"
```

5. Make tests conditional based on input:

```yaml
  test:
    runs-on: ubuntu-latest
    needs: build
    if: inputs.enable-tests
    steps:
      - run: echo "🧪 Running tests (enabled via input)"
```

> 💡 **Reusable Workflow Design:**
> - Use `workflow_call` trigger to make workflows reusable
> - Define inputs for flexibility
> - Define outputs to return data to caller
> - Keep workflows focused on a single purpose

---

## 4.2 Create a caller workflow

Now let's create a workflow that calls our reusable workflow.

1. Open the workflow file [reusable-workflow-template.yml](/.github/workflows/reusable-workflow-template.yml)
2. Update it to call our reusable workflow with different configurations:

```yaml
name: 04-1. Reusable Workflow Caller

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  # Call reusable workflow for DEV environment
  deploy-dev:
    name: Deploy to DEV
    uses: ./.github/workflows/job-dependencies.yml
    with:
      environment: 'development'
      enable-tests: true
      
  # Call reusable workflow for staging (after DEV)
  deploy-staging:
    name: Deploy to Staging
    needs: deploy-dev
    if: github.ref == 'refs/heads/main'
    uses: ./.github/workflows/job-dependencies.yml
    with:
      environment: 'staging'
      enable-tests: true
      
  # Use outputs from reusable workflow
  report-deployment:
    name: Report Deployment Info
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: always()
    steps:
      - name: Display deployment information
        run: |
          echo "## Deployment Report 📊" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "- **Build Version:** ${{ needs.deploy-staging.outputs.build-version }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Deployment URL:** ${{ needs.deploy-staging.outputs.deployment-url }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Status:** ${{ needs.deploy-staging.result }}" >> $GITHUB_STEP_SUMMARY
```

3. Commit changes to `main` branch
4. Go to **Actions** and observe the workflow calling the reusable workflow
5. Check the job summary for the deployment report

> 🎯 **Calling Reusable Workflows:**
> - Use `uses: ./.github/workflows/filename.yml` for local workflows
> - Use `uses: owner/repo/.github/workflows/filename.yml@ref` for external repos
> - Pass inputs with `with:` block
> - Access outputs with `needs.<job-id>.outputs.<output-name>`

---

## 4.3 Call external reusable workflows

Let's call some community reusable workflows.

1. Add these jobs to [reusable-workflow-template.yml](/.github/workflows/reusable-workflow-template.yml):

```yaml
  # Call a public reusable workflow (super-linter)
  code-quality:
    name: Code Quality Check
    uses: github/super-linter/.github/workflows/super-linter.yml@v5
    with:
      # Configure linter to be less strict for demo
      validate-all-codebase: false
      
  # Call another local reusable workflow
  greeting:
    name: Send Greeting
    uses: ./.github/workflows/greet-everyone.yml
    with:
      name: 'Workshop Participant'
```

> 📚 **Finding Reusable Workflows:**
> - Check organization's `.github` repository
> - Look in the GitHub Marketplace
> - Review popular repositories for workflow patterns
> - Create an internal catalog for your team

---

## 4.4 Create a reusable workflow with secrets

Let's create a reusable deployment workflow that handles secrets properly.

1. Create a new file `.github/workflows/reusable-deploy.yml`:

```yaml
name: Reusable Deployment Workflow

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      app-name:
        required: true
        type: string
    secrets:
      deploy-token:
        required: true
      notification-webhook:
        required: false
    outputs:
      deployment-status:
        description: 'Status of the deployment'
        value: ${{ jobs.deploy.outputs.status }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    outputs:
      status: ${{ steps.deploy-step.outputs.status }}
      
    steps:
      - uses: actions/checkout@v4
      
      - name: Validate inputs
        run: |
          echo "🔍 Validating deployment configuration"
          echo "Environment: ${{ inputs.environment }}"
          echo "Application: ${{ inputs.app-name }}"
          
      - name: Deploy application
        id: deploy-step
        env:
          DEPLOY_TOKEN: ${{ secrets.deploy-token }}
        run: |
          echo "🚀 Deploying ${{ inputs.app-name }} to ${{ inputs.environment }}"
          # Simulate deployment
          echo "Using authentication token: ${DEPLOY_TOKEN:0:10}***"
          echo "status=success" >> $GITHUB_OUTPUT
          
      - name: Send notification
        if: secrets.notification-webhook != ''
        env:
          WEBHOOK_URL: ${{ secrets.notification-webhook }}
        run: |
          echo "📢 Sending deployment notification"
          # In real scenario: curl -X POST $WEBHOOK_URL
```

2. Call it from your caller workflow:

```yaml
  deploy-with-secrets:
    name: Deploy with Secrets
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: 'production'
      app-name: 'my-awesome-app'
    secrets:
      deploy-token: ${{ secrets.MY_REPO_SECRET }}
      # Optional secrets can be omitted
```

> 🔐 **Secrets in Reusable Workflows:**
> - Declare required secrets in `secrets:` section
> - Mark optional secrets with `required: false`
> - Caller passes secrets explicitly (not inherited)
> - Use `secrets: inherit` to pass all secrets automatically

---

## 4.5 Reusable Workflows vs Composite Actions

Understanding when to use each:

### Use Reusable Workflows When:
- ✅ You need to call jobs with different runners
- ✅ You want to use environments and approvals
- ✅ You need complex job dependencies
- ✅ You're orchestrating multiple steps across jobs
- ✅ You need secrets and environment-specific configurations

### Use Composite Actions When:
- ✅ You need to group steps within a single job
- ✅ You want to reuse a set of run commands
- ✅ You're creating a building block for workflows
- ✅ You need to run on the same runner
- ✅ You want to publish to the Marketplace

### Example Comparison:

**Composite Action** (for reusable steps):
```yaml
# .github/actions/setup-app/action.yml
runs:
  using: "composite"
  steps:
    - run: npm install
    - run: npm run build
```

**Reusable Workflow** (for reusable jobs):
```yaml
# .github/workflows/build.yml
on: workflow_call
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: ./.github/actions/setup-app
      - run: npm test
```

---

## 4.6 Complete Examples

<details>
<summary>Complete Reusable Workflow (job-dependencies.yml)</summary>

```yaml
name: 02-2. Job Dependencies (Reusable)

on:
  workflow_call:
    inputs:
      environment:
        description: 'Target environment'
        required: false
        type: string
        default: 'development'
      enable-tests:
        description: 'Run test suite'
        required: false
        type: boolean
        default: true
    outputs:
      build-version:
        value: ${{ jobs.prepare.outputs.build-id }}
      deployment-url:
        value: ${{ jobs.deploy-staging.outputs.url }}
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      build-id: ${{ steps.build-info.outputs.build-id }}
    steps:
      - name: Generate build info
        id: build-info
        run: echo "build-id=v1.${{ github.run_number }}" >> $GITHUB_OUTPUT

  build:
    runs-on: ubuntu-latest
    needs: prepare
    steps:
      - run: echo "🔨 Building version ${{ needs.prepare.outputs.build-id }}"

  test:
    runs-on: ubuntu-latest
    needs: build
    if: inputs.enable-tests
    steps:
      - run: echo "🧪 Running tests"

  deploy-staging:
    runs-on: ubuntu-latest
    needs: [test, prepare]
    outputs:
      url: ${{ steps.deploy.outputs.url }}
    environment:
      name: ${{ inputs.environment }}
    steps:
      - name: Deploy
        id: deploy
        run: |
          URL="https://${{ inputs.environment }}.example.com"
          echo "url=$URL" >> $GITHUB_OUTPUT
          echo "🚀 Deployed to $URL"
```
</details>

<details>
<summary>Complete Caller Workflow (reusable-workflow-template.yml)</summary>

```yaml
name: 04-1. Reusable Workflow Caller

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-dev:
    name: Deploy to DEV
    uses: ./.github/workflows/job-dependencies.yml
    with:
      environment: 'development'
      enable-tests: true
      
  deploy-staging:
    name: Deploy to Staging  
    needs: deploy-dev
    if: github.ref == 'refs/heads/main'
    uses: ./.github/workflows/job-dependencies.yml
    with:
      environment: 'staging'
      enable-tests: true
      
  report:
    name: Deployment Report
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: always()
    steps:
      - name: Create report
        run: |
          echo "## Deployment Report 📊" >> $GITHUB_STEP_SUMMARY
          echo "- **Version:** ${{ needs.deploy-staging.outputs.build-version }}" >> $GITHUB_STEP_SUMMARY
          echo "- **URL:** ${{ needs.deploy-staging.outputs.deployment-url }}" >> $GITHUB_STEP_SUMMARY
```
</details>

---

## 🎓 Key Takeaways

- **Reusable workflows** reduce duplication and standardize processes
- **Inputs** make workflows flexible and configurable
- **Outputs** allow returning data to caller workflows
- **Secrets** must be explicitly passed to reusable workflows
- **Workflow_call** trigger makes a workflow reusable
- Use reusable workflows for **job-level reuse**, composite actions for **step-level reuse**

## 💡 Best Practices

1. **Design for Reuse:** Make workflows parameterized with inputs
2. **Clear Outputs:** Document what outputs are available
3. **Version Control:** Use Git tags or commit SHAs when calling
4. **Security:** Never expose secrets, use proper permissions
5. **Documentation:** Document inputs, outputs, and usage examples
6. **Organization:** Store reusable workflows in a centralized repository

## 🏢 Organization Patterns

For enterprise use, consider:
- Creating a `.github` repository with shared workflows
- Establishing naming conventions (e.g., `reusable-build.yml`)
- Version tagging for stability (`@v1`, `@v2`)
- Creating a workflow catalog/documentation
- Using required workflows for compliance

---

## ✅ Lab Complete!

You've mastered reusable workflows! You now understand:
- Creating reusable workflows with inputs and outputs
- Calling reusable workflows locally and externally
- Passing secrets securely
- When to use reusable workflows vs composite actions
- Best practices for workflow reuse

**Next:** Proceed to [Lab 5: Custom Actions](/labs/lab05.md)
