This is the **first time GitHub Actions becomes "programmable" by you**, not just using other people's actions.

Think of it this way:

| Before                      | Now                             |
| --------------------------- | ------------------------------- |
| Use `actions/checkout`      | Create your own action          |
| Use `actions/cache`         | Create your own reusable action |
| Copy-paste steps everywhere | Write once, reuse everywhere    |

---

# Step 1: What is a Composite Action?

Suppose you have this in 10 workflows:

```yaml
steps:
  - run: echo "Hello Abdulrahman"
  - run: date
  - run: uname -a
```

Instead of repeating it:

```yaml
Workflow A
Workflow B
Workflow C
Workflow D
```

You package it into:

```yaml
action.yaml
```

and reuse it:

```yaml
uses: ./my-action
```

Exactly like:

```yaml
uses: actions/checkout@v5
```

except **you wrote it yourself**.

---

# Step 2: The action.yaml

Example:

```yaml
name: 'Hello World'
description: 'Greet someone'

inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'World'

runs:
  using: composite

  steps:
    - run: echo "Hello $INPUT_WHO_TO_GREET"
      shell: bash
      env:
        INPUT_WHO_TO_GREET: ${{ inputs.who-to-greet }}
```

---

## name

```yaml
name: Hello World
```

Action name.

Appears in logs.

---

## description

```yaml
description: Greet someone
```

Documentation only.

---

## inputs

```yaml
inputs:
  who-to-greet:
```

Creates a variable the caller can pass.

Like a function parameter:

Python:

```python
def hello(name):
```

Composite action:

```yaml
inputs:
  who-to-greet:
```

---

## required

```yaml
required: true
```

User must provide value.

---

## default

```yaml
default: World
```

If user doesn't pass anything:

```yaml
Hello World
```

---

# Step 3: runs

```yaml
runs:
  using: composite
```

This tells GitHub:

> This action consists of shell steps.

---

# Step 4: steps

```yaml
steps:
```

Same as workflow steps.

---

```yaml
- run: echo "Hello $INPUT_WHO_TO_GREET"
```

Print greeting.

---

# Step 5: inputs usage

```yaml
env:
  INPUT_WHO_TO_GREET: ${{ inputs.who-to-greet }}
```

Take the input:

```yaml
who-to-greet
```

and store it in:

```bash
$INPUT_WHO_TO_GREET
```

for shell usage.

---

# Step 6: Folder Structure

Imagine:

```text
repo/
│
├── .github/
│   └── workflows/
│       └── hello.yml
│
└── my-action/
    └── action.yaml
```

---

# Step 7: Calling your action

Workflow:

```yaml
jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - uses: ./my-action
        with:
          who-to-greet: Abdulrahman
```

---

## Why checkout?

This confuses many beginners.

Without:

```yaml
- uses: actions/checkout@v5
```

the runner does **not** have your repository files.

So:

```yaml
uses: ./my-action
```

fails because:

```text
./my-action/action.yaml
```

doesn't exist on the runner.

Checkout downloads the repository first.

---

# What happens internally?

Workflow starts:

```yaml
- uses: actions/checkout@v5
```

GitHub clones:

```text
repo/
```

onto runner.

Then:

```yaml
- uses: ./my-action
```

GitHub finds:

```text
repo/my-action/action.yaml
```

and executes it.

---

# Real Example

Create:

```text
.github/actions/show-system-info/action.yaml
```

```yaml
name: Show System Info

runs:
  using: composite

  steps:
    - run: uname -a
      shell: bash

    - run: date
      shell: bash
```

---

Workflow:

```yaml
name: test-action

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - uses: ./.github/actions/show-system-info
```

Output:

```text
Linux runner...
Sat Jun ...
```

---

# When should you use Composite Actions?

Use them when you have:

✅ Shell scripts

✅ Repeated commands

✅ Setup steps

✅ Build steps

Examples:

* Docker login
* Python setup
* Flask deployment
* Kubernetes deployment
* Security scans

---

# Composite Action vs Reusable Workflow

### Composite Action

```yaml
uses: ./my-action
```

Runs as a **step**.

```yaml
steps:
  - uses: ./my-action
```

---

### Reusable Workflow

```yaml
uses: ./.github/workflows/deploy.yml
```

Runs as a **job**.

```yaml
jobs:
  deploy:
    uses: ...
```

---

A good mental model:

* Composite Action = custom function
* Reusable Workflow = custom program/module

For your DevOps journey, you'll probably create composite actions for things like:

* Docker image builds
* Flask deployment
* Kubernetes deployment
* Running tests
* Security scans

because they remove a lot of duplicated YAML across projects.
