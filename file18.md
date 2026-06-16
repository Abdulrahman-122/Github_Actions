This lesson is actually one of the most important GitHub Actions lessons because it explains **how jobs and steps communicate with GitHub itself**.

Think of Workflow Commands as a special language between your script and the GitHub Runner.

Normally:

```bash
echo "Hello"
```

just prints text.

But:

```bash
echo "::error::Something failed"
```

is interpreted by GitHub as:

> "Create an error annotation in the Actions UI."

---

# Mental Model

Imagine your workflow is talking to GitHub.

Normal text:

```bash
echo "Hello"
```

means:

```text
Just print this
```

Workflow command:

```bash
echo "::warning::Disk almost full"
```

means:

```text
GitHub, create a warning message
```

---

# Lab 1: Error Annotation

Workflow:

```yaml
name: error_demo

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - run: |
          echo "::error::Database connection failed"
```

Result:

```text
❌ Database connection failed
```

appears in the GitHub UI.
https://github.com/Abdulrahman-122/Github_actions_projects/actions/runs/27600679213/job/81600644915
---

## More Detailed Error

```bash
echo "::error file=app.py,line=20::Syntax Error"
```

Result:

```text
❌ Syntax Error
File: app.py
Line: 20
```

GitHub even creates clickable links.

---

# Lab 2: Warning Annotation

```yaml
- run: |
    echo "::warning::Low disk space"
```

Result:

```text
⚠️ Low disk space
```

---

# Lab 3: Notice Annotation

```yaml
- run: |
    echo "::notice::Deployment started"
```

Result:

```text
ℹ️ Deployment started
```

Useful for status messages.

---

# Lab 4: Grouping Logs

Without grouping:

```bash
Installing packages
Downloading dependencies
Building project
Running tests
```

Logs become huge.

---

Use:

```bash
echo "::group::Install Dependencies"

echo "Installing package A"
echo "Installing package B"

echo "::endgroup::"
```

GitHub creates a collapsible section.

Result:

```text
▶ Install Dependencies
```

User clicks to expand.

---

Example:

```yaml
- run: |
    echo "::group::Docker Build"

    docker build .

    echo "::endgroup::"
```

Very common in real CI/CD.

---

# Lab 5: Debug Logs

```bash
echo "::debug::Current directory is $(pwd)"
```

By default:

```text
Hidden
```

GitHub doesn't show it.

---

If repository secret:

```text
ACTIONS_STEP_DEBUG=true
```

then:

```text
Debug: Current directory is /home/runner
```

appears.

---

This is useful for troubleshooting.

---

# Lab 6: Passing Data Between Steps

This is the most important workflow command.

Suppose Step 1 calculates a version:

```yaml
- name: Generate version
  run: |
    echo "VERSION=1.0.0"
```

Problem:

Step 2 can't see it.

Every step runs in its own shell.

---

Wrong:

```yaml
- name: Step 1
  run: |
    VERSION=1.0.0

- name: Step 2
  run: |
    echo $VERSION
```

Output:

```text
(blank)
```

---

# Solution: GITHUB_ENV

Step 1:

```yaml
- run: |
    echo "VERSION=1.0.0" >> $GITHUB_ENV
```

Step 2:

```yaml
- run: |
    echo $VERSION
```

Output:

```text
1.0.0
```

---

Think:

```text
Current Step
     ↓
GITHUB_ENV
     ↓
Future Steps
```

---

# Lab 7: Outputs

Sometimes you want:

```text
Step A
 ↓
Step B
```

communication.

Use:

```bash
echo "version=1.0.0" >> $GITHUB_OUTPUT
```

---

Example:

```yaml
- name: Create output
  id: version
  run: |
    echo "version=1.0.0" >> $GITHUB_OUTPUT
```

---

Next step:

```yaml
- run: |
    echo "${{ steps.version.outputs.version }}"
```

Output:

```text
1.0.0
```

---

Visual:

```text
Step A
  |
  |---> GITHUB_OUTPUT
  |
Step B
```

---

# Lab 8: GITHUB_STATE

Less commonly used.

Stores information for:

```text
pre
main
post
```

action lifecycle.

Mostly useful when writing custom actions.

---

Example:

```bash
echo "temp_file=/tmp/test.txt" >> $GITHUB_STATE
```

Later:

```bash
STATE_temp_file
```

can be accessed by the post-action.

---

# Real DevOps Example

Suppose you're building a Docker image.

Step 1:

```yaml
- name: Generate tag
  id: tag
  run: |
    echo "image_tag=v1.5.2" >> $GITHUB_OUTPUT
```

Step 2:

```yaml
- name: Build image
  run: |
    docker build -t myapp:${{ steps.tag.outputs.image_tag }} .
```

Step 3:

```yaml
- name: Push image
  run: |
    docker push myapp:${{ steps.tag.outputs.image_tag }}
```

This pattern is used everywhere.

---

# Quick Cheat Sheet

| Command          | Purpose                                   |
| ---------------- | ----------------------------------------- |
| `::error::`      | Create error                              |
| `::warning::`    | Create warning                            |
| `::notice::`     | Create info message                       |
| `::debug::`      | Debug output                              |
| `::group::`      | Start collapsible log section             |
| `::endgroup::`   | End section                               |
| `$GITHUB_ENV`    | Share environment variables between steps |
| `$GITHUB_OUTPUT` | Share outputs between steps               |
| `$GITHUB_STATE`  | Store action state                        |

For a DevOps engineer, the two workflow commands you'll use constantly are:

```bash
echo "VAR=value" >> $GITHUB_ENV
```

and

```bash
echo "result=value" >> $GITHUB_OUTPUT
```

because nearly every non-trivial CI/CD pipeline passes data between steps this way.
