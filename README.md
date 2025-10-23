# GitHub Actions Workshop
> Comprehensive GitHub Actions training with modern workflow examples, hands-on labs, and best practices for CI/CD automation.

## 🎯 Workshop Overview

This workshop provides hands-on experience with GitHub Actions, from basic concepts to advanced CI/CD patterns. Whether you're new to GitHub Actions or familiar with other CI/CD tools, these labs will help you master automated workflows.

**Total Duration:** ~60 minutes of hands-on labs  
**Prerequisites:** GitHub account, basic Git knowledge, familiarity with YAML syntax

## 📚 Examples & Hands-on Labs

### 🚀 Getting Started
- [ ] **[Lab Setup](/labs/setup.md)** _(5-10 min)_ - Fork repository and enable GitHub Actions

### Module 1: Introduction to GitHub Actions _(5-10 min)_
Learn the fundamentals of GitHub Actions workflows, triggers, and job summaries.

- Example: [github-actions-demo.yml](/.github/workflows/github-actions-demo.yml) - Basic workflow structure
- Example: [greet-everyone.yml](/.github/workflows/greet-everyone.yml) - Simple automation
- [ ] **Hands-on Lab:** [Activity 1](/labs/lab01.md) - Create your first workflow with modern syntax

### Module 2: Workflow Syntax & Job Dependencies _(5-10 min)_
Master workflow syntax, job orchestration, and matrix strategies for parallel execution.

- Example: [simple-workflow.yml](/.github/workflows/simple-workflow.yml) - Clean workflow syntax
- Example: [job-dependencies.yml](/.github/workflows/job-dependencies.yml) - Complex job dependencies
- [ ] **Hands-on Lab:** [Activity 2](/labs/lab02.md) - Build workflows with dependencies and matrix builds

### Module 3: Environments, Secrets & Security _(10-15 min)_
Implement secure deployment workflows with environment protection and secret management.

- Example: [environments-secrets.yml](/.github/workflows/environments-secrets.yml) - Multi-environment deployments
- [ ] **Hands-on Lab:** [Activity 3](/labs/lab03.md) - Configure environments with approvals and secrets

### Module 4: Reusable Workflows _(10-15 min)_
Create modular, reusable workflows to reduce duplication and standardize processes.

- Example: [reusable-workflow-template.yml](/.github/workflows/reusable-workflow-template.yml) - Calling reusable workflows
- Example: [super-linter.yml](/.github/workflows/super-linter.yml) - Code quality automation
- [ ] **Hands-on Lab:** [Activity 4](/labs/lab04.md) - Build and consume reusable workflows

### Module 5: Custom Actions _(10-15 min)_
Develop custom actions using JavaScript, TypeScript, and Docker for specialized automation.

- Example: [github-script.yml](/.github/workflows/github-script.yml) - Inline GitHub API automation
- Example: [hello-world-composite.yml](/.github/workflows/hello-world-composite.yml) - Composite actions
- Example: [use-custom-actions.yml](/.github/workflows/use-custom-actions.yml) - Custom action usage
- Reference: [githubabcs/hello-world-composite-action](https://github.com/githubabcs/hello-world-composite-action)
- [ ] **Hands-on Lab:** [Activity 5](/labs/lab05.md) - Create and use custom actions

### Module 6: Self-hosted Runners & Scalability _(5-10 min)_
Deploy and manage self-hosted runners for specialized workloads and cost optimization.

- Example: [self-hosted-linux.yml](/.github/workflows/self-hosted-linux.yml) - Using self-hosted runners
- [ ] **Hands-on Lab:** [Activity 6](/labs/lab06.md) - Understand self-hosted runner patterns

### Module 7: CI/CD Best Practices _(15-20 min)_
Build production-ready CI/CD pipelines with caching, artifacts, testing, and deployment strategies.

- Example: [ci-workflow.yml](/.github/workflows/ci-workflow.yml) - Continuous Integration
- Example: [cd-workflow.yml](/.github/workflows/cd-workflow.yml) - Continuous Deployment
- [ ] **Hands-on Lab:** [Activity 7](/labs/lab07.md) - Implement complete CI/CD pipeline

---

## Learning Path
- [Automate your workflow with GitHub Actions](https://learn.microsoft.com/en-us/training/paths/automate-workflow-github-actions/)
- [Manage GitHub Actions in the enterprise](https://learn.microsoft.com/en-us/training/modules/manage-github-actions-enterprise/)

---

## Additional Resources
> Additional resources to continue your GitHub Actions learning journey.

### Training Manual
- [GitHub Actions Training Manual](https://githubtraining.github.io/actions-facilitator-guide/#/)

### GitHub Skills
- [GitHub Skills](https://github.com/skills)
- [GitHub Actions: Continuous Delivery with Azure](https://github.com/skills/continuous-delivery-azure)
- [Publish to GitHub Packages](https://github.com/skills/publish-packages)

### GitHub Actions Documentation
- [GitHub Actions](https://docs.github.com/en/actions)
- [GitHub Actions guides](https://docs.github.com/en/actions/guides)
- [Authenticating with GitHub Apps](https://docs.github.com/en/developers/apps/building-github-apps/authenticating-with-github-apps#generating-a-private-key)
- [Starter Workflows](https://github.com/actions/starter-workflows)
- [GitHub Actions Toolkit](https://github.com/actions/toolkit)
- [Reusing workflows](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/reusing-workflows)
- [How to start using reusable workflows with GitHub Actions](https://github.blog/2022-02-10-using-reusable-workflows-github-actions/)
- [Required workflows](https://docs.github.com/en/actions/using-workflows/required-workflows)
- [Defining configuration variables for multiple workflows](https://docs.github.com/en/actions/learn-github-actions/variables#defining-configuration-variables-for-multiple-workflows)
- [Creating Actions](https://docs.github.com/en/actions/creating-actions)
- [Security guides](https://docs.github.com/en/actions/security-guides)
- [GitHub Blog](https://github.blog/)
- [Awesome Actions](https://github.com/sdras/awesome-actions)

### GitHub self-hosted runners on Kubernetes
- [Autoscaling with self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/autoscaling-with-self-hosted-runners)
- [Managing self-hosted runners with Actions Runner Controller](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller)
- [actions-runner-controller - A Kubernetes controller for GitHub Actions self-hosted runners.](https://github.com/actions/actions-runner-controller)
- [GitHub Self-Hosted Runner Autoscaling with Kubernetes](https://tgrall.github.io/blog/2022/10/16/github-self-hosted-runner-autoscaling-with-kubernetes)
- [GitHub Actions: Dive into actions-runner-controller (ARC)](https://www.youtube.com/watch?v=_F5ocPrv6io)

### Actions Changelog
- [Changelog](https://github.blog/changelog/label/actions/)
- [GitHub Actions - Sharing actions and reusable workflows from private repositories is now GA](https://github.blog/changelog/2022-12-14-github-actions-sharing-actions-and-reusable-workflows-from-private-repositories-is-now-ga/)
- [GitHub Actions: Restrict workflows to specific runners using runner group names](https://github.blog/changelog/2022-11-01-github-actions-restrict-workflows-to-specific-runners-using-runner-group-names/)
- [GitHub Actions: Use the GITHUB_TOKEN with workflow_dispatch and repository_dispatch](https://github.blog/changelog/2022-09-08-github-actions-use-github_token-with-workflow_dispatch-and-repository_dispatch/)
- [GitHub Actions: Improvements to reusable workflows](https://github.blog/changelog/2022-08-22-github-actions-improvements-to-reusable-workflows-2/)
- [GitHub Actions: Enhance your actions with job summaries](https://github.blog/changelog/2022-05-09-github-actions-enhance-your-actions-with-job-summaries/)
- [Reusable workflows can be referenced locally](https://github.blog/changelog/2022-01-25-github-actions-reusable-workflows-can-be-referenced-locally/)
- [Share GitHub Actions within your enterprise](https://github.blog/changelog/2022-01-21-share-github-actions-within-your-enterprise/)
- [GitHub Actions: Prevent GitHub Actions from approving pull requests](https://github.blog/changelog/2022-01-14-github-actions-prevent-github-actions-from-approving-pull-requests/)
- [Reduce duplication with action composition](https://github.blog/changelog/2021-08-25-github-actions-reduce-duplication-with-action-composition/)
- [GitHub Actions: Control permissions for GITHUB_TOKEN](https://github.blog/changelog/2021-04-20-github-actions-control-permissions-for-github_token/)

### Articles
- [GitHub Actions & Security: Best practices](https://devopsjournal.io/blog/2021/02/06/GitHub-Actions)
- [Setup an internal GitHub Actions Marketplace](https://devopsjournal.io/blog/2021/10/14/GitHub-Actions-Internal-Marketplace)
- [How to build a CI/CD pipeline with GitHub Actions in four simple steps](https://github.blog/2022-02-02-build-ci-cd-pipeline-github-actions-four-steps/)
- [Supercharging GitHub Actions with Job Summaries](https://github.blog/2022-05-09-supercharging-github-actions-with-job-summaries/)
- [Keeping your actions up to date with Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot)
- [GitHub Actions for security and compliance](https://github.blog/2021-10-22-github-actions-for-security-compliance/)
- [Triggering a workflow from a workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow)
- [Introducing required workflows and configuration variables to GitHub Actions](https://github.blog/2023-01-10-introducing-required-workflows-and-configuration-variables-to-github-actions/)
