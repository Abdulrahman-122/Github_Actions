This section is one of the most important parts of modern DevOps because almost every real pipeline eventually needs to talk to something outside GitHub:

* AWS
* Azure
* Google Cloud
* Kubernetes clusters
* Docker registries
* Terraform Cloud
* Databases
* Vault

The question is:

> How does GitHub prove its identity to those systems?

---

# Before OIDC: Static Secrets

Imagine you have AWS.

You create:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside AWS IAM.

Then you save them as GitHub Secrets:

```text
Settings
  → Secrets and Variables
  → Actions
```

Example:

```yaml
steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      aws-region: us-east-1
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## What happens?

GitHub gives the workflow:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Then AWS says:

```text
✓ Credentials valid
```

and grants access.

---

# Why is this bad?

Suppose somebody steals:

```text
AWS_SECRET_ACCESS_KEY
```

They can use it:

```bash
aws s3 ls
aws ec2 terminate-instances
aws iam create-user
```

until you manually rotate it.

Could be:

```text
days
weeks
months
years
```

before anyone notices.

---

# Modern Solution: OIDC

OIDC = OpenID Connect

Instead of storing AWS credentials in GitHub:

```text
GitHub asks AWS for temporary credentials
```

every workflow run.

---

# Real-life analogy

Static secrets:

```text
Permanent office key
```

If stolen:

```text
Anyone can enter forever
```

---

OIDC:

```text
Temporary visitor badge
```

Valid for:

```text
15 minutes
```

Then expires automatically.

---

# OIDC flow

Imagine this workflow starts:

```yaml
jobs:
  deploy:
```

GitHub says:

```text
I need AWS access
```

GitHub creates a signed JWT token.

JWT = JSON Web Token.

Think:

```text
Digital ID card
```

containing:

```json
{
  "repository": "Abdulrahman-122/project",
  "branch": "main",
  "actor": "Abdulrahman-122"
}
```

---

GitHub sends:

```text
JWT
```

to AWS.

AWS verifies:

```text
Did GitHub sign this?
```

If yes:

```text
✓ trusted
```

AWS creates temporary credentials:

```text
AccessKey
SecretKey
SessionToken
```

valid for a short time.

---

# Why do we need this?

Because now:

```text
No AWS secret stored in GitHub
```

Nothing to leak.

Nothing to rotate.

Nothing to maintain.

---

# Why do we need this permission?

```yaml
permissions:
  id-token: write
```

Many beginners think:

```text
write = modify token
```

Not here.

It means:

```text
Allow GitHub to generate an OIDC token
```

without it:

```yaml
permissions:
  id-token: write
```

OIDC fails.

---

# Lab 1: Observe the Permission

Create:

```yaml
name: OIDC_Permission_Demo

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    permissions:
      id-token: write

    steps:
      - run: |
          echo "OIDC enabled"
```

Run it.

Nothing exciting happens yet.

But GitHub is now allowed to issue JWT tokens.

---

# Understanding the Role

This line:

```yaml
role-to-assume:
  arn:aws:iam::123456789012:role/github-actions-role
```

means:

```text
AWS Role = identity
```

Think:

```text
Linux user
```

Example:

```text
root
nginx
mysql
ubuntu
```

AWS equivalent:

```text
github-actions-role
```

---

# What permissions does the role have?

Maybe:

```json
{
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ]
}
```

Meaning:

```text
Can upload files to S3
```

but not:

```text
Delete EC2 instances
```

---

# The complete flow

Workflow starts:

```text
GitHub Runner
```

↓

Requests JWT

↓

GitHub signs JWT

↓

AWS validates JWT

↓

AWS grants temporary credentials

↓

Workflow deploys

↓

Credentials expire automatically

---

# Practical DevOps Example

Imagine your Flask app.

Today:

```text
GitHub
    |
    | static secret
    |
AWS
```

You store:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside GitHub.

---

Modern approach:

```text
GitHub
   |
   | JWT
   |
AWS IAM Role
   |
Temporary Credentials
```

No secret exists in GitHub.

---

# Lab 2 (No AWS Account Required)

Create:

```yaml
name: Compare_Authentication

on:
  workflow_dispatch:

jobs:

  static-secret:
    runs-on: ubuntu-latest

    env:
      API_KEY: ${{ secrets.MY_SECRET }}

    steps:
      - run: echo "Using static secret"

  oidc:
    runs-on: ubuntu-latest

    permissions:
      id-token: write

    steps:
      - run: echo "Using OIDC authentication"
```

You won't actually connect to AWS.

The goal is to visualize:

```text
Static Secret:
GitHub -> Secret -> Cloud

OIDC:
GitHub -> JWT -> Cloud -> Temporary Credentials
```

---

# What you should remember for interviews

If someone asks:

> Why is OIDC preferred over AWS access keys?

A strong answer is:

1. No long-lived credentials stored in GitHub.
2. Credentials are generated only when needed.
3. Credentials expire automatically.
4. Smaller blast radius if compromised.
5. No manual key rotation.
6. Permissions can be restricted to specific repositories and branches.

That's the main lesson of this chapter. In real DevOps jobs, OIDC with cloud providers is now considered the standard approach, while static access keys are generally used only when OIDC isn't available.
