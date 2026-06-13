Excellent. You're now reaching one of the most important GitHub Actions concepts after artifacts.

The key difference is:

| Feature  | Purpose                                   |
| -------- | ----------------------------------------- |
| Artifact | Pass files between jobs or download later |
| Cache    | Make future workflow runs faster          |

---

# 🧠 Why do we need Cache?

Imagine this workflow:

```yaml
steps:
  - uses: actions/checkout@v5

  - run: npm install
```

Every run:

```text
Download package A
Download package B
Download package C
...
```

Suppose it takes:

```text
3 minutes
```

every push.

That becomes annoying.

Cache allows GitHub to save downloaded dependencies and reuse them later.

---

# Lab 1 — Build Your Own Cache

Create:

```yaml
name: Cache_Lab

on:
  workflow_dispatch:

jobs:
  Cache_Test:
    runs-on: ubuntu-latest

    steps:

      - name: Restore Cache
        id: cache-demo
        uses: actions/cache@v4
        with:
          path: demo-cache
          key: my-cache-v1

      - name: Generate Data
        if: steps.cache-demo.outputs.cache-hit != 'true'
        run: |
          echo "Cache Miss!"
          mkdir -p demo-cache
          date > demo-cache/time.txt

      - name: Save Cache
        if: steps.cache-demo.outputs.cache-hit != 'true'
        uses: actions/cache@v4
        with:
          path: demo-cache
          key: my-cache-v1

      - name: Verify
        run: |
          echo "Cache hit? -> ${{ steps.cache-demo.outputs.cache-hit }}"
          cat demo-cache/time.txt
```

---

# First Run

Click:

```text
Actions
→ Cache_Lab
→ Run workflow
```

Output:

```text
Cache hit? -> false
Cache Miss!
```

Because:

```text
No cache exists yet
```

---

# Second Run

Run again.

Now:

```text
Cache hit? -> true
```

And the timestamp stays the same.

Example:

```text
Sat Jun 13 15:00:00 UTC 2026
```

even though the workflow ran later.

---

# What happened?

First run:

```text
Create directory
Save cache
```

GitHub stores:

```text
demo-cache/
```

using key:

```text
my-cache-v1
```

---

Second run:

GitHub sees:

```text
my-cache-v1
```

already exists.

So it restores:

```text
demo-cache/
```

before your script executes.

---

# Understanding the Key

This is the most important part:

```yaml
key: my-cache-v1
```

The cache is identified by its key.

Think:

```text
key = cache name
```

---

If you change:

```yaml
key: my-cache-v2
```

GitHub treats it as:

```text
brand new cache
```

and you'll get:

```text
cache-hit -> false
```

again.

---

# Lab 2 — Force a New Cache

Change:

```yaml
key: my-cache-v1
```

to:

```yaml
key: my-cache-v2
```

Push.

Run again.

Observe:

```text
cache-hit -> false
```

because GitHub cannot find `my-cache-v2`.

---

# Why npm Caching Exists

Without cache:

```yaml
npm install
```

downloads everything every run.

With:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: npm
```

GitHub automatically saves:

```text
~/.npm
```

and restores it later.

---

# Lab 3 — Observe Automatic Cache

Create:

```yaml
name: Node_Cache

on:
  workflow_dispatch:

jobs:
  Node:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm --version
```

Run twice.

Look at logs.

You'll see messages about:

```text
Restoring cache
Saving cache
```

---

# Real Flask Example

Suppose your project installs:

```text
Flask
SQLAlchemy
Gunicorn
Requests
```

every workflow.

Instead of:

```yaml
pip install -r requirements.txt
```

from scratch every run,

you cache:

```yaml
~/.cache/pip
```

Example:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: pip-cache-v1
```

Next runs become much faster.

---

# Artifact vs Cache Lab

Create this mental picture:

## Artifact

```text
Build Job
   |
   | upload-artifact
   ↓
GitHub Storage
   |
   | download-artifact
   ↓
Deploy Job
```

Purpose:

```text
Transfer files
```

---

## Cache

```text
Run #1
   |
   | save cache
   ↓

Run #2
   |
   | restore cache
   ↓

Run #3
   |
   | restore cache
```

Purpose:

```text
Speed up future runs
```

---

# Challenge for You

Create a workflow called:

```yaml
name: Flask_Dependency_Cache
```

that:

1. Creates a folder called:

```text
pip-cache
```

2. Saves a timestamp inside it.

3. Uses:

```yaml
uses: actions/cache@v4
```

4. Prints:

```yaml
${{ steps.cache-demo.outputs.cache-hit }}
```
```yml
name: Flask_dependency_cache
on:
  workflow_dispatch: 

jobs:
  Create_cache:
    runs-on: ubuntu-latest
    steps:
      - name: check-cache
        id: check-cache
        uses: actions/cache@v4
        with:
          path: check-cache
          key:  cache-v2
      - name: generate-cache
        if: steps.check-cache.outputs.cache-hit !='true'
        run: |
          echo "Cache Miss"
          mkdir -p check-cache
          echo "hello from flask this cache is new" >>check-cache/output.txt
          date >>check-cache/output.txt
      - name: save-cache
        if: steps.check-cache.outputs.cache-hit !='true'
        uses: actions/cache@v4
        with:
          path: check-cache
          key: cache-v2
      - name: verify-cache
        run: |
          echo "Welcome from new cache"
          echo "After doing that step you will notice the phase of creating new caches will be closed  "
          cat check-cache/output.txt
          echo "${{steps.check-cache.outputs.cache-hit}}"


```

Run it twice and tell me:

```text
What did the first run show? it run creating new cache and run not found cache so it created new one
What did the second run show? it just run the one that he created before and disable the button of creating new one
```

Then we'll move to matrices and conditional jobs, which are some of the most powerful GitHub Actions features.
