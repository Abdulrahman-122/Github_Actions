Interactive Decision Tree

Imagine you want GitHub Actions to do something.

For example:

Build Flask app
Run tests
Create Docker image
Deploy to Kubernetes

Ask yourself the questions in order.

Step 1: Does a Marketplace Action Already Exist?

Question:

Can someone else's action already do this?

Examples:

- uses: actions/checkout@v5
- uses: actions/setup-node@v4
- uses: docker/build-push-action@v6
- uses: aws-actions/configure-aws-credentials@v4

If YES:

Use Marketplace Action

No need to reinvent the wheel.

Real Example

Instead of:

git clone ...

Use:

- uses: actions/checkout@v5

GitHub already solved it.

Step 2: Can a Simple Shell Command Solve It?

Suppose you want:

Print Python version

You don't need an action.

Just:

- run: python --version

or

- run: |
    pip install -r requirements.txt
    pytest

That's enough.

Rule

If the task is:

1-5 shell commands

Keep it inline.

Step 3: Is the Shell Script Getting Large?

Example:

- run: |
    docker build .
    docker tag ...
    docker push ...
    kubectl apply ...
    kubectl rollout status ...

Now your workflow becomes ugly.

Move it to:

scripts/deploy.sh

Workflow:

- run: ./scripts/deploy.sh

Cleaner.

Step 4: Need Reuse Across Workflows?

Suppose you have:

test.yml
deploy.yml
release.yml

and all contain:

Install dependencies
Run tests
Generate reports

Don't copy-paste.

Use:

Composite Action

Example:

.github/actions/run-tests/
├── action.yml

Then:

- uses: ./.github/actions/run-tests

Now every workflow can reuse it.

Step 5: Need Entire Pipelines Reused?

Suppose every repository must run:

Build
Test
Security Scan
Deploy

exactly the same way.

Use:

Reusable Workflow

instead of Composite Action.

Because now you're sharing:

Jobs
Permissions
Secrets
Environments
Conditions

not just steps.

Composite vs Reusable Workflow
Composite Action

Shares:

Step 1
Step 2
Step 3
Reusable Workflow

Shares:

Job A
Job B
Job C
Permissions
Secrets

Entire CI/CD pipeline.

Step 6: Need Real Programming?

Suppose you need:

Read API
Parse JSON
Generate report
Complex logic

Shell becomes painful.

Question:

Do you need external dependencies?
Option A: No Dependencies

Use a normal script.

Example:

scripts/report.py

Workflow:

- run: python scripts/report.py

No custom action needed.

This is what the image means by:

Use an in-repo Python (or other) script
Example

Repository:

project/
├── scripts/
│   └── deploy.py
└── .github/

Workflow:

- run: python scripts/deploy.py

Simple.

Step 7: Need Dependencies?

Suppose your script requires:

Flask
Boto3
Terraform
Custom tools

Now packaging becomes important.

Question:

Do you like JS/TS?
Option A: JavaScript/TypeScript Action

Best GitHub integration.

Example:

runs:
  using: node20

Advantages:

Fast
Native GitHub SDK
Access outputs easily
Marketplace friendly

Good for:

GitHub API tools
PR automation
Issue automation
Release automation
Option B: Container Action

If you prefer:

Python
Go
Rust
Java
Anything

Use:

runs:
  using: docker

Advantages:

Any language
Any dependencies
Any OS packages

Example:

FROM python:3.12

RUN pip install flask boto3

COPY entrypoint.py .

ENTRYPOINT ["python","entrypoint.py"]

Perfect for Python developers.

If You Are Abdulrahman

Since you're learning:

Linux
Docker
Kubernetes
Flask
DevOps

your decision tree usually becomes:

Most Common
- run: |
    pytest
    docker build .
Sometimes
- run: python scripts/deploy.py
Reuse Needed
Composite Action
Organization-wide Standards
Reusable Workflow
Heavy Python Tool
Container Action
Rarely
JavaScript Action

unless you're building GitHub-specific integrations.

Publishing Actions

Suppose you create:

my-awesome-action/

and want others to use it.

Repository:

my-awesome-action/
├── action.yml
├── README.md

Users can call:

- uses: your-name/my-awesome-action@v1
Security Best Practice

Avoid:

- uses: actions/checkout@v5

because tags can move.

Prefer:

- uses: actions/checkout@ff7abcd0c3c05ccf6adc123a8cd1fd4fb30fb493

which pins an exact commit.

The Practical DevOps Order

For your projects, follow this order:

1. Marketplace Action
          ↓
2. Inline Bash
          ↓
3. Python Script in Repo
          ↓
4. Composite Action
          ↓
5. Reusable Workflow
          ↓
6. Container Action
          ↓
7. JavaScript/TypeScript Action
