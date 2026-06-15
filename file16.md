This lesson is teaching something very different from composite actions.

Up until now you've built:

```text
Workflow
 └── Composite Action
       └── Bash commands
```

Now you're building:

```text
Workflow
 └── JavaScript/TypeScript Action
       └── Node.js code
```

The action itself is a Node application.

---

# Why JavaScript Actions exist

Composite actions are great when you only need:

```yaml
- run: npm install
- run: npm test
```

But what if you need logic?

For example:

```text
Read input
Validate it
Call GitHub API
Create issue
Return output
```

Bash becomes messy.

That's where JavaScript actions shine.

---

# Let's rebuild the lab from scratch

## Goal

Create an action that:

```text
Input:
  milliseconds

Behavior:
  Waits N milliseconds

Output:
  Current time
```

---

# Project Structure

```text
.github/
actions/
  wait-action/
    action.yml
    package.json
    src/
      index.ts
      wait.ts
    dist/
      index.js
```

---

# Step 1: action.yml

This describes the action to GitHub.

```yaml
name: Wait Timer

inputs:
  milliseconds:
    description: Duration to wait
    required: true

outputs:
  time:
    description: Current time

runs:
  using: node20
  main: dist/index.js
```

---

## What does this mean?

```yaml
runs:
  using: node20
```

means:

> Run my action using Node.js version 20.

Unlike:

```yaml
runs:
  using: composite
```

which executes bash steps.

---

And:

```yaml
main: dist/index.js
```

means:

> Start execution from dist/index.js.

Like Python:

```python
python main.py
```

---

# Step 2: wait.ts

```ts
export function wait(ms: number): Promise<void> {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

---

What does it do?

If:

```ts
await wait(3000)
```

the program pauses for 3 seconds.

---

# Step 3: index.ts

```ts
import * as core from "@actions/core";
import { wait } from "./wait.js";

export async function run(): Promise<void> {
  try {
    const ms = core.getInput("milliseconds");

    core.info(`Waiting ${ms} milliseconds`);

    await wait(parseInt(ms, 10));

    core.setOutput("time", new Date().toTimeString());
  } catch (error) {
    if (error instanceof Error) {
      core.setFailed(error.message);
    }
  }
}

run();
```

---

# Understanding the GitHub API

This line:

```ts
const ms = core.getInput("milliseconds");
```

reads:

```yaml
with:
  milliseconds: 3000
```

from the workflow.

---

This line:

```ts
core.info("hello");
```

prints:

```text
hello
```

inside GitHub Actions logs.

---

This line:

```ts
core.setOutput("time", ...)
```

creates an output.

Equivalent concept:

```python
return value
```

---

This line:

```ts
core.setFailed(...)
```

marks the action as failed.

Equivalent:

```python
raise Exception()
```

---

# Step 4: package.json

```json
{
  "name": "wait-action",
  "version": "1.0.0",
  "type": "module",
  "dependencies": {
    "@actions/core": "^1.11.0"
  }
}
```

Install:

```bash
npm install
```

---

# Step 5: Build

TypeScript cannot run directly.

GitHub executes:

```text
dist/index.js
```

not:

```text
src/index.ts
```

So you compile:

```bash
npm run package
```

or:

```bash
npx tsc
```

depending on project setup.

This creates:

```text
dist/index.js
```

---

# Step 6: Workflow

```yaml
name: Test JS Action

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - id: wait
        uses: ./actions/wait-action
        with:
          milliseconds: 3000

      - run: echo "${{ steps.wait.outputs.time }}"
```

---

# Execution Flow

```text
Workflow starts
       │
       ▼
Checkout repo
       │
       ▼
Load action.yml
       │
       ▼
Node 20 starts
       │
       ▼
Execute dist/index.js
       │
       ▼
Read milliseconds input
       │
       ▼
Wait 3 seconds
       │
       ▼
Set output "time"
       │
       ▼
Workflow prints output
```

---
