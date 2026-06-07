---
name: code-standard
description: Review and revise the project's coding standard documents in .agents/context based on the actual source, or bootstrap one if none exist
---

# Instructions

Keep the project's coding standard in sync with the source. There are two modes — pick automatically based on what's already in `.agents/context/`.

## 1. Detect the current state

List markdown files under `.agents/context/`:

```bash
find .agents/context -maxdepth 2 -type f -name '*.md'
```

- **Bootstrap mode** — no standardization documents present (directory missing, empty, or contains only unrelated files like `tech-stack.md` and nothing covering coding rules). Go to §4.
- **Review mode** — at least one coding-standard-style document exists. Go to §2.

Do not hardcode `coding-standard.md` as the only valid name. Treat any file whose content lists project-wide coding/architecture rules as a standardization document.

## 2. Review mode — gather signal from the source

Default scan root is `src/` (fallback to repo root excluding `node_modules`, `dist`, `build`, `.git`, `.agents`, `.claude`, `.github`).

Build a picture of what the codebase actually does:

- Folder layout under `src/` and how modules are split (routes, layout components, module components, shared, ui, schema, types, hooks, store, config, utils).
- Routing library and registration pattern.
- State management split (global store vs local state).
- Data-fetching wrappers (which hooks wrap which HTTP verbs).
- Form library + validation library pairing.
- Animation library, UI kit, icon library.
- How API endpoints, query keys, enums, and field accessors are referenced.
- Notification / confirmation patterns.
- Anything that appears consistently across ≥2 modules.

## 3. Review mode — propose a delta, do not edit yet

Produce three lists and show them to the user before changing anything:

- **Existing rules that may not apply anymore** — broken pointers (referenced files don't exist), library names that don't match `package.json`, terminology that contradicts the source. Flag with reason. Do **not** remove yet.
- **Existing rules that need clarification** — vague rules where the source shows a more specific pattern.
- **Proposed additions** — conventions clearly followed in the source (≥2 examples) but missing from the document.

Ask the user which to apply. Accept answers like "all additions", "all + fixes", or specific item numbers. Then:

- Apply in-place edits for fixes/clarifications.
- Append new rules at the end of the relevant section (e.g. `# General Rules`) preserving the file's existing structure.
- Never silently delete a rule. If the user wants something removed, do it as an explicit separate step.

## 4. Bootstrap mode — create `coding-standard.md` from the scan

When no standardization document exists, run the §2 scan, then write `.agents/context/coding-standard.md` with a `# General Rules` section populated from observed patterns. Include only rules supported by ≥2 examples in the source. For each rule, prefer concrete references (folder paths, file names, library names from `package.json`) over abstract phrasing.

Template:

```markdown
# General Rules

- <observed rule 1>
- <observed rule 2>
- ...

# Skill Preference

- (leave empty or seed with obvious ones, e.g. scaffold-module for new modules)
```

After writing, show the user the file and ask whether to adjust any item before treating it as the new baseline.

## Guardrails

- Do not invent rules. Every rule must be traceable to either an existing document line or an observed source pattern.
- Do not edit files outside `.agents/context/` as part of this skill.
- Do not remove existing rules without explicit user approval.
- When in doubt about whether something is a real convention or a one-off, ask before adding.
