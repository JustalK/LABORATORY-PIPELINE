# LABORATORY-PIPELINE

## GitHub

#### Experiences

I have created many actions on this repositories for testing how it exactly works. Those are the different things that has been tested:

- Running a bash script
- Testing the concurrency by grouping actions
- Testing the dispatch for manually running action with inputs
- Playing with the context
- Testing the output and sharing information between jobs
- Creating a scheduled actions
- Playing with the secrets by environment and global secrets
- Creating a pipeline for CI

Some information that might be useful if you get a permission error with the scripts. You can set the permission with the commands below:

```bash
$ git update-index --chmod=+x ./.github/scripts/hello.sh
```

For starting an action by CURL:

```bash
curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: Bearer <BEARER TOKEN>" "https://api.github.com/repos/justalk/LABORATORY-PIPELINE/actions/workflows/dispatch.yml/dispatches" -d '{"ref":"feature/check-github-pipeline","inputs":{"testVariable":"KEVIN", "environment":"stagging"}}'
```

#### Continuous Integration

I have created a **React** project using **NX**. I created an action testing the builds and tests on two different jobs using a matrix around the os. I have put a rule on concurrency for blocking multiple action to run if I keep pushing. I am using npx because you cannot access directly the command nx. I added the workflow_dispatch for manually running the pipeline when I want.

```json
name: CI

on:
  workflow_dispatch:
  push:
  pull_request:

permissions: read-all

concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-22.04, ubuntu-20.04]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "16.x"
      - name: Install CI
        working-directory: ./project
        run: npm ci
      - name: Build the apps
        working-directory: ./project
        run: npx nx build
      - name: Run the test
        working-directory: ./project
        run: npx nx test
```
