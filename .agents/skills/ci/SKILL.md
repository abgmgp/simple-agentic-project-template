---
name: ci
description: Create a GitHub Actions CI workflow that runs on push or pull request activity targeting the main or master branch
---

# Instructions

Generate a GitHub Actions workflow at `.github/workflows/ci.yml` that triggers on push and pull request activity against the `main` or `master` branch. The workflow should run the project's standard validation steps (install/restore, lint/format, build, test) so that branch activity is verified automatically. The exact steps are derived from the detected stack — not from a fixed template.

## 1. Check for an existing workflow

- If `.github/workflows/ci.yml` already exists, read it and stop. Tell the user it already exists and ask whether to overwrite before proceeding.
- If `.github/workflows/` does not exist, create it.

## 2. Detect the stack

Do not assume Node. Inspect the repo to figure out what kind of project this is. Consult `.agents/context/tech-stack.md` first if it exists. Then check manifests at the repo root and one level down:

| Ecosystem | Manifest signal | Toolchain action | Install / restore | Typical validation commands |
|-----------|-----------------|------------------|-------------------|-----------------------------|
| Node.js | `package.json` (+ lockfile) | `actions/setup-node@v4` with lockfile-aware `cache` | `npm ci` / `pnpm install --frozen-lockfile` / `yarn install --frozen-lockfile` (per lockfile) | scripts that actually exist: `lint`, `build`, `test`, `typecheck`, etc. |
| .NET | `*.sln` / `*.csproj` / `Directory.Packages.props` (+ optional `global.json`) | `actions/setup-dotnet@v4` with version from `global.json` or the highest TFM found | `dotnet restore` | `dotnet build --no-restore -c Release`, `dotnet test --no-build -c Release` (only if any test project exists), `dotnet format --verify-no-changes` (only if `.editorconfig` exists or the project already uses it in another workflow) |
| Python | `pyproject.toml` / `setup.py` / `requirements*.txt` | `actions/setup-python@v5` with version from `pyproject.toml` / `.python-version` | `pip install -e .` or `pip install -r requirements.txt`, prefer `uv sync` / `poetry install` when the corresponding lockfile is present | `ruff check` / `flake8`, `mypy`, `pytest` — only if configured |
| Go | `go.mod` | `actions/setup-go@v5` with `go-version-file: go.mod` | `go mod download` | `go vet ./...`, `go build ./...`, `go test ./...` |
| Rust | `Cargo.toml` (+ `Cargo.lock`) | `dtolnay/rust-toolchain@stable` (or from `rust-toolchain.toml`) | `cargo fetch` | `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo build --release`, `cargo test` |
| Java/Kotlin (Maven) | `pom.xml` | `actions/setup-java@v4` with `cache: maven` | `mvn -B -ntp dependency:go-offline` | `mvn -B -ntp verify` |
| Java/Kotlin (Gradle) | `build.gradle*` (+ `gradlew`) | `actions/setup-java@v4` with `cache: gradle` | (none — gradle resolves on demand) | `./gradlew build` |
| PHP | `composer.json` | `shivammathur/setup-php@v2` | `composer install --prefer-dist --no-progress` | scripts that exist in `composer.json` (`lint`, `test`, etc.) |
| Ruby | `Gemfile` | `ruby/setup-ruby@v1` with `bundler-cache: true` | (handled by action) | `bundle exec rake` if defined, else `bundle exec rspec` / `bundle exec rubocop` if present |

When the repo has multiple ecosystems (e.g. a .NET backend with a Node frontend), generate one job per ecosystem in the same workflow, named accordingly (`build-dotnet`, `build-web`). Each job restricts itself to its own paths.

For each detected step, verify the command will actually do something before including it:

- Node: include `lint` / `build` / `test` only if the matching `scripts` entry exists in `package.json`.
- .NET: include `dotnet test` only if at least one test project is discoverable (`*.Test*.csproj`, `*.Tests.csproj`, or referencing `Microsoft.NET.Test.Sdk`).
- Python/Go/Rust: include lint/test steps only if the relevant config (`ruff.toml`, `mypy.ini`, `pytest.ini`, test files, `clippy.toml`, etc.) is present.

Do not invent scripts or test discovery. If a step has no backing configuration, omit it rather than emitting a placeholder.

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
- One or more jobs on `ubuntu-latest`. Each job:
  1. `actions/checkout@v4`
  2. The ecosystem's toolchain-setup action (with caching where supported)
  3. The install / restore step
  4. One `run:` step per validation command that the detection in §2 confirmed is real

Pin actions to a major version (`@v4`, `@v5`) — do not float to `@main`.

## 4. Report

- List the file written, the detected ecosystem(s), and each step included (and each candidate step that was skipped, with the reason).
- Remind the user to commit and push so the workflow is registered on GitHub.

## Guardrails

- Only target `main` and `master`. Do not add other branches unless the user asks.
- Do not include deploy, release, publish, or container-push steps — this skill is scoped to CI validation.
- Do not assume a stack. Drive every step from detected manifests and verified configuration.
- Prefer lockfile-aware / offline-restore installs (`npm ci`, `dotnet restore`, `go mod download`, `composer install --prefer-dist`) over unpinned installs.
- If the repo has no recognizable stack, stop and ask the user what to validate instead of guessing.
