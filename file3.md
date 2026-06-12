Some triggers you neeed to understand how workflows work??
Great idea. The best way to learn triggers is to create **one workflow per trigger** and observe when it runs.

---

# Lab 1 — Manual Trigger (`workflow_dispatch`)

You already did this one.

```yaml
name: Manual Trigger Lab

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "I was started manually"
```

### How to trigger

```text
GitHub
 ↓
Actions
 ↓
Manual Trigger Lab
 ↓
Run Workflow
```

### What you learn

* Manual execution
* Debugging
* Running deployments on demand

---

# Lab 2 — Push Trigger

Create:

```text
.github/workflows/push-lab.yml
```

```yaml
name: Push Trigger Lab

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Somebody pushed code"
      - run: echo $GITHUB_REF
      - run: echo $GITHUB_ACTOR
```

---

### Trigger it

```bash
echo "hello" > test.txt

git add .
git commit -m "Testing push trigger"
git push
```

---

### Observe

Actions tab:

```text
Push Trigger Lab
```

runs automatically.

---

# Lab 3 — Branch Filter

Create:

```yaml
name: Branch Filter Lab

on:
  push:
    branches:
      - "feature/*"

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Feature branch detected"
```

---

### Create branch

```bash
git checkout -b feature/login
```

Push:

```bash
git push -u origin feature/login
```

Workflow runs.

---

### Test failure

Switch:

```bash
git checkout main
```

Push something.

Workflow does NOT run.

---

### Lesson

```yaml
branches:
  - "feature/*"
```

means:

```text
Only run on branches beginning with feature/
```

---

# Lab 4 — Pull Request Trigger

Create:

```yaml
name: Pull Request Lab

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "A pull request was opened"
```

---

### Trigger it

Create:

```bash
git checkout -b feature/api
```

Make a change.

```bash
git push -u origin feature/api
```

Then on GitHub:

```text
Compare & Pull Request
```

Create the PR.

---

### Observe

Workflow starts automatically.

---

### Learn

Pull request events are used for:

```text
Linting
Unit Tests
Security Checks
Code Quality Gates
```

before code reaches main.

---

# Lab 5 — Path Filter

Create:

```yaml
name: Path Filter Lab

on:
  push:
    paths:
      - "docs/**"

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Documentation changed"
```

---

### Trigger

Create:

```bash
mkdir docs

echo "hello" > docs/readme.md
```

Commit and push.

Workflow runs.

---

### Now test

Create:

```bash
touch app.py
```

Commit and push.

Workflow does NOT run.

---

### Learn

Only files inside:

```text
docs/
```

cause execution.

This is heavily used in monorepos.

---

# Lab 6 — Excluding Files

```yaml
name: Path Exclusion Lab

on:
  push:
    paths:
      - "docs/**"
      - "!docs/*.txt"

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Allowed documentation file"
```

---

### Test

Runs:

```text
docs/readme.md
```

Does not run:

```text
docs/notes.txt
```

---

# Lab 7 — Schedule Trigger

Create:

```yaml
name: Schedule Lab

on:
  schedule:
    - cron: "* * * * *"

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Scheduled execution"
```

---

### Important

This means:

```text
Every minute
```

for testing.

GitHub may delay it slightly.

---

### Observe

Wait a few minutes.

Actions tab begins showing runs automatically.

No push.

No PR.

No clicking.

---

### Learn

Used for:

```text
Nightly Tests
Database Backups
Security Scans
Report Generation
```

---

# Lab 8 — Multiple Triggers

```yaml
name: Multi Trigger Lab

on:
  workflow_dispatch:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Event:"
      - run: echo $GITHUB_EVENT_NAME
```

---

### Try all three

Manual:

```text
workflow_dispatch
```

Push:

```text
push
```

PR:

```text
pull_request
```

---

### Observe

The log changes:

```bash
echo $GITHUB_EVENT_NAME
```

Outputs:

```text
push
```

or

```text
pull_request
```

or

```text
workflow_dispatch
```

depending on how it started.

---

# Lab 9 — See the Entire Event Payload

This is the most important trigger lab.

```yaml
name: Payload Lab

on:
  workflow_dispatch:
  push:
  pull_request:

jobs:
  payload:
    runs-on: ubuntu-latest

    steps:
      - run: echo "EVENT:"
      - run: echo $GITHUB_EVENT_NAME

      - run: cat $GITHUB_EVENT_PATH
```

---

Now trigger it three different ways:

### Manual Run

```text
workflow_dispatch
```

Inspect JSON.

---

### Push

```text
push
```

Inspect JSON.

---

### Pull Request

```text
pull_request
```

Inspect JSON.

---

You'll discover:

```text
Different trigger
        ↓
Different JSON payload
        ↓
Different information available
```

And that is the foundation of advanced GitHub Actions, where you deploy only when:

```yaml
if: github.ref == 'refs/heads/main'
```

or run jobs only when:

```yaml
if: github.event_name == 'pull_request'
```

Once you've completed these labs, you'll understand about 80% of the trigger system used in real DevOps CI/CD pipelines.
