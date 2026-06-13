Absolutely. This lesson is about **where your GitHub Actions jobs actually run**.

Many beginners think:

```text
GitHub Actions
     ↓
runs magically somewhere
```

But every job needs a machine (runner).

---

# Mental Model

When you write:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

GitHub does roughly:

```text
1. Create Ubuntu VM
2. Clone your repo
3. Run your commands
4. Destroy VM
```

You don't see the machine, but it exists.

---

# LAB 1: Discover Your Runner

Create:

```yaml
name: runner-lab

on:
  workflow_dispatch:

jobs:
  info:
    runs-on: ubuntu-latest

    steps:
      - name: Runner Info
        run: |
          echo "OS: ${{ runner.os }}"
          echo "ARCH: ${{ runner.arch }}"
          echo "NAME: ${{ runner.name }}"
```

Run it.

---

## What should you see?

Something like:

```text
OS: Linux
ARCH: X64
NAME: GitHub Actions 12
```

---

## What are these?

### runner.os

Operating system:

```text
Linux
Windows
macOS
```

---

### runner.arch

CPU architecture:

```text
X64
ARM64
```

---

### runner.name

Machine name assigned by GitHub.

Example:

```text
GitHub Actions 7
```

---

# LAB 2: Inspect the VM

Create:

```yaml
name: inspect-runner

on:
  workflow_dispatch:

jobs:
  inspect:
    runs-on: ubuntu-latest

    steps:
      - run: |
          uname -a
          pwd
          whoami
          df -h
```

---

## What are these commands?

### uname -a

Shows Linux information.

Example:

```text
Linux fv-az123...
```

---

### pwd

Shows current directory.

Example:

```text
/home/runner/work/repo/repo
```

---

### whoami

Shows current user.

Example:

```text
runner
```

---

### df -h

Shows disk space.

Example:

```text
84G available
```

---

# LAB 3: Ubuntu vs Windows

Create:

```yaml
name: compare-runners

on:
  workflow_dispatch:

jobs:

  ubuntu:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Hello from Linux"

  windows:
    runs-on: windows-latest

    steps:
      - shell: pwsh
        run: |
          echo "Hello from Windows"
```

Run it.

---

## What happens?

GitHub creates:

```text
Job 1:
Ubuntu VM

Job 2:
Windows VM
```

They run independently.

---

# Important Concept

These jobs DO NOT share data.

```text
Ubuntu Job
      X
Windows Job
```

If Ubuntu creates:

```text
file.txt
```

Windows cannot see it.

Different machines.

---

# LAB 4: See the Difference

Ubuntu:

```yaml
run: |
  uname -a
```

Windows:

```yaml
shell: pwsh
run: |
  Get-ComputerInfo
```

Notice:

```text
Linux commands work on Linux
PowerShell commands work on Windows
```

---

# LAB 5: Container Runner

This is the coolest part.

Create:

```yaml
name: container-lab

on:
  workflow_dispatch:

jobs:

  alpine:
    runs-on: ubuntu-latest

    container:
      image: alpine:3.20

    steps:
      - run: |
          cat /etc/os-release
```

---

# What is happening?

Normally:

```text
GitHub Ubuntu VM
      ↓
Run commands directly
```

Now:

```text
GitHub Ubuntu VM
      ↓
Start Docker container
      ↓
Run commands INSIDE container
```

---

## Visual

```text
Ubuntu VM
└── Alpine Container
      └── Your commands
```

---

## Output

You should see:

```text
Alpine Linux v3.20
```

Even though the host machine is Ubuntu.

---

# LAB 6: Compare Host vs Container

Create:

```yaml
name: compare-container

on:
  workflow_dispatch:

jobs:

  normal:
    runs-on: ubuntu-latest

    steps:
      - run: cat /etc/os-release

  alpine:
    runs-on: ubuntu-latest

    container:
      image: alpine:3.20

    steps:
      - run: cat /etc/os-release
```

---

## Expected Result

Job 1:

```text
Ubuntu 24.04
```

Job 2:

```text
Alpine Linux
```

---

# Why Use Containers?

Imagine your app needs:

```text
Python 3.13
Node 24
Terraform
kubectl
```

Instead of installing everything every run:

```yaml
container:
  image: my-devops-image:latest
```

Now everything is preinstalled.

Builds become faster and reproducible.

---

# LAB 7: Real DevOps Example

Pretend you're building a Flask app.

```yaml
name: flask-build

on:
  workflow_dispatch:

jobs:

  build:

    runs-on: ubuntu-latest

    container:
      image: python:3.12

    steps:
      - uses: actions/checkout@v4

      - run: |
          python --version
          pip --version
```

---

## Why?

Without container:

```text
GitHub VM
  Python version may change over time
```

With container:

```text
python:3.12
```

Always the same environment.

---

# What About Self-Hosted Runners?

Imagine you own a server:

```text
192.168.1.10
```

Install GitHub Runner:

```text
GitHub
   ↓
Your Server
```

Now:

```yaml
runs-on: self-hosted
```

Jobs run on YOUR machine.

---

Used for:

```text
Private databases
Internal networks
Large build servers
GPU workloads
Kubernetes clusters
```

---

# Final Challenge

Create one workflow with 3 jobs:

```yaml
ubuntu
windows
alpine-container
```

For each job print:

```bash
runner.os
runner.arch
runner.name
```

and for the container job also print:

```bash
cat /etc/os-release
```

Then compare:

```text
Host OS
Container OS
Runner Metadata
```
```yaml
name: Build_VM_linux_windows_container
on:
  push:
    branches:
      - main
jobs:
  linux_machine:
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "${{ runner.os }}"
          echo "${{ runner.arch }}"
          echo "${{ runner.name }}"
  widnows_machine:
    runs-on: windows-latest
    steps:
      - shell: pwsh
        run: |
          echo "${{ runner.os }}"
          echo "${{ runner.arch }}"
          echo "${{ runner.name }}"
  container_machine:
    runs-on: ubuntu-latest
    container: 
      image: alpine:3.20
    steps:
      - run: |
          echo "${{ runner.os }}"
          echo "${{ runner.arch }}"
          echo "${{ runner.name }}"
          cat /etc/os-release
```
Once you've done that, you'll understand the relationship between:

```text
GitHub Actions
      ↓
Runner
      ↓
VM
      ↓
Container
```


