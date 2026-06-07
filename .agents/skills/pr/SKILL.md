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

Use the template below. Fill it from the diff against the target branch (`git log <target>..HEAD` and `git diff <target>...HEAD`).

```markdown
## Summary
<1–3 bullets describing the change at a high level.>

## Implemented Fix
<What was changed and why. Reference key files and the user-visible effect.>

## Test Checklist
- [ ] Lint passes (`npm run lint`)
- [ ] Build passes (`npm run build:dev`)
- [ ] Manual smoke test of the affected module
- [ ] <add scenario-specific checks here>
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
