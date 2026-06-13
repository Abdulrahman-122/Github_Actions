This lab is teaching a very important DevOps security principle:

> **Give workflows the minimum permissions they need.**

Think of `GITHUB_TOKEN` as a temporary GitHub account that GitHub creates automatically for every workflow run.

---

# First: What is GITHUB_TOKEN?

When your workflow starts, GitHub creates a token:

```yaml
${{ secrets.GITHUB_TOKEN }}
```

This token can:

* Read repository contents
* Read pull requests
* Create issues
* Modify pull requests
* Create releases
* Push code

**But only if you grant the permissions.**

---

# Why do permissions exist?

Imagine this workflow:

```yaml
- run: rm everything
```

If the token had full write permissions everywhere, a compromised action could:

* Delete branches
* Edit PRs
* Create releases
* Modify issues

That's dangerous.

So GitHub lets you restrict permissions.

---

# Lab 1: Read-only permission

Create:

```yaml
name: Read_Only_Test

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    permissions:
      pull-requests: read

    steps:
      - uses: actions/checkout@v5

      - name: Show token exists
        run: echo "Token available"

      - name: Read permission
        run: echo "Can read PRs"
```

Notice:

```yaml
permissions:
  pull-requests: read
```

means:

```text
Can view PRs
Cannot modify PRs
```

---

# What does this command do?

```yaml
env:
  GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

run: gh pr list
```

The GitHub CLI (`gh`) uses:

```bash
GH_TOKEN
```

to authenticate.

Equivalent idea:

```bash
gh auth login
```

but automatically.

---

# Why does this work?

```bash
gh pr list
```

only reads information.

Reading is allowed.

---

# Why does this fail?

```bash
gh pr edit 1 --add-label documentation
```

because:

```yaml
permissions:
  pull-requests: read
```

doesn't allow modifying PRs.

GitHub returns:

```text
403 Forbidden
```

or permission denied.

---

# What is continue-on-error?

Without:

```yaml
continue-on-error: true
```

the workflow stops immediately when:

```bash
gh pr edit ...
```

fails.

With:

```yaml
continue-on-error: true
```

GitHub continues to the next step.

---

# What does if: failure() mean?

```yaml
- name: Confirm Failure
  if: failure()
  run: echo "Expected failure"
```

runs only when a previous step failed.

Think:

```python
if previous_step_failed:
    print("Expected failure")
```

---

# Lab 2: See failure yourself

Create:

```yaml
name: Failure_Test

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Force failure
        run: exit 1

      - name: Will never run
        run: echo "Hello"
```

Result:

```text
Step 1 failed
Workflow stopped
```

Now add:

```yaml
continue-on-error: true
```

to the first step.

The second step runs.

---

# Lab 3: Write permission

Now create:

```yaml
name: Permission_Test

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    permissions:
      pull-requests: write

    steps:
      - run: echo "Write access granted"
```

Now GitHub allows:

```bash
gh pr edit
```

```bash
gh pr comment
```

```bash
gh pr close
```

and other modifications.

---

# Understanding the permissions block

Examples:

### Read repository contents

```yaml
permissions:
  contents: read
```

Can:

```text
Read files
Clone repo
Checkout code
```

Cannot:

```text
Push commits
Create tags
```

---

### Write repository contents

```yaml
permissions:
  contents: write
```

Can:

```text
Push commits
Create releases
Create tags
```

---

### Read issues

```yaml
permissions:
  issues: read
```

Can:

```text
View issues
```

Cannot:

```text
Close issues
Comment on issues
```

---

### Write issues

```yaml
permissions:
  issues: write
```

Can:

```text
Create issues
Edit issues
Close issues
```

---

# Best practice

For most CI workflows:

```yaml
permissions:
  contents: read
```

is enough.

For deployment workflows:

```yaml
permissions:
  contents: write
```

may be needed.

For PR automation:

```yaml
permissions:
  pull-requests: write
```

may be needed.

---

## Practical lab for your course repo

Create a workflow:

```yaml
name: Show_Github_Token_Permissions

on:
  workflow_dispatch:

jobs:
  inspect:
    runs-on: ubuntu-latest

    permissions:
      contents: read

    steps:
      - uses: actions/checkout@v5

      - name: Show actor
        run: |
          echo "User: ${{ github.actor }}"
          echo "Repo: ${{ github.repository }}"
          echo "Branch: ${{ github.ref }}"
```

Run it and inspect the logs.

Then change:

```yaml
permissions:
  contents: read
```

to:

```yaml
permissions:
  contents: write
```

Nothing visible changes in the logs, but now the workflow has authority to push commits or create tags. That's the core lesson of this section: **permissions control what the workflow is allowed to do, not what it prints.**

