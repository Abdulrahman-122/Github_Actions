Absolutely. The biggest confusion is that **reusable workflows are not actions**.

You just learned:

* Composite Action = reusable **steps**
* Reusable Workflow = reusable **jobs/pipeline**

Think of it like this:

```text
Composite Action
└── Step 1
└── Step 2
└── Step 3

Reusable Workflow
└── Job A
└── Job B
└── Job C
```

---

# Step 1: Create the reusable workflow

Create:

```text
.github/workflows/reusable-node.yml
```

Contents:

```yaml
name: Reusable Node Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  node-job:
    runs-on: ubuntu-latest

    steps:
      - name: Print Node Version
        run: echo "Requested Node Version = ${{ inputs.node-version }}"
```

---

## What is happening?

This:

```yaml
on:
  workflow_call:
```

means:

> "Nobody runs me directly. Another workflow must call me."

Similar to a function in programming:

```python
def greet(name):
    print(name)
```

The function waits until someone calls it.

---

# Step 2: Create the caller workflow

Create:

```text
.github/workflows/main.yml
```

Contents:

```yaml
name: Main Workflow

on:
  workflow_dispatch:

jobs:
  call-reusable:
    uses: ./.github/workflows/reusable-node.yml

    with:
      node-version: "20"
```

---

# What happens?

You click:

```text
Actions
└── Main Workflow
      └── Run Workflow
```

GitHub executes:

```text
main.yml
   ↓
reusable-node.yml
   ↓
node-job
```

Output:

```text
Requested Node Version = 20
```

---

# Why is this different from composite actions?

## Composite Action

You call it inside a step:

```yaml
steps:
  - uses: ./actions/node-ci
```

because it is just a group of steps.

---

## Reusable Workflow

You call it inside a job:

```yaml
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable-node.yml
```

because it contains entire jobs.

---

# Let's extend the lab

Imagine a company wants every project to run:

```text
Checkout
Lint
Test
Build
```

They don't want developers changing it.

Create:

```yaml
name: Company CI

on:
  workflow_call:
    inputs:
      project-name:
        required: true
        type: string

jobs:
  ci:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Testing ${{ inputs.project-name }}"
      - run: echo "Linting..."
      - run: echo "Running tests..."
      - run: echo "Building..."
```

Then any repository can use:

```yaml
jobs:
  company-ci:
    uses: ./.github/workflows/company-ci.yml

    with:
      project-name: frontend
```

---

# What about secrets?

Suppose your reusable workflow needs:

```yaml
secrets:
  DOCKER_PASSWORD:
    required: true
```

inside:

```yaml
on:
  workflow_call:
```

Then the caller passes:

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/deploy.yml

    secrets:
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```

---

# Real-world example

Imagine you're a DevOps engineer.

You manage 30 repositories.

Without reusable workflows:

```text
repo1
  ci.yml

repo2
  ci.yml

repo3
  ci.yml

...
repo30
  ci.yml
```

Every repo duplicates the same pipeline.

---

With reusable workflows:

```text
central-ci.yml
```

All repositories use:

```yaml
jobs:
  ci:
    uses: company/platform/.github/workflows/central-ci.yml@main
```

Now changing one file updates CI for every repository.

---

# Rule of thumb

Use a **Composite Action** when:

```text
I want to reuse steps.
```

Example:

```text
Install Node
Run Tests
Upload Report
```

---

Use a **Reusable Workflow** when:

```text
I want to reuse entire jobs/pipelines.
```

Example:

```text
Build
Test
Deploy
Security Scan
```

That's the exact distinction this lab is trying to teach. Composite actions are reusable building blocks; reusable workflows are reusable pipelines.
