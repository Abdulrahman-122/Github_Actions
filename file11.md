This is one of the most powerful GitHub Actions topics. Let's build it piece by piece like a lab.

---

# 1. What is a Matrix?

Without a matrix, if you want to test:

* Python 3.10
* Python 3.11
* Python 3.12

you might write:

```yaml
jobs:
  py310:
    ...

  py311:
    ...

  py312:
    ...
```

Lots of duplicated YAML.

---

With a matrix:

```yaml
strategy:
  matrix:
    python: [3.10, 3.11, 3.12]
```

GitHub automatically creates:

```text
Job 1 -> Python 3.10
Job 2 -> Python 3.11
Job 3 -> Python 3.12
```

from one job definition.

---

# Lab 1 — Basic Matrix

Create:

```yaml
name: matrix_basic

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        language:
          - python
          - node
          - go

    steps:
      - run: echo "Language = ${{ matrix.language }}"
```

---

## What happens?

GitHub creates 3 jobs:

```text
demo (python)
demo (node)
demo (go)
```

Each runs separately.

Output:

```text
Language = python
```

```text
Language = node
```

```text
Language = go
```

---

# Multiple Matrix Variables

Now:

```yaml
strategy:
  matrix:
    number: [1,2]
    letter: [a,b]
```

GitHub generates every combination.

---

## Result

```text
1a
1b
2a
2b
```

4 jobs total.

---

# Lab 2 — Matrix Combinations

```yaml
name: matrix_combinations

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        number: [1,2]
        letter: [a,b]

    steps:
      - run: |
          echo "${{ matrix.number }}"
          echo "${{ matrix.letter }}"
```

Run it.

Observe 4 jobs.

---

# What is Exclude?

Suppose:

```text
1a
1b
1c
2a
2b
2c
```

But you don't want:

```text
1c
```

Use:

```yaml
exclude:
  - number: 1
    letter: c
```

---

GitHub creates:

```text
1a
1b
2a
2b
2c
```

and skips:

```text
1c
```

---

# Lab 3 — Exclude

```yaml
name: matrix_exclude

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        number: [1,2]
        letter: [a,b,c]

        exclude:
          - number: 1
            letter: c

    steps:
      - run: |
          echo "${{ matrix.number }}${{ matrix.letter }}"
```

Count the jobs.

You should see:

```text
1a
1b
2a
2b
2c
```

---

# What is if: ?

Conditionals.

Think:

```python
if x > 10:
    print("Hello")
```

GitHub equivalent:

```yaml
if: matrix.number == 2
```

---

# Lab 4 — Conditional Step

```yaml
name: matrix_if

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        number: [1,2]

    steps:
      - run: echo "Always runs"

      - if: matrix.number == 2
        run: echo "Only number 2 sees this"
```

---

Output:

Job 1:

```text
Always runs
```

Job 2:

```text
Always runs
Only number 2 sees this
```

---

# Understanding This Line

```yaml
if: ${{ !(matrix.number == 2 && matrix.letter == 'c') }}
```

means:

```text
Run everywhere
EXCEPT
2c
```

---

Truth table:

| Combination | Runs? |
| ----------- | ----- |
| 1a          | Yes   |
| 1b          | Yes   |
| 1c          | Yes   |
| 2a          | Yes   |
| 2b          | Yes   |
| 2c          | No    |

---

# What is fail-fast?

Example:

```yaml
strategy:
  fail-fast: true
```

Suppose:

```text
Job A running
Job B running
Job C running
```

Job B fails.

GitHub immediately cancels:

```text
Job A
Job C
```

to save time.

---

Without fail-fast:

```yaml
fail-fast: false
```

GitHub lets all jobs finish.

---

# What is timeout-minutes?

Suppose somebody writes:

```bash
sleep 999999
```

Workflow never ends.

Prevent that:

```yaml
timeout-minutes: 5
```

GitHub kills it after 5 minutes.

---

# Concurrency

This is extremely useful.

Imagine:

You push:

```text
commit A
```

Workflow starts.

30 seconds later:

```text
commit B
```

Workflow starts again.

Now two workflows are running.

---

Problem:

```text
A is outdated
B is newer
```

Why waste money running A?

---

Use:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

---

Result:

```text
Run #1 starts
```

Push again:

```text
Run #2 starts
```

GitHub automatically:

```text
Cancels Run #1
Keeps Run #2
```

---

# Lab 5 — Concurrency

Create:

```yaml
name: concurrency_demo

on:
  workflow_dispatch:

concurrency:
  group: demo-group
  cancel-in-progress: true

jobs:
  sleep:
    runs-on: ubuntu-latest

    steps:
      - run: |
          echo "Sleeping..."
          sleep 120
```

---

### Test

Run workflow manually.

While it's sleeping:

```text
Actions
→ Run workflow
```

again.

Watch:

```text
Run #1 → Cancelled
Run #2 → Running
```

---

# Real DevOps Example

Suppose you deploy your Flask app.

Bad:

```text
Commit A deploying
Commit B deploying
Commit C deploying
```

Three deployments running simultaneously.

---

Better:

```yaml
concurrency:
  group: production
  cancel-in-progress: true
```

Result:

```text
Commit A cancelled
Commit B cancelled
Commit C deployed
```

Only the newest code reaches production.

---

## Final Mental Model

### Matrix

```text
One job definition
→ many jobs
```

### Exclude

```text
Remove specific combinations
```

### if

```text
Run only when condition is true
```

### fail-fast

```text
One matrix job fails
→ cancel the others
```

### timeout-minutes

```text
Kill stuck jobs
```

### concurrency

```text
Cancel old workflow runs
Keep newest one
```

These six features are used constantly in professional CI/CD pipelines because they reduce duplication, save runner minutes, and prevent deploying outdated code.
