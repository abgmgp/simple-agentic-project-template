---
name: passcheck
description: Audit the source for code that does not comply with the project's documented context or system architecture
---

# Instructions

Scan the project source for violations of the standards and architecture documented in `.agents/context/`. Do not assume specific filenames — discover them at runtime so the skill works across projects.

## 1. Discover the standards

- List every markdown file under `.agents/context/`:
  ```bash
  find .agents/context -maxdepth 2 -type f -name '*.md'
  ```
- Read each one. Treat their contents as the authoritative rule set for this run. Typical files cover coding standards, module/system architecture, and tech stack — but adapt to whatever is present.
- If `.agents/context/` is missing or empty, stop and tell the user there is nothing to check against.

## 2. Extract checkable rules

From the files you read, derive a working list of rules grouped as:

- **Coding standards** — naming, formatting, allowed/disallowed APIs, comment policy, file structure within a module.
- **Architecture / module standards** — where files belong, allowed import directions, routing/state conventions, shared vs. per-module ownership.
- **Tech stack constraints** — only sanctioned packages, version expectations, framework idioms (e.g. routing library, state library).

For each rule, note how you will detect a violation (grep pattern, AST shape, folder lookup, dependency check). If a rule is too vague to check mechanically, flag it and skip rather than guess.

## 3. Scan the source

- Default scan root is `src/` if it exists; otherwise the repo root excluding `node_modules`, `dist`, `build`, `.git`, `.agents`, `.claude`, `.github`.
- Apply each derived rule. Prefer cheap checks first (grep, file existence, `package.json` inspection) before deeper reads.
- Cross-check imports against the documented architecture: flag wrong-direction imports, files placed outside their documented home, components living in the wrong layer.

## 4. Report

Group findings by severity:

- **Violation** — direct contradiction of a documented rule. Include rule source (file + section), offending path with line number, and a one-line explanation.
- **Drift** — likely violation but the rule is ambiguous; ask the user to confirm before treating it as a failure.
- **Skipped rule** — rule was too vague to mechanize; list it so the user knows it was not enforced.

Provide a count summary at the top. For large result sets, group by rule and show the top offenders, then offer to dump the full list.

## 5. Offer fixes (optional)

- Ask whether the user wants you to fix violations.
- If yes, fix in small batches grouped by rule, re-running the relevant check after each batch.
- Do not auto-fix Drift findings — confirm each one first.
- After fixes, run lint/build (via the `ci` skill or `npm run lint && npm run build:dev`) and report results.

## Guardrails

- Never invent rules that are not in `.agents/context/`. If the user expects a check that no context file documents, point that out and ask whether to add it to context.
- Do not modify files in `.agents/context/` as part of this skill — it reads them only.
- Treat the context files as the spec; treat the source as what is being audited. When they conflict, the context wins (unless the user says otherwise).
