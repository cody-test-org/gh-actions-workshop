# 2 - Workflow Syntax & Job Dependencies 🔄

In this lab you will master workflow syntax, job orchestration, and parallel execution strategies.

> **Duration:** 5-10 minutes

## Learning Objectives

By the end of this lab, you will be able to:
- ✅ Create complex job dependency chains
- ✅ Use matrix strategies for parallel testing
- ✅ Implement concurrency controls
- ✅ Optimize workflow execution with fail-fast strategies
- ✅ Pass data between jobs using outputs

## References
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Using jobs in a workflow](https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow)
- [Using a matrix strategy](https://docs.github.com/en/actions/using-jobs/using-a-build-matrix-for-your-jobs)
- [Using concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency)

---

## 2.1 Create job dependencies and parallelization

Let's build a complex deployment pipeline with fan-out/fan-in patterns.

1. Open the workflow file [job-dependencies.yml](/.github/workflows/job-dependencies.yml)
2. Edit the file and add the following jobs at the end:

```yaml
  build:
    runs-on: ubuntu-latest  
    strategy:
      fail-fast: true
      matrix:
        configuration: [debug, release]
    steps:
      - run: echo "🔨 Building configuration ${{ matrix.configuration }}"
      - run: echo "✅ Build completed for ${{ matrix.configuration }}"
  
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "🧪 Running tests after build completes"
      - run: echo "✅ All tests passed"
  
  deploy-staging:
    runs-on: ubuntu-latest
    needs: test
    environment:
      name: staging
    steps:
      - run: echo "🚀 Deploying to staging environment"
  
  deploy-production:
    runs-on: ubuntu-latest
    needs: test
    environment:
      name: production
    steps:
      - run: echo "🚀 Deploying to production environment"
  
  notify:
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-production]
    if: always()
    steps:
      - run: echo "📢 Deployment pipeline completed"
      - run: echo "Staging status - ${{ needs.deploy-staging.result }}"
      - run: echo "Production status - ${{ needs.deploy-production.result }}"
```

3. Update the workflow triggers to include manual dispatch and push:

```yaml
on:
  workflow_dispatch:
  push:
    branches:
      - main
    paths:
      - '.github/workflows/job-dependencies.yml'
```

4. Commit the changes to `main` branch
5. Go to `Actions` and manually trigger the workflow using the `Run workflow` button
6. Observe how jobs execute in parallel and sequence based on `needs`

> 💡 **Understanding job orchestration:**
> - Jobs without `needs` run in parallel
> - `needs` creates dependencies - jobs wait for dependencies to complete
> - `if: always()` ensures a job runs even if dependencies fail
> - Use `needs.<job_id>.result` to check status of dependent jobs

---

## 2.2 Implement matrix strategies for comprehensive testing

Matrix strategies allow you to test across multiple configurations simultaneously.

1. Update the `build` job to test across multiple dimensions:

```yaml
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false  # Don't cancel other jobs if one fails
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        configuration: [debug, release]
        include:
          # Add specific configuration for Ubuntu release
          - os: ubuntu-latest
            configuration: release
            artifact-name: ubuntu-release-build
        exclude:
          # Skip Windows debug builds to save time
          - os: windows-latest
            configuration: debug
    
    steps:
      - run: echo "🔨 Building ${{ matrix.configuration }} on ${{ matrix.os }}"
      
      - name: Conditional step for release builds
        if: matrix.configuration == 'release'
        run: echo "📦 Creating release package"
      
      - name: Upload artifact for Ubuntu release
        if: matrix.artifact-name
        run: echo "⬆️ Uploading ${{ matrix.artifact-name }}"
```

> 🎯 **Matrix Strategy Best Practices:**
> - Use `fail-fast: false` for comprehensive test coverage
> - Use `include` to add specific job configurations
> - Use `exclude` to skip unnecessary combinations
> - Combine with conditional steps for flexibility

---

## 2.3 Add concurrency controls to prevent race conditions

Concurrency ensures only one workflow runs at a time for a specific group.

1. Add concurrency configuration at the workflow level (after the `on:` section):

```yaml
# Prevent multiple deployments from running simultaneously
concurrency:
  group: deployment-${{ github.ref }}
  cancel-in-progress: true
```

2. Commit and test by triggering the workflow twice quickly
3. Observe that the second run cancels the first (if still running)

> 🔒 **Concurrency Use Cases:**
> - Prevent multiple deployments to the same environment
> - Avoid race conditions when updating shared resources
> - Save runner minutes by canceling superseded runs
> - Group by branch: `group: ${{ github.workflow }}-${{ github.ref }}`

---

## 2.4 Pass data between jobs using outputs

Let's create a build job that passes information to downstream jobs.

1. Add a new job that creates outputs:

```yaml
  prepare:
    runs-on: ubuntu-latest
    outputs:
      build-id: ${{ steps.build-info.outputs.build-id }}
      build-time: ${{ steps.build-info.outputs.build-time }}
      should-deploy: ${{ steps.build-info.outputs.should-deploy }}
    steps:
      - name: Generate build information
        id: build-info
        run: |
          echo "build-id=build-${{ github.run_number }}-${{ github.run_attempt }}" >> $GITHUB_OUTPUT
          echo "build-time=$(date -u +'%Y-%m-%dT%H:%M:%SZ')" >> $GITHUB_OUTPUT
          echo "should-deploy=true" >> $GITHUB_OUTPUT
      
      - name: Display build info
        run: |
          echo "Build ID: ${{ steps.build-info.outputs.build-id }}"
          echo "Build Time: ${{ steps.build-info.outputs.build-time }}"
```

2. Update an existing job to use these outputs:

```yaml
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [test, prepare]
    if: needs.prepare.outputs.should-deploy == 'true'
    environment:
      name: staging
    steps:
      - run: echo "🚀 Deploying build ${{ needs.prepare.outputs.build-id }}"
      - run: echo "📅 Build time was ${{ needs.prepare.outputs.build-time }}"
```

> 📤 **Outputs Best Practice:**
> - Define outputs at job level with `outputs:`
> - Create outputs in steps using `>> $GITHUB_OUTPUT`
> - Access in dependent jobs: `needs.<job-id>.outputs.<output-name>`
> - Outputs are strings - use comparisons carefully

---

## 2.5 Complete Workflow Example

<details>
<summary>Click to see the complete updated workflow</summary>

```yaml
name: 02-2. Job Dependencies and Matrix

on:
  workflow_dispatch:
  push:
    branches:
      - main
    paths:
      - '.github/workflows/job-dependencies.yml'

# Prevent multiple deployments from running simultaneously
concurrency:
  group: deployment-${{ github.ref }}
  cancel-in-progress: true

jobs:
  initial:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎬 Starting workflow pipeline"
      - run: echo "This job runs first and triggers fanout pattern"
  
  fanout1:
    runs-on: ubuntu-latest
    needs: initial
    steps:
      - run: echo "⚡ Fanout job 1 running in parallel with fanout2"
  
  fanout2:
    runs-on: ubuntu-latest
    needs: initial
    steps:
      - run: echo "⚡ Fanout job 2 running in parallel with fanout1"
  
  fanin:
    runs-on: ubuntu-latest
    needs: [fanout1, fanout2]
    steps:
      - run: echo "🔀 Fanin job - waits for both fanout jobs"

  prepare:
    runs-on: ubuntu-latest
    outputs:
      build-id: ${{ steps.build-info.outputs.build-id }}
      build-time: ${{ steps.build-info.outputs.build-time }}
      should-deploy: ${{ steps.build-info.outputs.should-deploy }}
    steps:
      - name: Generate build information
        id: build-info
        run: |
          echo "build-id=build-${{ github.run_number }}-${{ github.run_attempt }}" >> $GITHUB_OUTPUT
          echo "build-time=$(date -u +'%Y-%m-%dT%H:%M:%SZ')" >> $GITHUB_OUTPUT
          echo "should-deploy=true" >> $GITHUB_OUTPUT

  build:
    runs-on: ${{ matrix.os }}
    needs: prepare
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        configuration: [debug, release]
        include:
          - os: ubuntu-latest
            configuration: release
            artifact-name: ubuntu-release-build
        exclude:
          - os: windows-latest
            configuration: debug
    steps:
      - run: echo "🔨 Building ${{ matrix.configuration }} on ${{ matrix.os }}"
      - run: echo "📦 Build ID - ${{ needs.prepare.outputs.build-id }}"
      
      - name: Conditional step for release builds
        if: matrix.configuration == 'release'
        run: echo "📦 Creating release package"
  
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "🧪 Running comprehensive test suite"
      - run: echo "✅ All tests passed"
  
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [test, prepare]
    if: needs.prepare.outputs.should-deploy == 'true'
    environment:
      name: staging
    steps:
      - run: echo "🚀 Deploying build ${{ needs.prepare.outputs.build-id }} to staging"
      - run: echo "📅 Build time was ${{ needs.prepare.outputs.build-time }}"
  
  deploy-production:
    runs-on: ubuntu-latest
    needs: [test, prepare]
    if: needs.prepare.outputs.should-deploy == 'true'
    environment:
      name: production
    steps:
      - run: echo "🚀 Deploying build ${{ needs.prepare.outputs.build-id }} to production"
  
  notify:
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-production]
    if: always()
    steps:
      - run: echo "📢 Deployment pipeline completed"
      - run: |
          echo "Staging: ${{ needs.deploy-staging.result }}"
          echo "Production: ${{ needs.deploy-production.result }}"
```
</details>

---

## 🎓 Key Takeaways

- **Job dependencies:** Use `needs` to orchestrate complex workflows
- **Parallelization:** Jobs without dependencies run in parallel automatically
- **Matrix strategies:** Test across multiple OS, language versions, or configurations
- **Concurrency control:** Prevent race conditions and save runner minutes
- **Job outputs:** Pass data between jobs for coordinated pipelines
- **Conditional execution:** Use `if` with `always()`, `success()`, `failure()`

## 📊 Workflow Visualization

```
initial
   ↓
[fanout1, fanout2] → fanin
   ↓
prepare → build (matrix) → test → [deploy-staging, deploy-production] → notify
```

## 💡 Pro Tips

1. **fail-fast: false** - Get complete test results across all matrix combinations
2. **if: always()** - Ensure cleanup/notification jobs always run
3. **concurrency groups** - Use meaningful group names for better control
4. **Job outputs** - Keep outputs small and simple (they're strings!)

---

## ✅ Lab Complete!

You've mastered workflow orchestration! You now understand:
- Complex job dependency patterns
- Matrix strategies for parallel execution
- Concurrency controls for safe deployments
- Passing data between jobs
- Conditional job execution

**Next:** Proceed to [Lab 3: Environments, Secrets & Security](/labs/lab03.md)
