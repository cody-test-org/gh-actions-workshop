# 7 - CI/CD Best Practices 🚀

In this lab you will implement production-ready CI/CD pipelines with caching, artifacts, testing, and deployment strategies.

> **Duration:** 15-20 minutes

## Learning Objectives

By the end of this lab, you will be able to:
- ✅ Implement efficient caching strategies
- ✅ Use artifacts for build outputs and test results
- ✅ Create comprehensive testing pipelines
- ✅ Implement safe deployment patterns
- ✅ Use concurrency for deployment protection
- ✅ Add security scanning to your pipeline

## References
- [About continuous integration](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration)
- [Using a build matrix](https://docs.github.com/en/actions/using-jobs/using-a-build-matrix-for-your-jobs)
- [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
- [Caching dependencies](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [Using concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency)

---

## 7.1 Implement efficient caching

Caching can dramatically speed up your workflows by reusing dependencies.

1. Open the workflow file [ci-workflow.yml](/.github/workflows/ci-workflow.yml)
2. Add caching to the build job:

```yaml
name: 07-1. CI Workflow with Caching

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  lint:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'  # Built-in npm cache
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint || echo "No lint script found"

  test:
    name: Test Suite
    runs-on: ${{ matrix.os }}
    needs: lint
    
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20]
        exclude:
          # Skip Windows Node 18 to save time
          - os: windows-latest
            node-version: 18
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      # Advanced caching with custom key
      - name: Cache test results
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            coverage/
            .cache/
          key: ${{ runner.os }}-node-${{ matrix.node-version }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-${{ matrix.node-version }}-
            ${{ runner.os }}-node-
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test || echo "No test script found"
        env:
          CI: true
      
      - name: Generate coverage report
        if: matrix.os == 'ubuntu-latest' && matrix.node-version == '20'
        run: |
          echo "# Test Coverage Report" >> $GITHUB_STEP_SUMMARY
          echo "✅ Tests completed on ${{ matrix.os }}" >> $GITHUB_STEP_SUMMARY
```

> 💡 **Caching Best Practices:**
> - Use `hashFiles()` for cache keys to invalidate when dependencies change
> - Use `restore-keys` for fallback caches
> - Cache node_modules, pip packages, Maven/Gradle dependencies
> - Built-in cache options: `setup-node`, `setup-python`, `setup-java` have `cache:` parameter

---

## 7.2 Use artifacts to share data between jobs

Artifacts allow you to share build outputs, test results, and logs between jobs.

1. Update the test job to upload artifacts:

```yaml
  test:
    name: Test Suite
    runs-on: ubuntu-latest
    needs: lint
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm test || echo "No tests found"
      
      - name: Generate test artifacts
        run: |
          mkdir -p test-results
          echo "Test completed at $(date)" > test-results/summary.txt
          echo '{"tests": 42, "passed": 40, "failed": 2}' > test-results/results.json
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results-${{ github.run_number }}
          path: test-results/
          retention-days: 30
      
      - name: Upload logs
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: failure-logs-${{ github.run_number }}
          path: |
            **/*.log
            coverage/
          retention-days: 7

  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build || echo "Creating mock build"
      
      - name: Create build artifact
        run: |
          mkdir -p dist
          echo "Build $(git rev-parse --short HEAD)" > dist/version.txt
          echo "<h1>Hello from Build ${{ github.run_number }}</h1>" > dist/index.html
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: application-build-${{ github.run_number }}
          path: dist/
          retention-days: 90
```

---

## 7.3 Create a comprehensive deployment pipeline

1. Add deployment jobs that use the build artifact:

```yaml
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: application-build-${{ github.run_number }}
          path: ./dist
      
      - name: Display artifact contents
        run: |
          echo "📦 Downloaded artifact contents:"
          ls -lah dist/
          cat dist/version.txt
      
      - name: Deploy to staging
        run: |
          echo "🚀 Deploying to staging environment"
          echo "Simulating deployment of build from dist/"
          # In real scenario: 
          # aws s3 sync dist/ s3://staging-bucket/
          # or similar deployment command
      
      - name: Run smoke tests
        run: |
          echo "🧪 Running smoke tests on staging"
          echo "✅ Health check passed"
          echo "✅ API endpoints responding"
      
      - name: Create deployment summary
        run: |
          echo "## Staging Deployment 🎯" >> $GITHUB_STEP_SUMMARY
          echo "- **Build:** ${{ github.run_number }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Commit:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
          echo "- **URL:** https://staging.example.com" >> $GITHUB_STEP_SUMMARY
          echo "- **Status:** ✅ Deployed" >> $GITHUB_STEP_SUMMARY

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    # Prevent concurrent production deployments
    concurrency:
      group: production-deployment
      cancel-in-progress: false
    
    environment:
      name: production
      url: https://example.com
    
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: application-build-${{ github.run_number }}
          path: ./dist
      
      - name: Deploy to production
        run: |
          echo "🚀 Deploying to PRODUCTION"
          echo "Build version: $(cat dist/version.txt)"
      
      - name: Verify deployment
        run: |
          echo "🔍 Verifying production deployment"
          echo "✅ All systems operational"
      
      - name: Notify team
        run: |
          echo "## 🎉 Production Deployment Complete" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "- **Version:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Deployed by:** @${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Build Number:** ${{ github.run_number }}" >> $GITHUB_STEP_SUMMARY
          echo "- **URL:** https://example.com" >> $GITHUB_STEP_SUMMARY
```

---

## 7.4 Add security scanning

Let's add security scanning to the pipeline:

```yaml
  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Check for critical vulnerabilities
        run: |
          echo "🔒 Security scan completed"
          echo "Check the Security tab for detailed results"
```

---

## 7.5 Update the CD workflow with modern practices

1. Open [cd-workflow.yml](/.github/workflows/cd-workflow.yml)
2. Update it with modern patterns:

```yaml
name: 07-2. CD Workflow

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  NODE_VERSION: '20'

# Prevent multiple production deployments
concurrency:
  group: cd-${{ github.ref }}
  cancel-in-progress: false

jobs:
  build:
    name: Build and Package
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for versioning
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci --prefer-offline
      
      - name: Run tests
        run: npm test || echo "No tests configured"
      
      - name: Build application
        run: npm run build || mkdir -p dist && echo "Build mock"
      
      - name: Generate version
        id: version
        run: |
          VERSION="v1.0.${{ github.run_number }}"
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "$VERSION" > dist/VERSION
          echo "📦 Version: $VERSION"
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-${{ steps.version.outputs.version }}
          path: dist/
          retention-days: 30
          compression-level: 9

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    
    environment:
      name: production
      url: https://production.example.com
    
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: app-${{ needs.build.outputs.version }}
          path: ./dist
      
      - name: Display deployment info
        run: |
          echo "Deploying version: ${{ needs.build.outputs.version }}"
          ls -lah dist/
      
      - name: Deploy to production (simulate)
        run: |
          echo "🚀 Deploying version ${{ needs.build.outputs.version }}"
          # Real deployment example:
          # aws s3 sync dist/ s3://prod-bucket/ --delete
          # aws cloudfront create-invalidation --distribution-id XXX --paths "/*"
      
      - name: Health check
        run: |
          echo "🏥 Running health checks"
          sleep 2
          echo "✅ Application is healthy"
      
      - name: Create release
        run: |
          echo "## 🚀 Release ${{ needs.build.outputs.version }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### Deployment Details" >> $GITHUB_STEP_SUMMARY
          echo "- **Version:** ${{ needs.build.outputs.version }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Commit:** \`${{ github.sha }}\`" >> $GITHUB_STEP_SUMMARY
          echo "- **Deployed by:** @${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Environment:** Production" >> $GITHUB_STEP_SUMMARY
          echo "- **URL:** https://production.example.com" >> $GITHUB_STEP_SUMMARY
```

---

## 7.6 Implement rollback capability

Add a workflow for emergency rollbacks:

Create a new file `.github/workflows/rollback.yml`:

```yaml
name: 🔄 Rollback Deployment

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to rollback to (e.g., v1.0.42)'
        required: true
        type: string
      environment:
        description: 'Environment to rollback'
        required: true
        type: choice
        options:
          - staging
          - production

jobs:
  rollback:
    name: Rollback to ${{ inputs.version }}
    runs-on: ubuntu-latest
    
    environment:
      name: ${{ inputs.environment }}
    
    steps:
      - name: Validate version
        run: |
          echo "🔄 Rolling back to version: ${{ inputs.version }}"
          echo "Environment: ${{ inputs.environment }}"
      
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: app-${{ inputs.version }}
          path: ./dist
        continue-on-error: true
      
      - name: Perform rollback
        run: |
          echo "⏮️ Performing rollback to ${{ inputs.version }}"
          # Deployment commands here
          echo "✅ Rollback completed"
      
      - name: Verify rollback
        run: |
          echo "🔍 Verifying rollback"
          echo "✅ Application is healthy on previous version"
      
      - name: Notify team
        run: |
          echo "## ⚠️ Rollback Executed" >> $GITHUB_STEP_SUMMARY
          echo "- **Version:** ${{ inputs.version }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Environment:** ${{ inputs.environment }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Triggered by:** @${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
```

---

## 7.7 Complete examples

<details>
<summary>Complete CI Workflow</summary>

```yaml
name: 07-1. CI Workflow

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint || echo "No lint script"

  test:
    runs-on: ${{ matrix.os }}
    needs: lint
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test || echo "No tests"

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build || mkdir dist
      - uses: actions/upload-artifact@v4
        with:
          name: build-${{ github.run_number }}
          path: dist/

  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ github.run_number }}
      - run: echo "Deploy to staging"
```
</details>

---

## 🎓 Key Takeaways

- **Caching:** Speeds up builds by reusing dependencies
- **Artifacts:** Share data between jobs and store build outputs
- **Matrix builds:** Test across multiple configurations
- **Concurrency:** Prevent race conditions in deployments
- **Security scanning:** Integrate vulnerability detection
- **Job summaries:** Provide rich deployment feedback
- **Rollback workflows:** Enable quick recovery from issues

## 📊 CI/CD Pipeline Flow

```
Push to main
    ↓
Lint & Security Scan (parallel)
    ↓
Test (matrix: OS × Node versions)
    ↓
Build → Upload Artifact
    ↓
Deploy Staging (download artifact)
    ↓
Smoke Tests
    ↓
Deploy Production (manual approval + download artifact)
    ↓
Health Checks & Monitoring
```

## 💡 Production Best Practices

1. **Always cache dependencies:** Save minutes on every build
2. **Use artifacts for deployments:** Deploy exactly what you built/tested
3. **Implement proper artifact retention:** Balance storage costs vs. rollback needs
4. **Add security scanning:** Catch vulnerabilities early
5. **Use concurrency for production:** Prevent deployment conflicts
6. **Create rich job summaries:** Make deployment tracking easy
7. **Enable easy rollbacks:** Things go wrong - be prepared
8. **Monitor everything:** Logs, metrics, traces, alerts

## 🔐 Security Checklist

- ✅ Scan dependencies for vulnerabilities
- ✅ Use least-privilege permissions
- ✅ Never commit secrets
- ✅ Use OIDC for cloud authentication
- ✅ Scan container images
- ✅ Enable branch protection
- ✅ Require code reviews
- ✅ Use signed commits (optional)

---

## ✅ Lab Complete!

Congratulations! You've completed all 7 labs and now have comprehensive knowledge of GitHub Actions!

You've mastered:
- ✅ Workflow fundamentals and modern syntax
- ✅ Job dependencies and matrix strategies
- ✅ Secure secret management and environments
- ✅ Reusable workflows for DRY principles
- ✅ Custom actions for specialized needs
- ✅ Self-hosted runners and scalability
- ✅ Production-ready CI/CD pipelines

## 🎉 Next Steps

- Implement these patterns in your own projects
- Explore the [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- Join the [GitHub Community](https://github.community/)
- Read the [GitHub Blog](https://github.blog/) for latest updates
- Consider [GitHub Advanced Security](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security)

## 📚 Continue Learning

- [Microsoft Learn: Automate your workflow with GitHub Actions](https://learn.microsoft.com/training/paths/automate-workflow-github-actions/)
- [GitHub Skills](https://skills.github.com/)
- [Actions Documentation](https://docs.github.com/actions)

---

**Thank you for completing this workshop! Happy automating! 🚀**
