# 1 - Introduction to GitHub Actions 🎬

In this lab you will create and run your first workflow with modern GitHub Actions syntax.

> **Duration:** 5-10 minutes

## Learning Objectives

By the end of this lab, you will be able to:
- ✅ Understand workflow triggers and events
- ✅ Use modern GITHUB_OUTPUT syntax
- ✅ Create job summaries with markdown
- ✅ Work with context variables and expressions
- ✅ Pin actions to specific versions for security

## References
- [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Context information](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [Job summaries](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary)

---

## 1.1 Update workflow to trigger on specific events

Let's configure a workflow to run when changes are made to the labs folder.

1. Open the workflow file [github-actions-demo.yml](/.github/workflows/github-actions-demo.yml)
2. Edit the file and update the `on:` section to trigger on push events to the `main` branch when files in the `labs/` folder change:

```yaml
name: 01-1. GitHub Actions Demo
on: 
  workflow_dispatch:
  workflow_call:
  push:
    branches:
      - main
    paths:
      - 'labs/**'
      - '.github/workflows/github-actions-demo.yml'
```

3. Commit the changes to the `main` branch

> 💡 **Understanding triggers:**
> - `workflow_dispatch`: Allows manual triggering via UI
> - `workflow_call`: Allows the workflow to be called from other workflows
> - `push`: Triggers on code pushes
> - `paths`: Filters which files trigger the workflow (saves runner minutes!)

4. Test the workflow by making a change to any file in the `labs/` folder
5. Commit the change to trigger the workflow
6. Navigate to `Actions` tab and observe your running workflow

> ✅ **Success Check:** You should see a new workflow run appear after your commit!

---

## 1.2 Add modern action steps with pinned versions

Let's add steps using marketplace actions with proper version pinning for security.

1. Open the workflow file [github-actions-demo.yml](/.github/workflows/github-actions-demo.yml)
2. Add the following steps at the end of the `steps:` section (after the existing steps):

```yaml
      # Using local custom action from .github/actions/hello-world-js
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Hello world action
        uses: ./.github/actions/hello-world-js
        with:
          who-to-greet: "Mona the Octocat"
        id: hello
      
      # Access outputs from previous step
      - name: Echo the greeting's time
        run: echo 'The time was ${{ steps.hello.outputs.time }}'
      
      # Create a rich job summary with markdown
      - name: Add job summary
        run: |
          echo "## Workflow Execution Summary 🚀" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "✅ **Job Status:** ${{ job.status }}" >> $GITHUB_STEP_SUMMARY
          echo "🔀 **Event:** ${{ github.event_name }}" >> $GITHUB_STEP_SUMMARY
          echo "🌿 **Branch:** ${{ github.ref_name }}" >> $GITHUB_STEP_SUMMARY
          echo "👤 **Actor:** ${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "📦 **Repository:** ${{ github.repository }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "⏰ **Greeting Time:** ${{ steps.hello.outputs.time }}" >> $GITHUB_STEP_SUMMARY
```

3. Commit the changes to the `main` branch
4. Navigate to `Actions` and view the workflow execution
5. Click on the workflow run and then on the job name
6. Scroll to the bottom to see the **Job Summary** with formatted markdown!

> 🔒 **Security Best Practice:** Always pin actions to specific versions (@v2, @v4) or commit SHAs to prevent supply chain attacks. Avoid using @main or @latest in production.

---

## 1.3 Understanding Context Variables

GitHub Actions provides rich context information. Let's explore it:

1. Add this debug step to understand available context:

```yaml
      - name: Dump GitHub context
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: echo "$GITHUB_CONTEXT"
```

2. Commit and run the workflow
3. Check the logs to see all available GitHub context properties

> 💡 **Pro Tip:** Use `toJson()` function to pretty-print any context object for debugging.

---

## 1.4 Using modern GITHUB_OUTPUT

The old `set-output` command is deprecated. Let's use the modern approach:

1. Add a step that creates output using the new syntax:

```yaml
      - name: Set custom output
        id: custom-output
        run: |
          echo "current-time=$(date +'%Y-%m-%d %H:%M:%S')" >> $GITHUB_OUTPUT
          echo "random-number=$RANDOM" >> $GITHUB_OUTPUT
      
      - name: Use custom output
        run: |
          echo "Current time is: ${{ steps.custom-output.outputs.current-time }}"
          echo "Random number is: ${{ steps.custom-output.outputs.random-number }}"
```

2. Commit and observe the output in the workflow logs

> ⚠️ **Migration Note:** If you see `::set-output` in old workflows, update to `>> $GITHUB_OUTPUT` for future compatibility.

---

## 1.5 Complete Workflow Example

<details>
<summary>Click to see the complete updated workflow</summary>

```yaml
name: 01-1. GitHub Actions Demo
on: 
  workflow_dispatch:
  workflow_call:
  push:
    branches:
      - main
    paths:
      - 'labs/**'
      - '.github/workflows/github-actions-demo.yml'

jobs:
  Explore-GitHub-Actions:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ github.event_name }} event."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by GitHub!"
      - run: echo "🔎 The name of your branch is ${{ github.ref }} and your repository is ${{ github.repository }}."
      
      - name: Check out repository code
        uses: actions/checkout@v4
      
      - run: echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
      - run: echo "🖥️ The workflow is now ready to test your code on the runner."
      
      - name: List files in the repository
        run: |
          ls ${{ github.workspace }}
      
      - run: echo "🍏 This job's status is ${{ job.status }}."
      
      # Using local custom action from .github/actions/hello-world-js
      - name: Hello world action
        uses: ./.github/actions/hello-world-js
        with:
          who-to-greet: "Mona the Octocat"
        id: hello
      
      - name: Echo the greeting's time
        run: echo 'The time was ${{ steps.hello.outputs.time }}'
      
      # Modern output syntax
      - name: Set custom output
        id: custom-output
        run: |
          echo "current-time=$(date +'%Y-%m-%d %H:%M:%S')" >> $GITHUB_OUTPUT
          echo "random-number=$RANDOM" >> $GITHUB_OUTPUT
      
      - name: Use custom output
        run: |
          echo "Current time is: ${{ steps.custom-output.outputs.current-time }}"
          echo "Random number is: ${{ steps.custom-output.outputs.random-number }}"
      
      # Create rich job summary
      - name: Add job summary
        run: |
          echo "## Workflow Execution Summary 🚀" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "✅ **Job Status:** ${{ job.status }}" >> $GITHUB_STEP_SUMMARY
          echo "🔀 **Event:** ${{ github.event_name }}" >> $GITHUB_STEP_SUMMARY
          echo "🌿 **Branch:** ${{ github.ref_name }}" >> $GITHUB_STEP_SUMMARY
          echo "👤 **Actor:** ${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "📦 **Repository:** ${{ github.repository }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "⏰ **Greeting Time:** ${{ steps.hello.outputs.time }}" >> $GITHUB_STEP_SUMMARY
          echo "🎲 **Random Number:** ${{ steps.custom-output.outputs.random-number }}" >> $GITHUB_STEP_SUMMARY
```
</details>

---

## 🎓 Key Takeaways

- **Triggers matter:** Use `paths` and `branches` filters to optimize workflow runs
- **Pin your actions:** Use specific versions (@v4) for security and stability
- **Modern syntax:** Use `$GITHUB_OUTPUT` instead of deprecated `set-output`
- **Job summaries:** Provide rich feedback with markdown in `$GITHUB_STEP_SUMMARY`
- **Context is king:** Leverage `${{ github.* }}` context for dynamic workflows

## 📚 Additional Resources

- [GitHub Actions starter workflows](https://github.com/actions/starter-workflows)
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [Actions Marketplace](https://github.com/marketplace?type=actions)

---

## ✅ Lab Complete!

You've completed Lab 1! You now understand:
- How to configure workflow triggers
- Modern syntax for outputs and summaries
- Security best practices for action versioning
- How to use context variables effectively

**Next:** Proceed to [Lab 2: Workflow Syntax & Job Dependencies](/labs/lab02.md)
