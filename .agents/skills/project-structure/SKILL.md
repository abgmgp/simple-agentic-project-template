---
name: project-structure
description: Review and revise the project's structure/architecture document in .agents/context based on the actual source layout, or bootstrap one if none exists
---

# Instructions

Keep the project's structure/architecture document in sync with what the source actually looks like. Pick mode automatically based on what's in `.agents/context/`.

## 1. Detect the current state

List markdown files under `.agents/context/`:

```bash
find .agents/context -maxdepth 2 -type f -name '*.md'
```

- **Bootstrap mode** — no structure document present (directory missing, empty, or contains only files unrelated to source layout). Go to §4.
- **Review mode** — at least one structure-style document exists. Go to §2.

Do not hardcode `project-structure.md` as the only valid name. Treat any file whose content describes the folder layout / architectural responsibilities as a structure document.

## 2. Review mode — observe the actual layout

First, determine the scan root(s). Do not assume `src/`. Derive them from what's actually in the repo:

- Read `.agents/context/tech-stack.md` (or whichever file documents the stack) if present — it tells you what ecosystem(s) you're in.
- Detect manifest files at the repo root and one level down to confirm: `package.json`, `pyproject.toml` / `setup.py`, `*.csproj` / `*.sln` / `Directory.Packages.props`, `go.mod`, `Cargo.toml`, `pom.xml` / `build.gradle*`, `composer.json`, `Gemfile`, `mix.exs`, etc.
- Pick scan roots from what exists, in this order: `src/` if it exists → otherwise the directories that contain manifest files → otherwise the repo root excluding `node_modules`, `dist`, `build`, `bin`, `obj`, `target`, `.git`, `.agents`, `.claude`, `.github`, `vendor`, `.venv`.
- Multi-root layouts are normal (e.g. a .NET solution with `Services/`, `Infrastructure/`, `Gateway/` siblings; a monorepo with `packages/*`). Treat each as a scan root.

Map the layout:

```bash
find <scan-root> -maxdepth 2 -type d | sort
find <scan-root> -maxdepth 2 -type f | sort
```

For each top-level folder under your scan root(s), note:

- What kinds of files live there (extensions, naming pattern).
- Sample contents (peek at 1–2 representative files).
- How it relates to other folders (imports, references, project references).

Also confirm assertions made in the existing document — e.g. if it says a folder contains "two files" or names a specific file, verify that's still true.

## 3. Review mode — propose a delta, do not edit yet

Produce three lists and show them to the user before changing anything:

- **Existing entries that may not apply anymore** — folders the doc names that don't exist in `src/`, files it names that aren't there, descriptions that contradict what's actually inside. Flag with reason. Do **not** remove yet.
- **Existing entries that need clarification** — descriptions that are vague or no longer match the breadth of what the folder holds.
- **Proposed additions** — folders or significant sub-folders that exist in `src/` but aren't documented.

Ask which to apply. Accept answers like "all additions", "all + fixes", or specific item numbers. Then:

- Apply in-place edits for fixes/clarifications.
- Append new folder entries in alphabetical order (or to match the document's existing ordering convention).
- Never silently delete an entry. If the user wants something removed, do it as an explicit separate step.

## 4. Bootstrap mode — create `project-structure.md` from the scan

When no structure document exists, perform the §2 scan-root detection and then the §2 mapping, and write `.agents/context/project-structure.md`. For each top-level folder under your detected scan root(s), include:

- Folder name as a heading.
- One-line role description grounded in observed contents.
- Notable sub-folders or naming conventions, only when supported by what's actually there.

Do not impose ecosystem-specific groupings (frontend/backend, app/components, services/infrastructure, etc.) unless the source layout actually exhibits them. The document mirrors what exists; it does not normalize the project toward a template.

Template:

```markdown
# Project structure:

Do not add, remove or modify unless with specific approval from the user.

### <folder>
<one-line role>
- <bullet of notable contents / sub-folders, if any>
```

Order folders alphabetically. After writing, show the user the file and ask whether to adjust any entry before treating it as the new baseline.

## Guardrails

- Do not invent folders or sub-folders. Every entry must be traceable to a real path in the detected scan root(s).
- Do not assume any particular stack or layout shape. No frontend defaults, no backend defaults, no monorepo defaults — derive everything from detected manifests and observed folders.
- Do not edit files outside `.agents/context/` as part of this skill.
- Do not remove existing entries without explicit user approval.
- If the source is genuinely sparse (e.g. only a few folders), keep the document short — do not pad with speculative structure.
