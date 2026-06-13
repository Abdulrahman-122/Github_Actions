This is where GitHub Actions starts becoming a real CI/CD tool instead of just "running commands". I'll give you **3 labs** that build on each other.

---

# Mental Model First

Think of a workflow like this:

```text
Workflow
│
├── Job 1
│   ├── Step 1
│   ├── Step 2
│   └── Step 3
│
└── Job 2
    ├── Step 1
    └── Step 2
```

Important:

```text
Workflow Variables
    ↓ visible everywhere

Job Variables
    ↓ visible only inside that job

Step Variables
    ↓ visible only inside that step
```

---

# LAB 1: Understanding Variable Scope

Create:

```text
.github/workflows/env-scope-lab.yml
```

```yaml
name: Environment Scope Lab

on:
  workflow_dispatch:

env:
  COMPANY: OpenAI

jobs:
  backend:
    runs-on: ubuntu-latest

    env:
      PROJECT: FlaskAPI

    steps:
      - name: Step 1
        env:
          VERSION: v1.0

        run: |
          echo "COMPANY=$COMPANY"
          echo "PROJECT=$PROJECT"
          echo "VERSION=$VERSION"

      - name: Step 2
        run: |
          echo "COMPANY=$COMPANY"
          echo "PROJECT=$PROJECT"
          echo "VERSION=${VERSION:-NOT_FOUND}"

  frontend:
    runs-on: ubuntu-latest

    steps:
      - name: Frontend Step
        run: |
          echo "COMPANY=$COMPANY"
          echo "PROJECT=${PROJECT:-NOT_FOUND}"
          echo "VERSION=${VERSION:-NOT_FOUND}"
```

---

## Predict Before Running

### Backend Step 1

What exists?

```text
COMPANY ✓
PROJECT ✓
VERSION ✓
```

Output:

```text
COMPANY=OpenAI
PROJECT=FlaskAPI
VERSION=v1.0
```

---

### Backend Step 2

What exists?

```text
COMPANY ✓
PROJECT ✓
VERSION ✗
```

Output:

```text
COMPANY=OpenAI
PROJECT=FlaskAPI
VERSION=NOT_FOUND
```

---

### Frontend Job

What exists?

```text
COMPANY ✓
PROJECT ✗
VERSION ✗
```

Output:

```text
COMPANY=OpenAI
PROJECT=NOT_FOUND
VERSION=NOT_FOUND
```

---

## What You Learn

Workflow variable:

```yaml
env:
  COMPANY: OpenAI
```

Visible everywhere.

---

Job variable:

```yaml
jobs:
  backend:
    env:
      PROJECT: FlaskAPI
```

Visible only inside backend.

---

Step variable:

```yaml
steps:
  - env:
      VERSION: v1.0
```

Visible only inside that step.

---

# LAB 2: GITHUB_ENV

This is extremely important.

Imagine:

```text
Step 1 calculates version
Step 2 needs version
Step 3 needs version
```

How do we share it?

Using:

```bash
$GITHUB_ENV
```

---

Create:

```text
.github/workflows/github-env-lab.yml
```

```yaml
name: GITHUB_ENV Lab

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Create version
        run: |
          VERSION=1.0.0

          echo "VERSION=$VERSION" >> $GITHUB_ENV

      - name: Use version
        run: |
          echo "Version is $VERSION"

      - name: Use version again
        run: |
          echo "Still available: $VERSION"
```

---

## What Happens?

Step 1:

```bash
echo "VERSION=1.0.0" >> $GITHUB_ENV
```

tells GitHub:

> Make VERSION available to future steps in this job.

---

Output:

```text
Version is 1.0.0
Still available: 1.0.0
```

---

## Real DevOps Example

Calculate Docker tag:

```bash
VERSION=$(git rev-parse --short HEAD)

echo "VERSION=$VERSION" >> $GITHUB_ENV
```

Later:

```bash
docker build -t myapp:$VERSION .
```

---

# LAB 3: GITHUB_OUTPUT

Now we go one level higher.

Imagine:

```text
Job 1 builds application
Job 2 deploys application
```

Job 2 needs information from Job 1.

This is where:

```bash
$GITHUB_OUTPUT
```

comes in.

---

Create:

```text
.github/workflows/github-output-lab.yml
```

```yaml
name: Github_output_check
on:
  workflow_dispatch: 

jobs:
  Producer:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.generate.outputs.version }}
      # this will give me empty output version
    steps:
     - name: Generate_output
       id: generate
       run: |
        VERSION=2.0.4
        echo "version=$VERSION" >> $GITHUB_OUTPUT
     -  name: Varify
        run: |
          echo "Version inside Producer:"
          echo "${{steps.generate.outputs.version}}"
          # this will generate the same version 
  Consumer:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.Read.outputs.version }}
    steps:
     - name: Read output
       id : Read
       run: |
        Version=2.1.4
        echo "version: $Version " >> $GITHUB_OUTPUT
     - name: Verify
       run: |
         echo "${{steps.Read.outputs.version}}"
  anotherConsumer:
      runs-on: ubuntu-latest
      needs: 
      - Producer
      - Consumer
      steps:
       - name: Read both Versions
         run: |
          echo "This is the version of Job1 ${{needs.Producer.outputs.version}}"
          echo "This is the version of job2 ${{needs.Consumer.outputs.version}}"
```

---

# Visualize What Happens

Producer:

```text
VERSION=2.1.5
```

↓

Step Output:

```text
version=2.1.5
```

↓

Job Output:

```text
producer.outputs.version
```

↓

Consumer:

```text
needs.producer.outputs.version
```

---

Output:

```text
Received version:
2.1.5
```

---

# LAB 4: Why GITHUB_ENV Doesn't Cross Jobs

Create:

```yaml
name: Check_Output_of_github
on:
  workflow_dispatch:
  push:
   branches:
    - main
jobs:
  backend:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.gen.outputs.version }}
    steps:
      - name: apis
        id: gen
        run: |
         Version=7.1.0
         echo "version=$Version" >> $GITHUB_ENV
         echo "version=$Version" >> $GITHUB_OUTPUT
      - name: check_apis
        run: |
          echo "${{ steps.gen.outputs.version }}"
      # note ; to see the outputs you must define two steps one for  generation + other for varification as this is how github works but not the same at a time

  Frontend:
    runs-on: ubuntu-latest
    needs:
     - backend
    steps:
      - name: check_versions_of_last_job
        run: |
          echo "let's check version using environment variable(will not give you any value)"
          echo $Version
          echo "Let's check version of last job using outputs.version"
          echo " ${{ needs.backend.outputs.version }} "

```

---

## Predict

### Producer

Creates:

```text
VERSION=3.0.0
```

inside GITHUB_ENV.

Creates:

```text
version=3.0.0
```

inside GITHUB_OUTPUT.

---

### Consumer

Will see:

```text
ENV variable:
NOT_FOUND

Output variable:
3.0.0
```

---

# The Rule Every DevOps Engineer Remembers

### Same Job?

Use:

```bash
$GITHUB_ENV
```

Example:

```text
Build Version
Docker Tag
Artifact Path
```

---

### Different Job?

Use:

```bash
$GITHUB_OUTPUT
```

and

```yaml
needs:
```

Example:

```text
Build Job
↓
Test Job
↓
Deploy Job
```

