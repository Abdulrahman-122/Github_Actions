Now we're entering the **most powerful way to create GitHub Actions**.

Up until now:

* Composite Actions = shell scripts packaged together
* Reusable Workflows = entire pipelines

Now:

* JavaScript/TypeScript Actions = actual programs

---

# Why do JavaScript Actions exist?

Imagine you want an action that:

* Calls GitHub API
* Reads pull requests
* Creates issues
* Parses JSON
* Talks to AWS
* Makes HTTP requests
* Calculates stuff

Doing this in Bash becomes painful.

Instead:

```javascript
const response = await fetch(...)
```

Much easier.

---

# Think Like This

Composite Action:

```yaml
steps:
  - run: docker build .
  - run: docker push .
```

Mostly shell commands.

---

JavaScript Action:

```javascript
const prs = await github.rest.pulls.list(...)
```

Actual programming.

---

# Folder Structure

A typical JavaScript Action:

```text
my-action/
│
├── action.yml
├── package.json
├── src/
│   └── index.ts
│
└── dist/
    └── index.js
```

---

# action.yml

```yaml
name: Wait Timer

inputs:
  milliseconds:
    required: true

runs:
  using: node20
  main: dist/index.js
```

GitHub reads this.

---

## What does this mean?

```yaml
runs:
  using: node20
```

Means:

> "Run this action with Node.js."

Just like:

```bash
python app.py
```

or

```bash
node index.js
```

GitHub will run:

```bash
node dist/index.js
```

behind the scenes.

---

# main

```yaml
main: dist/index.js
```

Equivalent to:

```bash
node dist/index.js
```

GitHub starts execution from this file.

---

# Reading Inputs

Workflow:

```yaml
- uses: ./wait-action
  with:
    milliseconds: 5000
```

---

Inside action:

```typescript
const ms = core.getInput("milliseconds")
```

Gets:

```text
5000
```

---

Think:

Workflow:

```yaml
milliseconds: 5000
```

↓

Action:

```typescript
core.getInput("milliseconds")
```

↓

Returns:

```text
5000
```

---

# @actions/core

This package is GitHub's SDK.

Install:

```bash
npm install @actions/core
```

---

Without it:

```javascript
console.log("hello")
```

---

With it:

```javascript
core.info("hello")
```

GitHub understands it.

---

# Logging

Normal JS:

```javascript
console.log("hello")
```

---

GitHub way:

```javascript
core.info("hello")
```

Produces:

```text
hello
```

inside workflow logs.

---

# Debug Logs

```typescript
core.debug("Waiting...")
```

Only appears when debug mode enabled.

Useful when troubleshooting.

---

Example:

```typescript
core.debug(`Waiting ${ms}`)
```

Output:

```text
Waiting 5000
```

---

# Wait Function

```typescript
await wait(parseInt(ms,10))
```

Suppose:

```typescript
ms = 5000
```

Then:

```typescript
await wait(5000)
```

Action sleeps 5 seconds.

---

Equivalent Bash:

```bash
sleep 5
```

---

# Outputs

Very important.

Action:

```typescript
core.setOutput(
  "time",
  new Date().toTimeString()
)
```

Creates output:

```text
time=14:32:10
```

---

Workflow:

```yaml
- id: wait
  uses: ./wait-action
```

Later:

```yaml
- run: |
    echo "${{ steps.wait.outputs.time }}"
```

Output:

```text
14:32:10
```

---

# Error Handling

```typescript
try {
   ...
}
catch(error) {
   core.setFailed(error.message)
}
```

If action crashes:

```typescript
throw new Error("Database failed")
```

GitHub marks workflow:

```text
❌ FAILED
```

instead of silently continuing.

---

# Why TypeScript?

TypeScript:

```typescript
const ms: string =
    core.getInput("milliseconds")
```

---

JavaScript:

```javascript
const ms =
    core.getInput("milliseconds")
```

---

TypeScript catches mistakes.

Example:

```typescript
const x:number = "hello"
```

Compiler:

```text
Error!
```

before publishing.

---

# Why Build Dist Folder?

GitHub cannot run:

```typescript
src/index.ts
```

directly.

It only understands:

```javascript
dist/index.js
```

---

So:

```bash
npm run package
```

does:

```text
TypeScript
     ↓
Compile
     ↓
JavaScript
     ↓
dist/index.js
```

---

# Three Versions Mentioned

## Version 1

No Build

```text
action/
 ├─ index.js
 └─ node_modules/
```

Commit everything.

Works.

Bad practice.

Huge repository.

---

## Version 2

JavaScript + Build

```text
src/index.js
dist/index.js
```

Build before publishing.

Better.

---

## Version 3

TypeScript + Build

```text
src/index.ts
```

↓

```bash
npm run package
```

↓

```text
dist/index.js
```

Best practice.

Most professional actions use this.

---

# Real DevOps Example

Suppose you create an action:

```yaml
- uses: company/check-k8s
```

Inside TypeScript:

```typescript
const pods =
 await k8s.listPods()

if (pods.some(p => p.status !== "Running"))
{
   core.setFailed(
      "Cluster unhealthy"
   )
}
```

Now every repository can reuse it.

---

# When Should You Use This?

As a DevOps engineer:

### Use Composite Action

When:

```text
docker build
docker push
kubectl apply
```

Simple shell commands.

---

### Use JavaScript/TypeScript Action

When:

```text
Call GitHub API
Read PRs
Create Issues
Parse JSON
Interact with cloud APIs
Custom logic
```

---

A useful rule:

**80% of DevOps automation can be done with Composite Actions and Reusable Workflows.**

You usually reach for **JavaScript/TypeScript Actions** when you want to build something that feels like a real GitHub product or tool rather than just a sequence of shell commands.
