# 6 - Self-hosted Runners & Scalability 🖥️

In this lab you will learn about self-hosted runners, when to use them, and modern deployment patterns.

> **Duration:** 5-10 minutes

## Learning Objectives

By the end of this lab, you will be able to:
- ✅ Understand when to use self-hosted vs GitHub-hosted runners
- ✅ Know the security considerations for self-hosted runners
- ✅ Learn about runner groups and access controls
- ✅ Understand Actions Runner Controller (ARC) for Kubernetes
- ✅ Explore GitHub-hosted larger runners

## References
- [About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)
- [Adding self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners)
- [Using self-hosted runners in a workflow](https://docs.github.com/en/actions/hosting-your-own-runners/using-self-hosted-runners-in-a-workflow)
- [Actions Runner Controller](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller)
- [GitHub-hosted larger runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-larger-runners)

---

## 6.1 GitHub-hosted vs Self-hosted Runners

### GitHub-hosted Runners ☁️

**Pros:**
- ✅ Zero maintenance
- ✅ Automatic updates
- ✅ Isolated environment (security)
- ✅ Fast provisioning
- ✅ Multiple OS options
- ✅ Free for public repos

**Cons:**
- ❌ Fixed resources (2-4 cores, 7-14GB RAM)
- ❌ No persistent state
- ❌ No access to private networks
- ❌ Can't install custom software globally

**Best for:**
- Public repositories
- Standard build/test workloads
- Teams without infrastructure
- Security-sensitive workflows

### Self-hosted Runners 🏠

**Pros:**
- ✅ Custom hardware (GPUs, more RAM/CPU)
- ✅ Access to private networks/databases
- ✅ Persistent cache between runs
- ✅ Custom software pre-installed
- ✅ Cost control for high usage

**Cons:**
- ❌ You maintain and secure them
- ❌ You update them
- ❌ Security considerations
- ❌ Infrastructure costs

**Best for:**
- Private networks access needed
- Specialized hardware requirements
- Very high usage (cost optimization)
- Persistent local cache needs
- Specific compliance requirements

---

## 6.2 Security considerations for self-hosted runners

> ⚠️ **Critical Security Warning:** Self-hosted runners should **NEVER** be used with public repositories!

### Why Public Repos are Dangerous:

1. **Arbitrary Code Execution:** Anyone can fork your repo, modify workflows, and execute code on your runner
2. **Secret Exfiltration:** Malicious PRs could steal secrets and credentials
3. **Resource Abuse:** Cryptomining, DDoS attacks, etc.
4. **Network Access:** Attack your internal infrastructure

### Security Best Practices:

✅ **DO:**
- Use self-hosted runners only for private repositories
- Implement runner groups with restricted access
- Use just-in-time (ephemeral) runners that are destroyed after each job
- Run runners in isolated networks/VMs
- Monitor runner activity and logs
- Keep runners updated
- Use minimal permissions
- Implement network segmentation

❌ **DON'T:**
- Use self-hosted runners for public repositories
- Store secrets on the runner filesystem
- Give runners broad network access
- Reuse runner environments without cleanup
- Run as root/administrator

---

## 6.3 Setting up self-hosted runners (Conceptual)

### Manual Setup Process:

1. **Navigate to Settings:**
   - Organization: Settings → Actions → Runners
   - Repository: Settings → Actions → Runners

2. **Add New Runner:**
   - Click "New self-hosted runner"
   - Choose OS (Linux, Windows, macOS)
   - Follow the provided commands

3. **Example Installation (Linux):**

```bash
# Create a folder
mkdir actions-runner && cd actions-runner

# Download the latest runner package
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# Extract the installer
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configure
./config.sh --url https://github.com/org/repo --token YOUR_TOKEN

# Install and start service (Linux)
sudo ./svc.sh install
sudo ./svc.sh start
```

4. **Label Your Runners:**
   - Add custom labels for targeting
   - Examples: `gpu`, `high-memory`, `production-deploy`

---

## 6.4 Using self-hosted runners in workflows

### Basic Usage:

```yaml
name: Use Self-hosted Runner

on: [push]

jobs:
  build:
    # Use default labels
    runs-on: self-hosted
    
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running on self-hosted runner"
```

### Using Custom Labels:

```yaml
jobs:
  gpu-training:
    # Match multiple labels (AND logic)
    runs-on: [self-hosted, linux, x64, gpu]
    
    steps:
      - uses: actions/checkout@v4
      - run: nvidia-smi  # GPU available!
      - run: python train_model.py

  high-memory-job:
    runs-on: [self-hosted, linux, high-memory]
    
    steps:
      - run: java -Xmx32g -jar large-app.jar
```

### Runner Groups (Enterprise):

```yaml
jobs:
  production-deploy:
    runs-on: 
      group: production-runners  # Only use runners in this group
      labels: [self-hosted, linux, deploy]
    
    environment: production
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy-to-prod.sh
```

---

## 6.5 Actions Runner Controller (ARC) for Kubernetes

ARC enables auto-scaling self-hosted runners on Kubernetes, combining the best of both worlds.

### Benefits:

- ✅ **Auto-scaling:** Scale runners based on demand
- ✅ **Ephemeral runners:** Fresh environment for each job (security!)
- ✅ **Cost efficient:** Only run runners when needed
- ✅ **K8s native:** Leverage existing Kubernetes infrastructure
- ✅ **Multi-tenant:** Support multiple repos/organizations

### Architecture:

```
GitHub Actions Workflow
        ↓
   GitHub API
        ↓
ARC Controller (in K8s)
        ↓
    Runner Pods
        ↓
  Your Workflow Jobs
```

### Example ARC Setup:

```yaml
# Install ARC using Helm
helm install arc \
  --namespace arc-systems \
  --create-namespace \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller

# Create Runner Scale Set
helm install arc-runner-set \
  --namespace arc-runners \
  --create-namespace \
  --set githubConfigUrl="https://github.com/org/repo" \
  --set githubConfigSecret.github_token="YOUR_PAT" \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

### Use ARC Runners:

```yaml
jobs:
  build:
    runs-on: arc-runner-set  # Name of your runner scale set
    
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running on ephemeral Kubernetes runner!"
```

> 💡 **ARC Best Practice:** Perfect for enterprise environments with Kubernetes. Each job gets a fresh, isolated pod.

---

## 6.6 GitHub-hosted Larger Runners (Team/Enterprise)

For teams that need more power without managing infrastructure:

### Available Sizes:

| Size | Cores | RAM | Storage |
|------|-------|-----|---------|
| Standard | 2 | 7 GB | 14 GB |
| Large | 4 | 16 GB | 14 GB |
| X-Large | 8 | 32 GB | 14 GB |
| 2X-Large | 16 | 64 GB | 14 GB |
| 3X-Large | 32 | 128 GB | 14 GB |
| 4X-Large | 64 | 256 GB | 14 GB |

### Features:

- ✅ Static IP addresses
- ✅ Faster startup times
- ✅ More concurrent jobs
- ✅ macOS and Windows available
- ✅ Still fully managed by GitHub

### Using Larger Runners:

```yaml
jobs:
  heavy-build:
    runs-on: ubuntu-latest-8-cores  # 8-core runner
    
    steps:
      - uses: actions/checkout@v4
      - run: npm run build  # Much faster!
```

> 💰 **Cost Consideration:** Larger runners cost more per minute, but can reduce total job time and improve developer experience.

---

## 6.7 Example workflow for self-hosted runners

Here's an example using different runner types:

```yaml
name: 06-1. Multi-Runner Workflow

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      use-self-hosted:
        description: 'Use self-hosted runner'
        required: false
        type: boolean
        default: false

jobs:
  # Run on GitHub-hosted (always available)
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test

  # Conditionally use self-hosted for deployment
  deploy:
    runs-on: ${{ inputs.use-self-hosted && 'self-hosted' || 'ubuntu-latest' }}
    needs: lint-and-test
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Check runner type
        run: |
          echo "## Runner Information 🖥️" >> $GITHUB_STEP_SUMMARY
          echo "- **Runner Name:** ${{ runner.name }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Runner OS:** ${{ runner.os }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Runner Arch:** ${{ runner.arch }}" >> $GITHUB_STEP_SUMMARY
          
      - name: Deploy application
        run: |
          echo "Deploying from ${{ runner.name }}"
          # Your deployment commands
```

---

## 6.8 Monitoring and maintenance

### Key Metrics to Monitor:

1. **Runner Health:**
   - Online/Offline status
   - CPU and memory usage
   - Disk space available

2. **Job Metrics:**
   - Queue times
   - Job duration
   - Success/failure rates

3. **Security:**
   - Failed authentication attempts
   - Unusual network activity
   - Unauthorized access attempts

### Maintenance Tasks:

- 📅 **Weekly:** Check runner status and logs
- 📅 **Monthly:** Update runner software
- 📅 **Quarterly:** Review runner usage and costs
- 📅 **Annually:** Rotate authentication tokens

---

## 🎓 Key Takeaways

- **GitHub-hosted:** Easy, secure, sufficient for most use cases
- **Self-hosted:** Needed for custom hardware, private networks, or high usage
- **Security first:** Never use self-hosted runners with public repos
- **ARC:** Best practice for Kubernetes environments
- **Larger runners:** Good middle ground - more power, still managed
- **Ephemeral runners:** Most secure self-hosted approach

## 📊 Decision Tree

```
Need specialized hardware (GPU, etc.)? 
    ├─ Yes → Self-hosted
    └─ No ↓
    
Need private network access?
    ├─ Yes → Self-hosted
    └─ No ↓
    
Need more power than 4 cores/16GB?
    ├─ Yes (Enterprise) → Larger runners
    └─ No ↓
    
Use GitHub-hosted (standard) ✅
```

## 💡 Pro Tips

1. **Start with GitHub-hosted:** Don't self-host until you have a clear need
2. **Use runner groups:** Organize and control access to runners
3. **Ephemeral is better:** Destroy runners after each job for security
4. **Monitor costs:** Track usage to optimize runner allocation
5. **Document your runners:** Label them clearly and document their purpose

---

## ✅ Lab Complete!

You've mastered runner strategies! You now understand:
- When to use self-hosted vs GitHub-hosted runners
- Critical security considerations
- Actions Runner Controller for Kubernetes
- GitHub-hosted larger runners
- Best practices for runner management

**Next:** Proceed to [Lab 7: CI/CD Best Practices](/labs/lab07.md)
