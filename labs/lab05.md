# 5 - Custom Actions 🛠️

In this lab you will create and use custom actions to extend GitHub Actions capabilities.

> **Duration:** 10-15 minutes

## Learning Objectives

By the end of this lab, you will be able to:
- ✅ Use GitHub Script for inline automation
- ✅ Create composite actions for reusable step sequences
- ✅ Understand JavaScript and Docker action patterns
- ✅ Publish and version custom actions
- ✅ Choose the right action type for your needs

## References
- [Creating actions](https://docs.github.com/en/actions/creating-actions)
- [Creating a composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
- [Creating a JavaScript action](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)
- [GitHub Actions Toolkit](https://github.com/actions/toolkit)
- [actions/github-script](https://github.com/actions/github-script)

---

## 5.1 Use GitHub Script for inline automation

GitHub Script allows you to use the GitHub API directly in your workflows using JavaScript.

1. Open the workflow file [github-script.yml](/.github/workflows/github-script.yml)
2. Update it with modern practices and add new functionality:

```yaml
name: 05-1. GitHub Script Automation

on:
  issues:
    types: [opened, edited, reopened, labeled]
  workflow_dispatch:

# Minimal permissions for security
permissions:
  contents: read
  issues: write

jobs:
  auto-comment:
    name: Add Welcome Comment
    runs-on: ubuntu-latest
    steps:
      - name: Welcome new contributors
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            
            // Check if this is the author's first issue
            const { data: userIssues } = await github.rest.issues.listForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo,
              creator: issue.user.login,
              state: 'all'
            });
            
            if (userIssues.length === 1) {
              await github.rest.issues.createComment({
                issue_number: issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: `👋 Welcome @${issue.user.login}! Thanks for opening your first issue in this repository. We appreciate your contribution! 🎉`
              });
            } else {
              await github.rest.issues.createComment({
                issue_number: issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: `👋 Thank you @${issue.user.login} for your contribution! We appreciate your feedback.`
              });
            }

  apply-labels:
    name: Auto-label Issues
    runs-on: ubuntu-latest
    steps:
      - name: Label based on content
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const labels = [];
            
            // Auto-label based on title/body content
            const content = (issue.title + ' ' + issue.body).toLowerCase();
            
            if (content.includes('bug') || content.includes('error')) {
              labels.push('bug');
            }
            if (content.includes('feature') || content.includes('enhancement')) {
              labels.push('enhancement');
            }
            if (content.includes('documentation') || content.includes('docs')) {
              labels.push('documentation');
            }
            if (content.includes('question') || content.includes('help')) {
              labels.push('question');
            }
            
            // Add training label for all issues
            labels.push('training');
            
            if (labels.length > 0) {
              await github.rest.issues.addLabels({
                issue_number: issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                labels: labels
              });
            }

  analyze-issue:
    name: Analyze and Report
    runs-on: ubuntu-latest
    steps:
      - name: Generate issue analysis
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            
            // Get issue metrics
            const wordCount = issue.body ? issue.body.split(/\s+/).length : 0;
            const hasCodeBlocks = issue.body ? issue.body.includes('```') : false;
            const assigneeCount = issue.assignees ? issue.assignees.length : 0;
            
            const analysis = `
            ## Issue Analysis 📊
            
            - **Word Count:** ${wordCount}
            - **Contains Code:** ${hasCodeBlocks ? 'Yes ✅' : 'No ❌'}
            - **Assignees:** ${assigneeCount}
            - **Labels:** ${issue.labels.length}
            - **Created By:** @${issue.user.login}
            `;
            
            await github.rest.issues.createComment({
              issue_number: issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: analysis
            });
```

3. Commit to `main` branch
4. Create a new issue or edit an existing one to trigger the workflow
5. Observe the automatic comments and labels!

> 💡 **GitHub Script Benefits:**
> - No separate action repository needed
> - Full access to GitHub API via `@octokit/rest`
> - Access to workflow context
> - Great for prototyping and simple automation

---

## 5.2 Create a composite action

Composite actions group multiple steps into a reusable action.

1. Open [.github/actions/hello-world-composite-action/action.yml](/.github/actions/hello-world-composite-action/action.yml)
2. Update it with modern syntax:

```yaml
name: 'Hello World Composite Action'
description: 'A reusable composite action demonstrating inputs, outputs, and step composition'
author: 'Your Name'

inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'World'
  include-time:
    description: 'Include timestamp in greeting'
    required: false
    default: 'true'

outputs:
  random-number:
    description: "Random number generated"
    value: ${{ steps.random-number-generator.outputs.random-id }}
  greeting-time:
    description: "Time when greeting occurred"
    value: ${{ steps.hello-step.outputs.time }}

runs:
  using: "composite"
  steps:
    - name: Display greeting
      id: hello-step
      shell: bash
      run: |
        CURRENT_TIME=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
        echo "time=$CURRENT_TIME" >> $GITHUB_OUTPUT
        
        if [ "${{ inputs.include-time }}" == "true" ]; then
          echo "👋 Hello ${{ inputs.who-to-greet }} at $CURRENT_TIME!"
        else
          echo "👋 Hello ${{ inputs.who-to-greet }}!"
        fi
    
    - name: Generate random number
      id: random-number-generator
      shell: bash
      run: |
        RANDOM_NUM=$RANDOM
        echo "random-id=$RANDOM_NUM" >> $GITHUB_OUTPUT
        echo "🎲 Generated random number: $RANDOM_NUM"
    
    - name: Add to PATH
      shell: bash
      run: echo "${{ github.action_path }}" >> $GITHUB_PATH
    
    - name: Use marketplace action
      uses: actions/hello-world-javascript-action@v2
      with:
        who-to-greet: "${{ inputs.who-to-greet }}"
      id: hello-js
    
    - name: Display output from nested action
      shell: bash
      run: echo "⏰ JavaScript action time was ${{ steps.hello-js.outputs.time }}"
```

3. Update the workflow that uses this action [hello-world-composite.yml](/.github/workflows/hello-world-composite.yml):

```yaml
name: 05-2. Composite Action Demo

on:
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  use-composite-action:
    runs-on: ubuntu-latest
    name: Test Composite Action
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Use local composite action
        id: hello-composite
        uses: ./.github/actions/hello-world-composite-action
        with:
          who-to-greet: 'Workshop Participant'
          include-time: 'true'
      
      - name: Display outputs
        run: |
          echo "Random number: ${{ steps.hello-composite.outputs.random-number }}"
          echo "Greeting time: ${{ steps.hello-composite.outputs.greeting-time }}"
      
      - name: Create job summary
        run: |
          echo "## Composite Action Results 🎯" >> $GITHUB_STEP_SUMMARY
          echo "- **Random Number:** ${{ steps.hello-composite.outputs.random-number }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Greeting Time:** ${{ steps.hello-composite.outputs.greeting-time }}" >> $GITHUB_STEP_SUMMARY
```

4. Commit and create a pull request to test
5. Observe the composite action in action!

> 🎯 **Composite Action Best Practices:**
> - Always specify `shell:` for run steps
> - Use `${{ github.action_path }}` to reference action files
> - Document all inputs and outputs clearly
> - Keep actions focused on a single responsibility

---

## 5.3 Understand custom action types

GitHub Actions supports three types of custom actions:

### 1. JavaScript/TypeScript Actions
**Best for:** Fast execution, GitHub API integration

```yaml
# action.yml
name: 'JavaScript Action'
runs:
  using: 'node20'
  main: 'dist/index.js'
```

**Pros:**
- ✅ Fast startup time
- ✅ Direct access to Actions Toolkit
- ✅ No Docker overhead
- ✅ Works on all runners

**Cons:**
- ❌ Requires Node.js knowledge
- ❌ Need to bundle dependencies

### 2. Docker Actions
**Best for:** Specific runtime requirements, any language

```yaml
# action.yml
name: 'Docker Action'
runs:
  using: 'docker'
  image: 'Dockerfile'
```

**Pros:**
- ✅ Use any language
- ✅ Control full environment
- ✅ Complex dependencies

**Cons:**
- ❌ Slower startup (build/pull image)
- ❌ Only works on Linux runners
- ❌ Larger resource usage

### 3. Composite Actions
**Best for:** Combining existing steps/actions

```yaml
# action.yml
name: 'Composite Action'
runs:
  using: 'composite'
  steps:
    - run: echo "Hello"
      shell: bash
```

**Pros:**
- ✅ Easiest to create
- ✅ No programming required
- ✅ Combine existing actions
- ✅ Works on all runners

**Cons:**
- ❌ Limited error handling
- ❌ No complex logic

---

## 5.4 Test existing custom actions

Let's test the JavaScript and Docker actions in the repository.

1. Open [use-custom-actions.yml](/.github/workflows/use-custom-actions.yml)
2. Update it to test both types:

```yaml
name: 05-3. Custom Actions Showcase

on:
  pull_request:
    types: [labeled]
  workflow_dispatch:

permissions:
  contents: read
  issues: write

jobs:
  test-javascript-actions:
    name: JavaScript Actions
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Hello from JS action
        uses: ./.github/actions/hello-world-js
      
      - name: Get a joke
        uses: ./.github/actions/joke-action
        id: joke
      
      - name: Display joke
        run: echo "😄 Joke: ${{ steps.joke.outputs.joke-output }}"
      
      - name: Create issue with joke
        uses: ./.github/actions/issue-maker-js
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          joke: ${{ steps.joke.outputs.joke-output }}
          issue-title: "Daily Joke from Custom Actions 😄"

  test-docker-actions:
    name: Docker Actions
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Hello from Docker action
        uses: ./.github/actions/hello-world-docker
      
      - name: Get cat fact
        uses: ./.github/actions/cat-facts
        id: cat
      
      - name: Display cat fact
        run: echo "🐱 Fact: ${{ steps.cat.outputs.fact }}"
      
      - name: Create issue with cat fact
        uses: ./.github/actions/issue-maker-docker
        with:
          repoToken: ${{ secrets.GITHUB_TOKEN }}
          catFact: ${{ steps.cat.outputs.fact }}
          issueTitle: "Cat Fact from Docker Action 🐱"
```

3. Commit to `main` and manually trigger the workflow
4. Check the created issues!

---

## 5.5 (Optional) Create your own JavaScript action

If you want to create a new JavaScript action:

### Quick Start:

1. Create a new repository (e.g., `my-custom-action`)
2. Initialize with npm:

```bash
npm init -y
npm install @actions/core @actions/github
```

3. Create `index.js`:

```javascript
const core = require('@actions/core');
const github = require('@actions/github');

try {
  const nameToGreet = core.getInput('who-to-greet');
  console.log(`Hello ${nameToGreet}!`);
  
  const time = new Date().toTimeString();
  core.setOutput('time', time);
  
  // Access GitHub context
  console.log(`Event: ${github.context.eventName}`);
  
} catch (error) {
  core.setFailed(error.message);
}
```

4. Create `action.yml`:

```yaml
name: 'My Custom Action'
description: 'Greets someone'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time:
    description: 'The time we greeted'
runs:
  using: 'node20'
  main: 'index.js'
```

5. Commit and push
6. Use in workflows:

```yaml
- uses: your-username/my-custom-action@v1
  with:
    who-to-greet: 'Octocat'
```

---

## 5.6 Publishing and versioning actions

When you're ready to share your action:

### Best Practices:

1. **Use Semantic Versioning:**
   ```bash
   git tag -a v1.0.0 -m "First release"
   git push origin v1.0.0
   ```

2. **Create major version tags:**
   ```bash
   git tag -fa v1 -m "Update v1 tag"
   git push origin v1 --force
   ```

3. **Add to GitHub Marketplace:**
   - Add `branding` to action.yml
   - Create a release on GitHub
   - Check "Publish to Marketplace"

4. **Document thoroughly:**
   - Clear README with examples
   - Document all inputs/outputs
   - Provide usage examples
   - Include a LICENSE

### Example action.yml with branding:

```yaml
name: 'My Awesome Action'
description: 'Does something awesome'
branding:
  icon: 'activity'
  color: 'blue'
inputs:
  # ...
runs:
  # ...
```

---

## 🎓 Key Takeaways

- **GitHub Script:** Quick automation without separate repositories
- **Composite Actions:** Group steps for reusability
- **JavaScript Actions:** Fast, full-featured custom logic
- **Docker Actions:** Any language, controlled environment
- **Choose wisely:** Match action type to your needs
- **Version properly:** Use semantic versioning and tags

## 📊 Decision Matrix

| Need | Use This |
|------|----------|
| GitHub API automation | GitHub Script |
| Group existing steps | Composite Action |
| Fast custom logic | JavaScript Action |
| Specific runtime | Docker Action |
| Simple reuse | Composite Action |
| Marketplace publishing | JavaScript/Docker Action |

---

## ✅ Lab Complete!

You've mastered custom actions! You now understand:
- Using GitHub Script for inline automation
- Creating composite actions
- Different action types and when to use them
- Publishing and versioning actions
- Best practices for action development

**Next:** Proceed to [Lab 6: Self-hosted Runners](/labs/lab06.md)
