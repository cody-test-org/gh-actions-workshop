# 🚀 Hands-on Labs Setup
In this lab you will fork the repository and enable GitHub Actions to begin your learning journey.

> **Duration:** 5-10 minutes

## Prerequisites

Before starting, ensure you have:
- ✅ A GitHub account (create one at [github.com](https://github.com))
- ✅ Basic familiarity with Git and GitHub
- ✅ A web browser (Chrome, Firefox, Safari, or Edge)
- ✅ Basic understanding of YAML syntax (helpful but not required)

## References
- [Fork a repository](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
- [Quickstart for GitHub Actions](https://docs.github.com/en/actions/quickstart)
- [About workflows](https://docs.github.com/en/actions/using-workflows/about-workflows)

## Step 1: Fork the Repository

1. Navigate to the repository: [gh-actions-workshop](https://github.com/cody-test-org/gh-actions-workshop)
2. Click the **Fork** button in the top-right corner
3. Select your account as the destination
4. **Important:** Keep the repository name or choose a memorable name
5. Click **Create fork**

> 💡 **Tip:** Forking creates your own copy where you can experiment freely without affecting the original repository.

## Step 2: Enable GitHub Actions

After forking, you need to enable workflows:

1. Navigate to the **Actions** tab in your forked repository
2. You'll see a warning message:
   > *Workflows aren't being run on this forked repository*
3. Click the green button: **I understand my workflows, go ahead and enable them**
4. You should now see the list of all available workflows

> 🔒 **Security Note:** GitHub disables workflows in forks by default to prevent malicious code execution. Always review workflows before enabling them.

## Step 3: Verify Your Setup

Let's verify everything is working:

1. Go to **Actions** tab
2. Click on any workflow from the list (e.g., "01-1. GitHub Actions Demo")
3. Click **Run workflow** dropdown
4. Click the green **Run workflow** button
5. Wait a few seconds and refresh - you should see your workflow running!

> ✅ **Success:** If you see a yellow dot (in progress) or green checkmark (completed), your setup is working!

## Step 4: (Optional) Track Your Progress

Create an issue to track your learning progress:

1. Go to the **Issues** tab in your repository
2. Click **New issue**
3. Use this template:

**Title:** `🎓 Training Progress - GitHub Actions Workshop`

**Body:**
```markdown
## Hands-on Labs & Activities Progress

### Setup
- [x] Fork repository
- [x] Enable GitHub Actions
- [x] Verify workflows

### Modules
- [ ] Module 1: Introduction to GitHub Actions (5-10 min)
- [ ] Module 2: Workflow Syntax & Job Dependencies (5-10 min)
- [ ] Module 3: Environments, Secrets & Security (10-15 min)
- [ ] Module 4: Reusable Workflows (10-15 min)
- [ ] Module 5: Custom Actions (10-15 min)
- [ ] Module 6: Self-hosted Runners (5-10 min)
- [ ] Module 7: CI/CD Best Practices (15-20 min)

---
**Total Estimated Time:** ~60 minutes
**Started:** [DATE]
```

4. Click **Submit new issue**

## Step 5: Stay Up to Date (Optional)

If the original repository gets updates, you can sync your fork:

1. Go to your fork on GitHub
2. Click **Sync fork** button (appears when upstream has changes)
3. Click **Update branch**

Alternatively, using Git CLI:
```bash
# Add upstream remote (one time)
git remote add upstream https://github.com/cody-test-org/gh-actions-workshop.git

# Fetch and merge updates
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

## 🎉 You're Ready!

Your environment is now set up. Proceed to [Lab 1: Introduction to GitHub Actions](/labs/lab01.md) to start learning!

## Troubleshooting

**Problem:** "I don't see the Actions tab"  
**Solution:** Actions may be disabled. Go to Settings → Actions → General, and ensure "Allow all actions and reusable workflows" is selected.

**Problem:** "Workflows won't run"  
**Solution:** Check that you clicked "I understand my workflows, go ahead and enable them" in the Actions tab.

**Problem:** "I can't fork the repository"  
**Solution:** Ensure you're logged into GitHub and have permissions to create repositories in your account.
