---
name: stack
description: Review and revise the project's tech stack document in .agents/context against package.json, flagging security risks and enforced deprecations
---

# Instructions

Keep the project's tech stack document in sync with what is actually installed, and surface any package risks. Pick mode automatically based on what's in `.agents/context/`.

## 1. Detect the current state

List markdown files under `.agents/context/`:

```bash
find .agents/context -maxdepth 2 -type f -name '*.md'
```

- **Bootstrap mode** — no tech-stack document present. Go to §5.
- **Review mode** — at least one tech-stack-style document exists. Go to §2.

Do not hardcode `tech-stack.md` as the only valid name. Treat any file whose content lists installed packages / framework versions as the tech-stack document.

## 2. Review mode — read the truth from package metadata

Read `package.json` (and `package-lock.json` if present). For each entry:

- Capture name and declared version range.
- Note `dependencies` vs `devDependencies` split.

Compare against the document:

- **Missing in doc** — packages listed in `package.json` not mentioned.
- **Stale in doc** — packages in the doc no longer in `package.json`, or with a version range that no longer matches.
- **Drifted classification** — doc lists a package under the wrong role (e.g. as dev tooling when it's a runtime dep).

## 3. Review mode — security & deprecation pass

In addition to the version reconciliation, run a risk check:

```bash
npm audit --json 2>/dev/null || true
npm outdated --json 2>/dev/null || true
```

Inspect each result and surface:

- **Security risks** — any vulnerability reported by `npm audit` at `moderate` or higher; include package name, severity, and the advisory title/url if available.
- **Enforced deprecations** — packages flagged as deprecated by the registry. If `npm audit` doesn't report deprecation, also check `npm ls --depth=0` output and `npm view <pkg> deprecated` for top-level deps the doc names.
- **Major behind** — packages where `npm outdated` shows the installed version is a full major version behind latest (informational; not auto-fail).

Report each finding with a one-line recommendation (upgrade target, replacement package, or "monitor"). Treat security and enforced-deprecation findings as **must-inform** even if the user only asked for a doc review.

If `npm` is unavailable in the environment, say so and skip §3 rather than guessing.

## 4. Review mode — propose a delta, do not edit yet

Show three lists plus the risk report before changing anything:

- **Existing entries that may not apply anymore** — stale packages, wrong versions, wrong classification.
- **Existing entries that need clarification** — vague groupings or outdated descriptions.
- **Proposed additions** — packages present in `package.json` but missing from the doc.
- **Risk report** — security risks, enforced deprecations, major-behind notes from §3.

Ask which doc changes to apply. Accept "all additions", "all + fixes", or specific item numbers. Then:

- Apply in-place edits for fixes/clarifications.
- Append new entries under the appropriate group (Runtime / Data & forms / UI / Utilities / Tooling, or whatever grouping the doc already uses).
- Never silently delete a stale entry. Confirm removal explicitly.

The risk report is informational — do not edit packages or run installs as part of this skill unless the user explicitly asks.

## 5. Bootstrap mode — create the tech stack doc from `package.json`

When no tech-stack document exists, generate `.agents/context/tech-stack.md` from `package.json`:

- Group packages by purpose (Runtime / framework, Data & forms, UI, Utilities, Tooling). Infer the group from the package name and what folders in `src/` consume it; when in doubt, put it under Utilities.
- One line per package: `- <name> <version-range>` (optionally a short parenthetical role, only when the role isn't obvious from the name).
- Keep it terse — this file is meant to be token-light context for agents.

After writing, run §3 once and show the user any risks before treating it as the new baseline.

## Guardrails

- Truth lives in `package.json`. The doc is a summary, not a source of versions.
- Do not run `npm install`, `npm update`, or modify `package.json` as part of this skill.
- Do not invent groupings the codebase doesn't actually justify.
- If a security or enforced-deprecation finding appears, surface it even if the user's request was only "review the doc".
