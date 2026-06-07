---
name: pr
description: Create a pull request to main/master on GitHub or Azure DevOps with summary, implemented fix, and test checklist
---

# Instructions

Open a pull request from the current branch to the repository's default integration branch (`main` or `master`). Works for both GitHub and Azure DevOps remotes.

## 1. Detect the remote provider

Inspect the `origin` URL:

```bash
git remote get-url origin
```

- Contains `github.com` → use **GitHub CLI** (`gh`).
- Contains `dev.azure.com` or `visualstudio.com` → use **Azure CLI** (`az repos pr`).
- If neither matches, ask the user which provider to target.

## 2. Determine the target branch

Check which exists on the remote and prefer in this order: `main`, then `master`.

```bash
git ls-remote --heads origin main master
```

Use the first one that resolves. If neither exists, ask the user.

## 3. Sync the branch

- Ensure the working tree is clean. If there are uncommitted changes, list them and ask the user before continuing.
- Push the current branch with upstream tracking if not already pushed:

```bash
git push -u origin HEAD
```

## 4. Draft the PR body

Use the template below. Fill it from the diff against the target branch (`git log <target>..HEAD` and `git diff <target>...HEAD`). Derive the Test Checklist items from the **detected stack and the changed files** — do not hardcode `npm` commands.

Stack-detection cues for the checklist:

- Consult `.agents/context/tech-stack.md` first if present.
- Otherwise inspect manifests at the repo root and one level down (`package.json`, `*.csproj`/`*.sln`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`/`build.gradle*`, `composer.json`, `Gemfile`, etc.).
- Prefer commands that already exist: `package.json` scripts, `.github/workflows/ci.yml` steps, `Makefile` targets, `justfile` recipes. If the repo has a CI workflow, mirror its validation commands in the checklist rather than guessing.

Map detected stack → default checklist entries (only include what's real):

| Stack | Typical entries |
|---|---|
| Node | `npm run lint` / `npm test` / `npm run build` (only if the matching script exists; use `pnpm` / `yarn` if that's the lockfile) |
| .NET | `dotnet build <sln> -c Release`, `dotnet test <sln> -c Release` (only if a test project exists) |
| Python | `ruff check` / `mypy` / `pytest` (only if configured) |
| Go | `go vet ./...`, `go test ./...` |
| Rust | `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo test` |
| Java/Kotlin | `./gradlew build` or `mvn -B -ntp verify` |

Always add a "Manual smoke test of the affected module/endpoint/CLI surface" line plus scenario-specific checks pulled from the diff (new endpoints, new flags, changed migrations, UI screens touched, etc.).

```markdown
## Summary
<1–3 bullets describing the change at a high level.>

## Implemented Fix
<What was changed and why. Reference key files and the user-visible effect.>

## Test Checklist
- [ ] <validation command 1 from detected stack>
- [ ] <validation command 2 from detected stack>
- [ ] Manual smoke test of the affected surface
- [ ] <scenario-specific check derived from the diff>
```

Title: short, imperative, under 70 characters. Pull from the most descriptive commit subject if it fits; otherwise summarize.

## 5. Create the PR

### GitHub
```bash
gh pr create \
  --base <target-branch> \
  --head "$(git branch --show-current)" \
  --title "<title>" \
  --body "$(cat <<'EOF'
<body from step 4>
EOF
)"
```

### Azure DevOps
```bash
az repos pr create \
  --target-branch <target-branch> \
  --source-branch "$(git branch --show-current)" \
  --title "<title>" \
  --description "<body from step 4>" \
  --output table
```

If `az` is not authenticated, prompt the user to run `az login` and `az devops configure --defaults organization=<url> project=<name>` before retrying.

## 6. Report

Return the PR URL (both CLIs print it on success).

## Guardrails

- Never force-push.
- Do not push directly to `main`/`master`.
- Do not create the PR if `git status` shows uncommitted changes — ask first.
