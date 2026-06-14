This is one of the most important GitHub Actions concepts because it answers:

> "When should I create a Composite Action and when should I create a Reusable Workflow?"

---

# Think of it like Python

## Composite Action = Function

Python:

```python
def greet(name):
    print(f"Hello {name}")
```

Usage:

```python
greet("Abdulrahman")
```

---

GitHub:

```yaml
uses: ./.github/actions/greet
with:
  name: Abdulrahman
```

A composite action is basically a reusable function.

---

## Reusable Workflow = Entire Program

Python:

```python
def deploy_pipeline():
    build()
    test()
    deploy()
```

Usage:

```python
deploy_pipeline()
```

---

GitHub:

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/deploy.yml
```

A reusable workflow is basically a reusable pipeline.

---

# Step 1: The Reusable Workflow

Normal workflow:

```yaml
on:
  push:
```

runs when code is pushed.

---

Reusable workflow:

```yaml
on:
  workflow_call:
```

This means:

> "Don't run me directly. Another workflow must call me."

Like:

```python
def deploy():
```

The function exists but does nothing until called.

---

# Step 2: Inputs

```yaml
inputs:
  foo:
    required: true
    type: string
```

Equivalent Python:

```python
def deploy(foo):
```

Caller must pass:

```yaml
with:
  foo: bar
```

---

Another input:

```yaml
environment:
```

Equivalent:

```python
def deploy(environment):
```

Caller chooses:

```yaml
with:
  environment: staging
```

or

```yaml
with:
  environment: production
```

---

# Step 3: Secrets

Reusable workflow:

```yaml
secrets:
  EXAMPLE_REPOSITORY_SECRET:
    required: true
```

Equivalent:

```python
def deploy(secret):
```

The caller must send the secret.

---

# Step 4: The Job

Inside reusable workflow:

```yaml
jobs:
  reusable-workflow-job:
```

This is a normal job.

Nothing special.

---

# Step 5: Environment

This line is powerful:

```yaml
environment: ${{ inputs.environment }}
```

Suppose GitHub has:

---

Staging Environment

```text
DATABASE_URL=staging-db
API_KEY=staging-key
```

---

Production Environment

```text
DATABASE_URL=prod-db
API_KEY=prod-key
```

---

Caller:

```yaml
environment: staging
```

loads staging secrets.

---

Caller:

```yaml
environment: production
```

loads production secrets.

---

Same workflow.

Different credentials.

---

# Step 6: Why use environments?

Imagine deployment.

Bad:

```yaml
deploy-staging.yml
deploy-production.yml
```

Duplicate code.

---

Better:

```yaml
deploy.yml
```

Input:

```yaml
environment
```

Choose staging or production.

---

# Step 7: Using Inputs

Inside workflow:

```yaml
FOO: ${{ inputs.foo }}
```

Caller:

```yaml
foo: bar
```

Result:

```bash
FOO=bar
```

Then:

```bash
echo $FOO
```

prints:

```text
bar
```

---

# Step 8: Using Secrets

```yaml
EXAMPLE_REPOSITORY_SECRET: ${{ secrets.EXAMPLE_REPOSITORY_SECRET }}
```

Same concept.

GitHub injects the secret.

---

# Step 9: Caller Workflow

Now comes the magic.

---

Reusable workflow:

```text
.github/workflows/deploy.yml
```

Caller:

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/deploy.yml
```

Notice:

```yaml
uses:
```

inside jobs.

Not inside steps.

This is the biggest difference.

---

Composite Action:

```yaml
steps:
  - uses: ...
```

Reusable Workflow:

```yaml
jobs:
  deploy:
    uses: ...
```

---

# Visual Difference

Composite Action

```yaml
Job
 ├── Step 1
 ├── Step 2
 └── Composite Action
       ├── Step A
       └── Step B
```

Runs inside a step.

---

Reusable Workflow

```yaml
Caller Workflow
 ├── Job A
 ├── Job B
 └── Reusable Workflow
        ├── Job X
        ├── Job Y
        └── Job Z
```

Runs entire jobs.

---

# Step 10: secrets: inherit

This line:

```yaml
secrets: inherit
```

means:

> "Pass all available secrets to the reusable workflow."

Without it:

```yaml
secrets:
  EXAMPLE_REPOSITORY_SECRET: ${{ secrets.EXAMPLE_REPOSITORY_SECRET }}
```

you must manually forward every secret.

---

Example:

Caller has:

```text
DB_PASSWORD
API_KEY
JWT_SECRET
```

---

Using:

```yaml
secrets: inherit
```

all 3 are available.

---

Without:

```yaml
secrets:
```

you must specify each one.

---

# Why SHA is used here?

Relative path:

```yaml
uses: ./.github/workflows/deploy.yml
```

Uses current repository version.

---

Cross repository:

```yaml
uses: owner/repo/.github/workflows/deploy.yml@f2d2e4a
```

Uses exact commit.

Security.

Same reason you learned:

```yaml
actions/checkout@ff7abcd...
```

instead of:

```yaml
actions/checkout@v5
```

---

# Real DevOps Example

Imagine 30 repositories.

Each one needs:

```text
Checkout
Run Tests
Build Docker Image
Push Docker Image
Deploy Kubernetes
```

Without reusable workflows:

```text
repo1 -> 100 lines
repo2 -> 100 lines
repo3 -> 100 lines
...
repo30 -> 100 lines
```

3000 lines duplicated.

---

With reusable workflow:

```text
deploy.yml
```

contains everything once.

---

Every repo:

```yaml
jobs:
  deploy:
    uses: company/shared-pipelines/.github/workflows/deploy.yml@sha
```

Done.

---

# Interview Question

If somebody asks:

> When should I use Composite Actions vs Reusable Workflows?

Answer:

| Use Case                         | Choose            |
| -------------------------------- | ----------------- |
| Reuse a few steps                | Composite Action  |
| Reuse an entire CI/CD pipeline   | Reusable Workflow |
| Runs inside a step               | Composite Action  |
| Runs as a job                    | Reusable Workflow |
| Docker login helper              | Composite Action  |
| Company-wide deployment pipeline | Reusable Workflow |

For someone heading toward DevOps, you'll likely create **many composite actions** (Docker build, Helm deploy, security scan) and a few **reusable workflows** (standard CI pipeline, Kubernetes deployment pipeline, release pipeline) that all repositories share.
