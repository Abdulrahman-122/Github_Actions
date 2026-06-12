Absolutely. Let's break down **why you're forking the repository** and then walk through the setup step-by-step.

## Why Fork the Repository?

Suppose the course repository is:

[devops-directive-github-actions-course repository](https://github.com/sidpalas/devops-directive-github-actions-course?utm_source=chatgpt.com)

You **don't own** that repository.

GitHub Actions workflows often need permissions to:

* Run CI/CD pipelines
* Create artifacts
* Use secrets
* Trigger workflow runs
* Push changes

If you only clone the original repository, you may not have permission to run everything.

When you **fork**, GitHub creates:

```
Original:
sidpalas/devops-directive-github-actions-course

Your fork:
yourusername/devops-directive-github-actions-course
```

Now **you own the repository**.

Benefits:

✅ Can run GitHub Actions workflows

✅ Can modify files freely

✅ Can create branches and pull requests

✅ Can experiment without affecting the original repo

✅ Can store your own GitHub Secrets

For a GitHub Actions course, this is extremely important because many exercises require workflow execution.

---

# Step 1: Fork the Repository

Go to:

[Course Repository](https://github.com/sidpalas/devops-directive-github-actions-course?utm_source=chatgpt.com)

Click:

```
Fork
```

(top-right corner)

GitHub creates:

```
https://github.com/YOUR_USERNAME/devops-directive-github-actions-course
```

---

# Step 2: Clone Your Fork

Copy the URL of **your fork**, not the original.

Example:

```bash
git clone --recurse-submodules \
https://github.com/abdulrahman/devops-directive-github-actions-course.git
```

Then:

```bash
cd devops-directive-github-actions-course
```

---

# Why `--recurse-submodules`?

Some repositories contain other repositories inside them.

Without:

```bash
git clone
```

those nested repos remain empty.

With:

```bash
git clone --recurse-submodules
```

Git automatically downloads them.

You can check later with:

```bash
git submodule status
```

---

# Step 3: Install Devbox

Since you're on Arch Linux (from previous discussions), install it using the official method.

The install guide is here:

[Devbox Installation Guide](https://www.jetify.com/docs/devbox/installing_devbox/?utm_source=chatgpt.com)

Verify:

```bash
devbox version
```

---

# What is Devbox?

Think of Devbox as:

```
package.json
      +
Docker-like reproducibility
      +
developer tools
```

Instead of telling students:

```
Install Go
Install Node
Install Terraform
Install Act
Install Helm
```

the course provides:

```json
{
  "packages": [...]
}
```

and Devbox installs everything automatically.

---

# Step 4: Enter the Devbox Environment

Inside the repository:

```bash
devbox shell
```

Devbox reads:

```bash
devbox.json
devbox.lock
```

and installs the required tools.

You'll see a new shell prompt.

Check what's installed:

```bash
which act
which node
which go
which terraform
```

(depending on what the course includes)

---

# Step 5: Install Docker

GitHub Actions frequently run inside containers.

Many lessons use:

* Docker images
* Docker builds
* Local workflow execution with `act`

Install:

[Docker Documentation](https://docs.docker.com/get-docker/?utm_source=chatgpt.com)

On Arch Linux you probably already have Docker, but verify:

```bash
docker version
```

Start the service:

```bash
sudo systemctl enable --now docker
```

Check:

```bash
docker run hello-world
```

---

# Step 6: Install VS Code Extensions

Install:

### GitHub Actions

Helps with:

* workflow validation
* autocomplete
* syntax hints

Extension:

[GitHub Actions Extension](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-github-actions&utm_source=chatgpt.com)

---

### YAML

Provides:

* YAML linting
* autocomplete
* schema validation

Extension:

[YAML Extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml&utm_source=chatgpt.com)

---

# Step 7: Verify Everything

Inside the project:

```bash
devbox shell
```

Then:

```bash
git --version
docker --version
devbox version
```

If the course uses Act:

```bash
act --version
```

---

# What Is `act`?

Normally:

```
Push code
 ↓
GitHub server
 ↓
Workflow runs
```

With `act`:

```
Workflow
 ↓
Runs locally
 ↓
Using Docker
```

Example:

```bash
act
```

This allows you to test workflows without pushing every time.

Project structure usually looks like:

```text
devops-directive-github-actions-course/
│
├── .github/
│   └── workflows/
│
├── devbox.json
├── devbox.lock
└── README.md
```

---

For your Arch Linux setup, after you fork and clone the repo, run:

```bash
cd devops-directive-github-actions-course

devbox shell

docker run hello-world

act --version
```

