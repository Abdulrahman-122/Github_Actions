This is one of the most useful GitHub Actions labs because it teaches you how to move **files** between jobs.

---

# Why do we need artifacts?

Remember:

```yaml
jobs:
  build:
```

runs on a VM.

When the job finishes:

```text
VM destroyed
All files deleted
```

So if Job 1 creates:

```text
app.zip
report.txt
coverage.html
```

Job 2 cannot see them.

That's where artifacts come in.

Think of artifacts as:

```text
Job 1
   ↓ upload
GitHub Storage
   ↓ download
Job 2
```

---

# Lab 1: Your First Artifact

Create a new workflow:

```yaml
name: artifact_lab

on:
  workflow_dispatch:

jobs:

  producer:
    runs-on: ubuntu-latest

    steps:

      - name: Create file
        run: |
          echo "Hello Abdulrahman" > info.txt
          date >> info.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: my-file
          path: info.txt

  consumer:
    runs-on: ubuntu-latest

    needs: producer

    steps:

      - name: Download artifact
        uses: actions/download-artifact@v5
        with:
          name: my-file

      - name: Read file
        run: |
          cat info.txt
```

---

# What happens?

## Producer Job

Creates:

```text
info.txt
```

Contents:

```text
Hello Abdulrahman
Sat Jun 13 ...
```

Then:

```yaml
uses: actions/upload-artifact@v4
```

uploads it to GitHub storage.

---

## Consumer Job

Waits because of:

```yaml
needs: producer
```

Then:

```yaml
uses: actions/download-artifact@v5
```

downloads the file.

Now:

```bash
cat info.txt
```

works.

---

# Visual Flow

```text
Producer Job

info.txt
    │
    ▼
upload-artifact
    │
    ▼
GitHub Artifact Storage
    │
    ▼
download-artifact
    │
    ▼
Consumer Job
```

---

# Lab 2: Simulate a Build

Pretend you're building a Flask application.

```yaml
name: flask_artifact

on:
  workflow_dispatch:

jobs:

  build:
    runs-on: ubuntu-latest

    steps:

      - name: Create package
        run: |
          mkdir dist
          echo "Flask Application" > dist/app.txt

      - name: Upload package
        uses: actions/upload-artifact@v4
        with:
          name: flask-package
          path: dist/

  deploy:
    runs-on: ubuntu-latest

    needs: build

    steps:

      - name: Download package
        uses: actions/download-artifact@v5
        with:
          name: flask-package

      - name: Verify package
        run: |
          ls -R
          cat app.txt
```

---

# Where do I see the artifact?

After the workflow finishes:

1. Open your fork on GitHub.
2. Click **Actions**.
3. Open the workflow run.
4. Scroll down.

You'll see something like:

```text
Artifacts
└── flask-package
```

Click it and GitHub downloads a ZIP file.

This is commonly used for:

```text
Python wheels
Docker build reports
Test results
Coverage reports
Compiled binaries
React build folders
```

---

# Important Difference

You already learned:

### `$GITHUB_OUTPUT`

Passes:

```text
Strings
Numbers
IDs
Versions
```

Example:

```text
version=1.2.0
```

---

### Artifacts

Pass:

```text
Actual files
Directories
Archives
Reports
```

Example:

```text
app.zip
report.html
coverage.xml
dist/
```

A good rule:

```text
Need to pass text/value?
→ GITHUB_OUTPUT

Need to pass file/folder?
→ Artifact
```

---

# Challenge Lab

Try building this yourself:

```text
Job 1:
- Create file called student.txt
- Put your name in it
- Upload as artifact

Job 2:
- Download artifact
- Print its contents
- Show ls -la
```
```yaml
name: Create_artifact
on:
  workflow_dispatch:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: create_file
        run: |
          echo "Hello from yml ">>student.txt
          echo $date >> student.txt
      - name: upload_filee
        uses: actions/upload-artifact@v4
        with:
          name: Stu_file
          path: student.txt
  deploy:
    runs-on: ubuntu-latest
    needs:
      - build
    steps:
      - name: Download_file
        uses: actions/download-artifact@v5
        with:
          name: Stu_file
      - name: more_info
        run: |
          uname -a
          ls -R
          echo "Operation done"

  

```




If you can do that without looking at my solution, you've understood artifacts correctly.
