---
name: ci
description: Create a GitHub Actions CI workflow that runs on push or pull request activity targeting the main or master branch
---

# Instructions

Generate a GitHub Actions workflow at `.github/workflows/ci.yml` that triggers on push and pull request activity against the `main` or `master` branch. The workflow should run the project's standard validation steps (install, lint, build/test) so that branch activity is verified automatically.

## 1. Check for an existing workflow

- If `.github/workflows/ci.yml` already exists, read it and stop. Tell the user it already exists and ask whether to overwrite before proceeding.
- If `.github/workflows/` does not exist, create it.

## 2. Determine the validation steps

Inspect the repo before writing the workflow:

- Read `package.json` (or the language equivalent) to confirm which scripts exist (`lint`, `build`, `build:dev`, `test`, etc.). Only include steps that map to scripts actually defined in the project.
- Detect the package manager from the lockfile (`package-lock.json` → `npm ci`, `pnpm-lock.yaml` → `pnpm install --frozen-lockfile`, `yarn.lock` → `yarn install --frozen-lockfile`).
- Pick a Node.js version that matches `engines.node` in `package.json` if specified; otherwise default to the current active LTS.

## 3. Write the workflow

Create `.github/workflows/ci.yml` with:

- `name: CI`
- Triggers:
  ```yaml
  on:
    push:
      branches: [main, master]
    pull_request:
      branches: [main, master]
  ```
- A single `build` job on `ubuntu-latest` containing, in order:
  1. `actions/checkout@v4`
  2. `actions/setup-node@v4` with the chosen Node version and lockfile-aware `cache`
  3. Install step using the detected package manager
  4. One `run:` step per validation script that exists in the project (lint, build, test, etc.)

Do not invent scripts. If the project has no lint or build script, omit that step rather than adding a placeholder.

## 4. Report

- List the file written and each step included.
- Remind the user to commit and push so the workflow is registered on GitHub.

## Guardrails

- Only target `main` and `master`. Do not add other branches unless the user asks.
- Do not include deploy, release, or publish steps — this skill is scoped to CI validation.
- Do not skip lockfile-aware installs (`npm ci`, not `npm install`).
