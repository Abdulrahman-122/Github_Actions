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
```yml
my-repo/
├── devbox.json       <-- here
├── .github/
│   └── workflows/
│       └── test.yml
└── actions/
    └── wait_action/
        ├── action.yml
        ├── package.json
        ├── rollup.config.js
        ├── tsconfig.json
        ├── src/
        │   ├── index.ts
        │   └── wait.ts
        └── dist/
```
---
make the environment ;
1.Devbox environment -> to isolate the project from the environment around it
Create a devbox.json
```
Inside your project root:

{
  "packages": [
    "nodejs@20"
  ],

  "shell": {
    "init_hook": [
      "echo 'Node development environment loaded'"
    ]
  }
}

Initialize:

devbox shell

Check:

node --version
npm --version

```
2.make package.json file;
4. Recommended package.json
```
For your TypeScript GitHub Action:

{
  "name": "wait-action",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build": "rollup -c"
  },
  "dependencies": {
    "@actions/core": "^1.11.0"
  },
  "devDependencies": {
    "@rollup/plugin-commonjs": "^28.0.0",
    "@rollup/plugin-node-resolve": "^16.0.0",
    "@rollup/plugin-typescript": "^12.0.0",
    "rollup": "^4.0.0",
    "typescript": "^5.0.0"
  }
}

Then inside Devbox:

npm install
npm run build

```
3. start compile ts files;
```
Install:

npm install -D typescript
Since the course uses Rollup

Install:

npm install -D rollup \
  @rollup/plugin-commonjs \
  @rollup/plugin-node-resolve \
  @rollup/plugin-typescript
Create tsconfig.json

Example:

{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "Node",
    "strict": true,
    "outDir": "dist"
  },
  "include": ["src/**/*"]
}
Update package.json

Add scripts:

{
  "name": "wait-action",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build": "rollup -c"
  },
  "dependencies": {
    "@actions/core": "^1.11.0"
  },
  "devDependencies": {
    "@rollup/plugin-commonjs": "...",
    "@rollup/plugin-node-resolve": "...",
    "@rollup/plugin-typescript": "...",
    "rollup": "...",
    "typescript": "..."
  }
}
Create rollup.config.js

The file you showed earlier:

import commonjs from '@rollup/plugin-commonjs'
import nodeResolve from '@rollup/plugin-node-resolve'
import typescript from '@rollup/plugin-typescript'

const config = {
  input: 'src/index.ts',
  output: {
    file: 'dist/index.js',
    format: 'es',
    sourcemap: true
  },
  plugins: [
    typescript(),
    nodeResolve({ preferBuiltins: true }),
    commonjs()
  ]
}

export default config
Build the action

Run:

npm run build

This should create:

dist/index.js
```
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
``` yaml
now after doing all of these setup
1.build action.yml file
2.build workflow file test
then understand how action connected with index.js(compiled file from index.ts) that we compiled it into js file
then run workflow
but note; before you run the workflow -> just add some lines to index.ts then compile then push to github then test
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
Good catch. I explained the lesson, but I didn't give you actual **hands-on labs** like the composite action exercises.

Here are labs in increasing difficulty.

---

# 🧪 Lab 1 — Greeting Action (Beginner)

## Goal

Build a JavaScript action that:

Input:

```yaml id="96txs5"
name: Abdulrahman
```

Output:

```text id="a06y9z"
Hello Abdulrahman!
```

---

## Requirements

### action.yml

Define:

```yaml id="l5ud74"
inputs:
  name:
```

---

### index.ts

Use:

```ts id="nsbv3q"
core.getInput(...)
```

to read the name.

Print:

```ts id="lk6vb9"
core.info(...)
```

---

## Workflow

```yaml id="tx7vhq"
with:
  name: Abdulrahman
```

---

## Expected logs

```text id="9ej03u"
Hello Abdulrahman!
```

---

# 🧪 Lab 2 — Greeting + Output

## Goal

Create an output.

Input:

```yaml id="w6w9ep"
name: Abdulrahman
```

Output:

```text id="3kccy7"
Hello Abdulrahman!
```

---

### New Requirement

Use:

```ts id="w5umq3"
core.setOutput(...)
```

---

Workflow:

```yaml id="yx8e7l"
- id: greet
  uses: ./actions/greeting
```

Then:

```yaml id="f2szh0"
- run: echo "${{ steps.greet.outputs.message }}"
```

---

Expected:

```text id="d4x5so"
Hello Abdulrahman!
```

---

# 🧪 Lab 3 — Wait Timer (Course Lab)

## Goal

Input:

```yaml id="jyt8dq"
milliseconds: 5000
```

Action:

```text id="0h24d3"
Wait 5 seconds
```

Output:

```text id="ykm4ez"
Current Time
```

---

Skills learned:

* async
* Promise
* await
* outputs

---

# 🧪 Lab 4 — Validate Input

## Goal

Accept:

```yaml id="sz8w2r"
age: 17
```

If age < 18:

Fail the action.

---

Use:

```ts id="aqz0s8"
core.setFailed(...)
```

---

Expected:

```text id="kdf4cl"
Age must be at least 18
```

Workflow becomes red.

---

# 🧪 Lab 5 — Linux Developer Utility

Since you're a Linux guy, this one is useful.

Input:

```yaml id="a5hmwm"
directory: .
```

Action should:

* count files
* count folders
* output totals

---

Example output:

```text id="1cbzj4"
Files: 42
Directories: 8
```

---

Hints:

```ts id="m3h4h9"
import fs from "fs";
```

and

```ts id="u22vpg"
fs.readdirSync(...)
```

---

# 🧪 Lab 6 — GitHub Username Action

Input:

```yaml id="qq77gd"
username: octocat
```

Use:

```ts id="bk4lyh"
fetch(...)
```

to call:

```text id="dtkhzr"
https://api.github.com/users/octocat
```

Return:

```text id="5yj7la"
Followers
Public Repos
Bio
```

---

Skills:

* APIs
* JSON
* async requests

This is where JavaScript actions become much more powerful than composite actions.

---

# 🧪 Lab 7 — Release Notes Generator

Input:

```yaml id="y4weow"
version: v1.0.0
```

Generate:

```text id="8m6mbh"
Release v1.0.0
Generated at:
2026-06-15 ...
```

Output the text as a workflow output.

---

# 🚀 Final Project

Build a real DevOps action:

### Docker Image Checker

Input:

```yaml id="ggrx1u"
image-name: nginx
tag: latest
```

Action:

* query Docker Hub API
* verify image exists
* output digest/tag information

This combines:

* TypeScript
* APIs
* Inputs
* Outputs
* Error handling
* GitHub Actions

and feels like something you'd actually use in CI/CD.

---
