This lesson is really teaching **three separate concepts**:

1. **Secrets** (private values)
2. **Variables** (public configuration values)
3. **Contexts** (`github`, `env`, `needs`, `vars`, `secrets`, etc.)

Let's build a lab for each.

---

# Lab 1: Repository Variables

## Goal

Learn how `vars` work.

### Step 1

Go to your GitHub repository:

```text
Repository
 ↓
Settings
 ↓
Secrets and variables
 ↓
Actions
 ↓
Variables
```

Click:

```text
New repository variable
```

Create:

```text
Name: PROJECT_NAME
Value: MyBlog
```

---

### Step 2

Create:

```yaml
name: Variables Lab

on:
  workflow_dispatch:

jobs:

  test:

    runs-on: ubuntu-latest

    env:
      PROJECT: ${{ vars.PROJECT_NAME }}

    steps:

      - run: |
          echo "Project is:"
          echo "$PROJECT"
```

---

### Prediction

Output:

```text
Project is:
MyBlog
```

---

# Lab 2: Repository Secrets

## Goal

Learn how secrets are hidden.

### Step 1

Go to:

```text
Settings
 ↓
Secrets and variables
 ↓
Actions
 ↓
Secrets
```

Create:

```text
Name: API_KEY
Value: super-secret-key
```

---

### Step 2

Workflow:

```yaml
name: Secrets Lab

on:
  workflow_dispatch:

jobs:

  test:

    runs-on: ubuntu-latest

    env:
      API_KEY: ${{ secrets.API_KEY }}

    steps:

      - run: |
          echo "API key:"
          echo "$API_KEY"
```

but now as you see github will masked it as it wants to prevent any secret key to be allowed in logs
---

### Prediction

You might think:

```text
super-secret-key
```

But GitHub prints:

```text
***
```

because secrets are masked.

---

## Why?

Imagine:

```text
AWS_KEY
DATABASE_PASSWORD
DOCKER_TOKEN
```

You don't want those appearing in logs.

GitHub automatically replaces them with:

```text
***
```

---

# Lab 3: Variable vs Secret

Create both:

```text
Variable:
PROJECT_NAME=MyBlog

Secret:
API_KEY=super-secret-key
```

Workflow:

```yaml
name: Secret_vs_Variable

on:
  workflow_dispatch:

jobs:

  test:

    runs-on: ubuntu-latest

    env:
      PROJECT: ${{ vars.PROJECT_NAME }}
      API_KEY: ${{ secrets.API_KEY }}

    steps:

      - run: |
          echo "PROJECT=$PROJECT"
          echo "API_KEY=$API_KEY"
```

---

### Prediction

Output:

```text
PROJECT=MyBlog
API_KEY=***
```

---

# Mental Model

Use variables for:

```text
Project Name
Region
Environment Name
Docker Image Name
```

Use secrets for:

```text
Passwords
API Keys
Tokens
Private Certificates
```

---

# Lab 4: Environments

Now the interesting part.

Imagine:

```text
Staging
Production
```

You don't want both to use the same credentials.

---

## Create Environment

Repository:

```text
Settings
 ↓
Environments
 ↓
New Environment
```

Create:

```text
staging
```

and

```text
production
```

---

## Add Environment Variables

Inside **staging**:

```text
DATABASE_URL=staging-db.company.com
```

Inside **production**:

```text
DATABASE_URL=prod-db.company.com
```

---

## Workflow

```yaml
name: Environment Lab

on:
  workflow_dispatch:

jobs:

  staging:

    runs-on: ubuntu-latest

    environment: staging

    env:
      DATABASE_URL: ${{ vars.DATABASE_URL }}

    steps:

      - run: echo "$DATABASE_URL"

  production:

    runs-on: ubuntu-latest

    environment: production

    env:
      DATABASE_URL: ${{ vars.DATABASE_URL }}

    steps:

      - run: echo "$DATABASE_URL"
```

---

### Prediction

Staging:

```text
staging-db.company.com
```

Production:

```text
prod-db.company.com
```

Same workflow.

Different environment.

Different values.

---

# Lab 5: Understanding Contexts

This is where `${{ }}` comes from.

Think:

```text
GitHub stores information
       ↓
Contexts expose that information
       ↓
You read it with ${{ }}
```

---

## github Context

Workflow:

```yaml
name: Github Context Lab

on:
  workflow_dispatch:

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - run: |
          echo "Actor: ${{ github.actor }}"
          echo "Repository: ${{ github.repository }}"
          echo "Workflow: ${{ github.workflow }}"
```

---

### Example Output

```text
Actor: AbdulRahmanQasim
Repository: AbdulRahmanQasim/devops-directive-github-actions-course
Workflow: Github Context Lab
```

---

# Lab 6: Event Context

Use:

```yaml
on:
  push:
```

Workflow:

```yaml
steps:

  - run: |
      echo "Branch:"
      echo "${{ github.ref }}"
```

Push to main.

Output:

```text
refs/heads/main
```

---

# Lab 7: env Context

Workflow:

```yaml
env:
  PROJECT: FlaskAPI

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - run: |
          echo "${{ env.PROJECT }}"
```

Output:

```text
FlaskAPI
```

---

# Lab 8: needs Context

This combines what you've already learned.

```yaml
name: Needs Context Lab

on:
  workflow_dispatch:

jobs:

  producer:

    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.gen.outputs.version }}

    steps:

      - id: gen

        run: |
          echo "version=1.0.0" >> $GITHUB_OUTPUT

  consumer:

    runs-on: ubuntu-latest

    needs: producer

    steps:

      - run: |
          echo "${{ needs.producer.outputs.version }}"
```

Output:

```text
1.0.0
```

---
To_Sum_up
```yaml
name: understand_context
on:
  workflow_dispatch:
  push:
jobs:
  context:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.gen.outputs.version }}
    steps:
      - name: check
        id: gen
        run: |
          echo "Actor: ${{ github.actor }}"
          echo "Repo: ${{ github.repository }}"
          echo "Workflow: ${{ github.workflow }}"
          echo "version=1.0.0">> $GITHUB_OUTPUT

  check_branch:
    runs-on: ubuntu-latest
    env:
     Project: Full_stack
    needs:
      - context
    steps:
       - name: check_branch
         run: |
          echo "Branch is "
          echo ${{ github.ref }}
          echo " $Project "
          echo "${{env.Project}}"
          echo "The version of the first job is ${{ needs.context.outputs.version}}"


```

# Final Challenge

Create a workflow that prints:

1. Your GitHub username (`github.actor`)
2. Repository variable `PROJECT_NAME`
3. Repository secret `API_KEY`
4. Version output from a previous job


