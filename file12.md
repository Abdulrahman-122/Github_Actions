This lesson is less about learning new YAML syntax and more about learning **how to find and trust Actions written by other people**.

Think of the Marketplace as:

> PyPI for Python packages
> npm for JavaScript packages
> Docker Hub for Docker images
> Marketplace for GitHub Actions

---

# Lab 1 — Your First Marketplace Action

## Goal

Use a Marketplace Action instead of writing everything yourself.

Create:

`.github/workflows/file29.yml`

```yaml
name: marketplace_lab

on:
  workflow_dispatch:

jobs:
  checkout_demo:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - run: |
          pwd
          ls -la
```

---

## What is happening?

Without:

```yaml
uses: actions/checkout@v5
```

the runner starts with an empty workspace.

There is no repository code.

GitHub gives you a VM, not your files.

---

### Experiment

Remove:

```yaml
- uses: actions/checkout@v5
```

Run again.

Notice:

```bash
ls -la
```

shows almost nothing.

Now add it back.

The repository appears.

This teaches why checkout is needed.

---

# Lab 2 — Explore an Action

Open:

```text
https://github.com/marketplace?type=actions
```

Search:

```text
setup-node
```

Open:

```text
actions/setup-node
```

Read:

* Description
* Inputs
* Example usage

You will see:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
```

The README teaches you how to use it.

That's how almost every Action is learned.

---

# Lab 3 — Use setup-node

Create:

```yaml
name: setup_node_lab

on:
  workflow_dispatch:

jobs:
  node:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/setup-node@v4

        with:
          node-version: 20

      - run: node --version
      - run: npm --version
```

---

## What happened?

Without installing Node:

```bash
node --version
```

may not be the version you want.

Marketplace action installs/configures it.

Just like:

```bash
sudo pacman -S nodejs
```

but automatically inside the runner.

---

# Lab 4 — Use Cache Action

You already learned this one.

```yaml
- uses: actions/cache@v4
```

This action:

```text
Saves files
Restores files
```

between workflow runs.

You didn't write any cache logic yourself.

Marketplace Action did it.

---

# Lab 5 — Use Upload Artifact Action

Create:

```yaml
name: artifact_lab

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: |
          echo "Hello World" > app.txt

      - uses: actions/upload-artifact@v4

        with:
          name: app
          path: app.txt
```

---

### What happened?

The action:

```yaml
uses: actions/upload-artifact@v4
```

uploads your file to GitHub storage.

You don't need to code ZIP creation or storage yourself.

---

# Lab 6 — Understand Why Actions Are Dangerous

Imagine this:

```yaml
- uses: random-user/super-action@v1
```

Questions:

* Who wrote it?
* Is it maintained?
* Does it steal secrets?
* Does it modify your repository?

Actions execute code on your runner.

They can access:

```text
GITHUB_TOKEN
Secrets
Repository files
Artifacts
```

if permissions allow.

Therefore:

Never blindly trust actions.

---

# Lab 7 — Pinning Versions (Most Important Security Lab)

Bad:

```yaml
- uses: actions/checkout@v5
```

Why?

Because someday:

```text
v5
```

could point to a different commit.

---

Better:

```yaml
- uses: actions/checkout@v5.0.0
```

---

Best:

```yaml
- uses: actions/checkout@ff7abcd0c3c05ccf6adc123a8cd1fd4fb30fb493
```

Git commit hashes never change.

---

## Real-World Analogy

Bad:

```bash
pip install flask
```

You don't know what version you'll get later.

Better:

```bash
pip install flask==3.1.0
```

Same idea.

---

# Lab 8 — Inspect Action Source Code

Open the repository for:

[actions/checkout repository](https://github.com/actions/checkout?utm_source=chatgpt.com)

Look at:

```text
action.yml
```

This file tells GitHub:

* Inputs
* Outputs
* Entrypoint

Every Action has one.

---

# Final Mental Model

When you see:

```yaml
steps:
  - uses: actions/checkout@v5

  - uses: actions/setup-node@v4

  - uses: actions/cache@v4

  - run: npm test
```

Read it as:

```text
Step 1:
Download somebody's automation code (checkout)

Step 2:
Run it

Step 3:
Download another automation code (setup-node)

Step 4:
Run it

Step 5:
Download another automation code (cache)

Step 6:
Run it

Step 7:
Run my own shell command
```

The biggest takeaway from this lesson is:

1. Find reusable Actions in the Marketplace.
2. Read the README before using them.
3. Prefer well-maintained or official Actions.
4. Pin to commit SHAs in production workflows.
5. Treat Actions like third-party dependencies because that's exactly what they are.
