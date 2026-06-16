Excellent. Container Actions are the most powerful type of GitHub Action because you can package **any language**, **any dependency**, and **any OS tools** inside Docker.

Think of it like this:

| Action Type       | Runs Where?      | Best For                       |
| ----------------- | ---------------- | ------------------------------ |
| Composite Action  | Runner shell     | Simple Bash steps              |
| Reusable Workflow | Entire workflow  | Standard CI/CD pipelines       |
| JavaScript Action | Node.js          | GitHub API integrations        |
| Container Action  | Docker container | Python, Go, Rust, custom tools |

---

#  What Problem Does Container Action Solve?

Suppose you want an action that:

```python
print("Hello Abdulrahman")
```

using Python.

A Composite Action would require:

```yaml
- run: python script.py
```

But what if the runner doesn't have:

```bash
python
pip
flask
numpy
postgres drivers
```

installed?

Container Actions solve that.

You package everything in Docker:

```Dockerfile
FROM python:3.12

COPY entrypoint.py .

ENTRYPOINT ["python","entrypoint.py"]
```

Now GitHub downloads the container and runs it.

No dependency problems.

---

#  Anatomy of a Container Action

A Container Action normally contains:

```text
python-container-action/
│
├── action.yml
├── Dockerfile
└── entrypoint.py
```

---

## action.yml

This is the action definition.

```yaml
name: Hello (Python)

inputs:
  who-to-greet:
    required: true

outputs:
  greeting:

runs:
  using: docker
  image: Dockerfile

  args:
    - ${{ inputs.who-to-greet }}
```

---

### What is happening?

GitHub reads:

```yaml
using: docker
```

and understands:

> "This action runs inside Docker."

---

Then:

```yaml
image: Dockerfile
```

means:

> Build Docker image from Dockerfile.

Equivalent to:

```bash
docker build .
```

---

Then:

```yaml
args:
  - ${{ inputs.who-to-greet }}
```

means:

```bash
python entrypoint.py Abdulrahman
```

---

#  Dockerfile

Example:

```Dockerfile
FROM python:3.12

COPY entrypoint.py .

ENTRYPOINT ["python","entrypoint.py"]
```

---

When GitHub runs:

```yaml
uses: ./python-container-action
```

it automatically does:

```bash
docker build .
docker run image
```

behind the scenes.

---

#  Understanding entrypoint.py

```python
#!/usr/bin/env python3

import os
import datetime as dt
```

Load Python libraries.

---

Read input:

```python
name = os.getenv("INPUT_WHO_TO_GREET", "World")
```

GitHub automatically converts:

```yaml
who-to-greet
```

into:

```bash
INPUT_WHO_TO_GREET
```

inside the container.

---

If workflow says:

```yaml
with:
  who-to-greet: Abdulrahman
```

then:

```python
name
```

becomes:

```text
Abdulrahman
```

---

#  Print Something

```python
greeting = f"Hello, {name}"
print(greeting)
```

Output:

```text
Hello Abdulrahman
```

appears in Actions logs.

---

#  GitHub Workflow Commands

This line:

```python
print(f"::notice::{greeting}")
```

creates a GitHub annotation.

Log:

```text
Notice: Hello Abdulrahman
```

shows in the UI.

---

GitHub recognizes special syntax:

```text
::notice::
::warning::
::error::
```

---

Example:

```python
print("::warning::Disk is almost full")
```

produces:

⚠️ Warning in Actions UI.

---

#  Outputs

This is the most important part.

```python
with open(os.environ["GITHUB_OUTPUT"], "a") as fh:
    fh.write(f"greeting={greeting}\n")
```

GitHub provides a special file:

```bash
$GITHUB_OUTPUT
```

Anything written there becomes an output.

---

Suppose:

```python
greeting = "Hello Abdulrahman"
```

then:

```text
greeting=Hello Abdulrahman
```

is written.

GitHub stores it.

---

#  Consuming the Output

Workflow:

```yaml
- name: Run container action
  id: hello
  uses: ./python-container-action
  with:
    who-to-greet: Abdulrahman
```

---

GitHub saves:

```yaml
steps.hello.outputs.greeting
```

---

Later:

```yaml
- run: echo "${{ steps.hello.outputs.greeting }}"
```

Output:

```text
Hello Abdulrahman
```

---

#  Dynamic Build vs Prebuilt Image

## Option 1

```yaml
image: Dockerfile
```

GitHub builds every run.

```text
docker build .
docker run ...
```

Pros:

* Easy

Cons:

* Slow

---

## Option 2

```yaml
image: docker://ghcr.io/company/my-action:v1
```

GitHub pulls image:

```bash
docker pull ghcr.io/company/my-action:v1
```

instead of building.

Pros:

* Much faster

Cons:

* Need registry

---

#  What Happens During Execution?

When workflow runs:

```yaml
- uses: ./python-container-action
```

GitHub performs:

### Step 1

Read:

```yaml
action.yml
```

---

### Step 2

See:

```yaml
using: docker
```

---

### Step 3

Build image:

```Dockerfile
FROM python:3.12
```

---

### Step 4

Start container:

```bash
docker run
```

---

### Step 5

Inject inputs:

```bash
INPUT_WHO_TO_GREET=Abdulrahman
```

---

### Step 6

Run:

```python
entrypoint.py
```

---

### Step 7

Capture outputs:

```bash
GITHUB_OUTPUT
```

---

### Step 8

Expose outputs to later steps:

```yaml
steps.hello.outputs.greeting
```

---

# When Should You Use Container Actions?

For a DevOps/Python engineer, Container Actions are ideal when:

* Python automation
* Flask deployment helpers
* Kubernetes utilities
* Docker build tooling
* Terraform wrappers
* PostgreSQL backup actions
* Security scanners
* Custom CLI tools

In real companies, you'll probably write:

* Composite Actions → most often
* Reusable Workflows → very often
* Container Actions → sometimes
* JavaScript Actions → rarely unless building GitHub integrations

---
Here are the **complete working files** for your lab. You can copy-paste them directly and it will work as a real GitHub Container Action.

---

#  1. `entrypoint.py`

```python
import os
from datetime import datetime, timezone

# Read input from GitHub Actions
name = os.getenv("INPUT_WHO_TO_GREET", "World")

# Get current UTC time
now = datetime.now(timezone.utc)
utc_time = now.strftime("%Y-%m-%d %H:%M:%S UTC")

# Build output message
greeting = f"Hello {name}, current UTC time is {utc_time}"

# Print GitHub notice
print(f"::notice::Greeting generated successfully")

# Write output to GitHub Actions output file
with open(os.environ["GITHUB_OUTPUT"], "a") as f:
    f.write(f"greeting={greeting}\n")
```

---

# 🐳 2. `Dockerfile`

```dockerfile
FROM python:3.12

WORKDIR /app

COPY entrypoint.py .

# Make sure script runs
ENTRYPOINT ["python", "entrypoint.py"]
```

---

# 3. `action.yml`

```yaml
name: Python UTC Greeting Action

description: Generates a greeting message with current UTC time

inputs:
  who-to-greet:
    description: Name of the person to greet
    required: true
    default: World

outputs:
  greeting:
    description: The generated greeting message

runs:
  using: docker
  image: Dockerfile
  args:
    - ${{ inputs.who-to-greet }}
```

---

#  4. Example Workflow (`.github/workflows/test.yml`)

```yaml
name: Test Container Action

on:
  push:

jobs:
  test-action:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Run Python Container Action
        id: greet
        uses: ./python-container-action
        with:
          who-to-greet: Abdulrahman

      - name: Print output
        run: echo "${{ steps.greet.outputs.greeting }}"
```

---

#  What you should see in logs

### GitHub notice:

```
::notice::Greeting generated successfully
```

### Output step:

```
Hello Abdulrahman, current UTC time is 2026-06-16 12:30:00 UTC
```


