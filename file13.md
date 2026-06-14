Excellent. This is where GitHub Actions becomes much more powerful.

Up until now you've been **using Actions**:

```yaml
- uses: actions/checkout@v5
- uses: actions/cache@v4
```

Now you're learning how to **create your own Actions**.

---

# Big Picture

Imagine you have 20 repositories.

Every repository needs:

```bash
Install Python
Install dependencies
Run tests
Check formatting
```

Copying that YAML 20 times is annoying.

GitHub gives you 4 ways to package that logic.

Think of them like increasing levels of power.

---

# Level 1: Composite Actions

## What problem does it solve?

You keep repeating the same steps.

Example:

```yaml
steps:
  - run: python --version
  - run: pip install -r requirements.txt
  - run: pytest
```

in 10 workflows.

Instead of copying:

```yaml
steps:
  - uses: my-python-test-action
```

---

## Mental Model

A Composite Action is:

```text
Many steps
↓
Become
↓
One step
```

---

## Visual

Before:

```yaml
steps:
  - run: step1
  - run: step2
  - run: step3
```

After:

```yaml
steps:
  - uses: my-action
```

---

# Lab 1

Create:

```text
.github/actions/hello/action.yml
```

```yaml
name: Hello Action

runs:
  using: composite

  steps:
    - run: echo "Hello Abdulrahman"
      shell: bash

    - run: date
      shell: bash
```

---

Workflow:

```yaml
name: Test Composite

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - uses: ./.github/actions/hello
```

---

Question:

What do you think happens?

Answer:

GitHub executes:

```bash
echo "Hello Abdulrahman"
date
```

even though your workflow contains only one step.

---

# Level 2: Reusable Workflows

This is bigger.

---

## Composite Action

Can replace:

```yaml
steps:
```

only.

---

## Reusable Workflow

Can replace:

```yaml
jobs:
```

---

Imagine 50 repositories need:

```yaml
lint
test
build
deploy
```

You don't want to copy entire workflows.

---

Instead create:

```text
.github/workflows/python-ci.yml
```

Once.

Then call it everywhere.

---

Visual:

Normal:

```yaml
repo1
 └─ 200 lines

repo2
 └─ 200 lines

repo3
 └─ 200 lines
```

Reusable Workflow:

```yaml
repo1
 └─ call workflow

repo2
 └─ call workflow

repo3
 └─ call workflow
```

---

# Lab 2

Reusable workflow:

```yaml
name: Python CI

on:
  workflow_call:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Running tests"
```

---

Caller workflow:

```yaml
jobs:
  call-ci:
    uses: ./.github/workflows/python-ci.yml
```

---

Question:

Does GitHub run a step?

No.

GitHub runs an entire job.

---

Mental model:

```text
Composite Action
=
Reusable Step

Reusable Workflow
=
Reusable Job
```

---

# Level 3: JavaScript/TypeScript Actions

Now we stop writing shell scripts.

We write actual programs.

---

Question:

What if you need logic like:

```text
Read GitHub API
Check PR title
Comment on PR
Create labels
Analyze files
```

Shell becomes messy.

Use JavaScript.

---

Example:

```javascript
console.log("Hello");
```

becomes an Action.

---

GitHub runs:

```text
Node.js
↓
Your code
↓
GitHub API
```

---

Visual:

```yaml
steps:
  - uses: my-js-action
```

Inside:

```javascript
GitHub API
Business Logic
Validation
```

---

Real-world examples:

* checkout
* setup-node
* cache

Most famous Actions are JavaScript Actions.

---

# Level 4: Container Actions

This is the most flexible.

---

Question:

What if your Action needs:

```text
Python
PostgreSQL client
Terraform
kubectl
Docker
```

and 100 dependencies?

JavaScript Action becomes painful.

---

Use a Docker container.

---

Visual:

```yaml
steps:
  - uses: my-container-action
```

GitHub does:

```text
Pull container
Start container
Run code
Destroy container
```

---

Example:

Container contains:

```text
Python 3.12
Flask
Requests
Kubectl
Terraform
```

---

Your Action runs:

```python
print("Hello")
```

inside the container.

---

# Which One Should You Use?

### Composite Action

Use when:

```text
I just want to bundle shell commands.
```

Examples:

* lint
* build
* test

---

### Reusable Workflow

Use when:

```text
I want to reuse entire jobs.
```

Examples:

* company CI pipeline
* deployment pipeline

---

### JavaScript Action

Use when:

```text
I need GitHub API integration.
```

Examples:

* PR validation
* label automation
* comment bots

---

### Container Action

Use when:

```text
I need custom tools or another language.
```

Examples:

* Python automation
* Go automation
* Terraform automation
* Kubernetes automation

---

## Quick Quiz

If you want to reuse:

```yaml
- run: npm test
- run: npm lint
```

what do you choose?

➡️ Composite Action.

---

If you want to reuse:

```yaml
build
test
deploy
```

as an entire pipeline?

➡️ Reusable Workflow.

---

If you want to automatically comment on every Pull Request using the GitHub API?

➡️ JavaScript Action.

---

If you want an Action written in Python with 20 Python packages installed?

➡️ Container Action.

---

The next lesson in the course will almost certainly start with **Composite Actions**, because they're the simplest custom Actions to build and the easiest bridge from the GitHub workflows you've already been writing.
