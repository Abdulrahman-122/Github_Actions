You're at the point where you're learning **how professional teams develop GitHub Actions**, not just how to write YAML.

The document is really teaching **Developer Experience (DX)**:

> "How do I test, debug, and iterate on Actions and Workflows quickly without pushing 20 commits to GitHub?"

---

# Part 1 — Commands & Tools You Need To Learn

## 1. `@github/local-action`

### Problem

Suppose you're building a JavaScript/TypeScript Action.

Normally:

```text
Edit code
↓
git push
↓
Wait for workflow
↓
See error
↓
Repeat
```

Very slow.

---

### Solution

Run the Action locally:

```bash
npx @github/local-action . src/main.ts .env
```

Meaning:

```text
.
│
├── action.yml
├── src/main.ts
└── .env
```

Run the action exactly as GitHub would.

---

### Example `.env`

```env
INPUT_NAME=Abdulrahman
```

GitHub input:

```yaml
inputs:
  name:
```

becomes:

```bash
INPUT_NAME
```

locally.

---

## 2. Unit Tests (Jest)

### Problem

You don't want GitHub to tell you your code is broken.

You want your laptop to tell you first.

---

Example:

```ts
export function greet(name:string){
    return `Hello ${name}`
}
```

Test:

```ts
test("greeting",()=>{
   expect(greet("Abdulrahman"))
      .toBe("Hello Abdulrahman")
})
```

Run:

```bash
npm test
```

Result:

```text
✓ greeting
```

No GitHub needed.

---

## 3. `act`

This is probably the most important tool for workflow development.

### Problem

You want to test:

```yaml
jobs:
  build:
  deploy:
```

locally.

Not an Action.

A full Workflow.

---

Install:

```bash
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash
```

Run:

```bash
act
```

or

```bash
act workflow_dispatch
```

It launches Docker containers that behave like GitHub runners.

---

Mental model:

```text
GitHub Runner
       ↓
Docker Container
       ↓
act creates it locally
```

---

# 4. Debug Logging

Normal:

```bash
echo "hello"
```

Always shown.

---

Debug:

```bash
echo "::debug::Current directory $(pwd)"
```

Hidden.

Enable:

```text
ACTIONS_STEP_DEBUG=true
```

Now visible.

---

Useful for:

```bash
pwd
ls
env
whoami
```

debugging.

---

# 5. Log Levels

### Debug

```bash
echo "::debug::Debug message"
```

Hidden normally.

---

### Info

```bash
echo "::notice::Build completed"
```

Shows as blue info.

---

### Warning

```bash
echo "::warning::Missing file"
```

Shows warning.

---

### Error

```bash
echo "::error::Build failed"
```

Shows red error.

---

# 6. Breakpoints

Normally:

```text
Workflow failed
Runner destroyed
```

You can't inspect it.

---

Breakpoint Action:

```yaml
- uses: namespacelabs/breakpoint-action
```

When failure happens:

```text
SSH into runner
```

and investigate manually.

This is advanced debugging.

---

# Part 2 — What is a Taskfile?

This is probably the part confusing you.

---

## Without Taskfile

Workflow:

```yaml
run: |
  npm install
  npm run build
  npm test
  npm run lint
```

Huge YAML.

---

## With Taskfile

Move logic:

### Taskfile.yml

```yaml
version: "3"

tasks:
  build:
    cmds:
      - npm install
      - npm run build

  test:
    cmds:
      - npm test
```

---

Run locally:

```bash
task build
```

Run in GitHub:

```yaml
run: task build
```

Same command.

---

Benefit:

```text
Local machine
     and
GitHub Actions

use identical commands
```

No duplication.

---

# Labs To Build (In Order)

## Lab 1 — Debug Logs

Create workflow:

```yaml
echo "::debug::pwd=$(pwd)"
echo "::warning::warning"
echo "::error::error"
```

Enable:

```text
ACTIONS_STEP_DEBUG=true
```

Observe difference.

---

## Lab 2 — Build Your First Taskfile

Create:

### Taskfile.yml

```yaml
version: "3"

tasks:
  hello:
    cmds:
      - echo "Hello Abdulrahman"
```

Install Task:

```bash
sudo snap install task --classic
```

Run:

```bash
task hello
```

Expected:

```text
Hello Abdulrahman
```

---

## Lab 3 — Workflow Uses Taskfile

Workflow:

```yaml
run: task hello
```

Instead of:

```yaml
run: echo "Hello Abdulrahman"
```

Learn separation of concerns.

---

## Lab 4 — Local Workflow Testing With `act`

Workflow:

```yaml
name: Test

on:
  workflow_dispatch

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Hello"
```

Run:

```bash
act workflow_dispatch
```

No GitHub push needed.

---

## Lab 5 — Custom Action + Local Action

Create:

```text
action.yml
src/main.ts
```

Run:

```bash
npx @github/local-action . src/main.ts .env
```

Learn Action testing.

---

## Lab 6 — Full Developer Workflow

Build:

```text
Taskfile.yml
↓
Workflow
↓
act
↓
GitHub Actions
```

Flow:

```text
Write code
↓
task test
↓
act
↓
git push
↓
GitHub Actions
```

This is very close to how many professional teams work.

---

If your goal is becoming strong at GitHub Actions, I'd learn them in this order:

1. Workflows YAML
2. Composite Actions
3. JavaScript Actions
4. Docker/Container Actions
5. Taskfile
6. act
7. Debug logs
8. Breakpoints
9. Reusable Workflows

That's roughly the progression from beginner to someone who can maintain CI/CD pipelines professionally.
