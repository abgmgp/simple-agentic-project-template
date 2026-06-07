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

First, determine scan root(s). Do not assume `src/`. Derive them from what's actually present:

- Consult `.agents/context/tech-stack.md` and `.agents/context/project-structure.md` if they exist — they tell you the ecosystem and where code lives.
- Detect manifests at the repo root and one level down: `package.json`, `pyproject.toml` / `setup.py`, `*.csproj` / `*.sln` / `Directory.Packages.props`, `go.mod`, `Cargo.toml`, `pom.xml` / `build.gradle*`, `composer.json`, `Gemfile`, `mix.exs`, etc.
- Pick scan roots in this order: `src/` if it exists → otherwise the directories containing manifest files → otherwise the repo root excluding `node_modules`, `dist`, `build`, `bin`, `obj`, `target`, `.git`, `.agents`, `.claude`, `.github`, `vendor`, `.venv`.
- Multi-root layouts are normal (e.g. a .NET solution with `Services/` + `Infrastructure/` + `Gateway/`; a monorepo with `packages/*`). Treat each as a scan root.

Then build a picture of what the codebase actually does. The dimensions to look at depend on the detected stack — let the tech-stack doc and the manifests drive the questions, not a fixed checklist. Generic dimensions worth probing whenever they apply:

- **Module layout** — how the code is split (folders, projects, packages) and what kinds of files cluster together.
- **Entry points & composition** — where the app/service is wired (`Program.cs`, `main.*`, `index.*`, `app.module.*`), including DI/IoC registration patterns, middleware/pipeline ordering, configuration loading.
- **Public surface conventions** — for HTTP services: controller/route/handler shape, request/response envelope, status-code handling, versioning, auth attribute/middleware usage. For libraries: public-API shape, exported names, stability boundary. For CLIs: command registration, flag parsing.
- **Data layer** — ORM/query-builder usage, migration location, repository vs direct-context style, transaction boundaries.
- **Cross-cutting** — error handling, logging, validation, authn/authz, configuration access, caching.
- **Naming & file conventions** — file/class naming patterns (`*Controller`, `*Service`, `*Request`, `*Response`, `use*`, `*Store`, etc.), folder casing, one-type-per-file vs grouped.
- **Test layout** — where tests live, naming pattern, test framework, fixture style.
- **Anything else that appears consistently across ≥2 modules** in the detected layout.

Only investigate dimensions that the stack actually exposes. Skip frontend-only probes (routing library, state store, UI kit, form/validation pairing, animation library) unless the manifests show a frontend framework. Skip backend-only probes (ORM, controller envelope, middleware pipeline) unless the manifests show a server framework. A rule belongs in the doc only when ≥2 examples in this repo support it.

## 3. Review mode — propose a delta, do not edit yet

Produce three lists and show them to the user before changing anything:

- **Existing rules that may not apply anymore** — broken pointers (referenced files don't exist), library/skill/tool names that don't match the detected manifests or the available skill list, terminology that contradicts the source. Flag with reason. Do **not** remove yet.
- **Existing rules that need clarification** — vague rules where the source shows a more specific pattern.
- **Proposed additions** — conventions clearly followed in the source (≥2 examples) but missing from the document.

Ask the user which to apply. Accept answers like "all additions", "all + fixes", or specific item numbers. Then:

- Apply in-place edits for fixes/clarifications.
- Append new rules at the end of the relevant section (e.g. `# General Rules`) preserving the file's existing structure.
- Never silently delete a rule. If the user wants something removed, do it as an explicit separate step.

## 4. Bootstrap mode — create `coding-standard.md` from the scan

When no standardization document exists, run the §2 scan-root detection and signal-gathering, then write `.agents/context/coding-standard.md` with a `# General Rules` section populated from observed patterns. Include only rules supported by ≥2 examples in the source. For each rule, prefer concrete references (folder paths, file names, package names from the detected manifest) over abstract phrasing. Do not impose ecosystem-specific rule categories (frontend/backend, MVC, hooks, etc.) unless the source actually exhibits them.

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

- Do not invent rules. Every rule must be traceable to either an existing document line or an observed source pattern (≥2 examples).
- Do not assume any particular stack or layout shape. No frontend defaults, no backend defaults — derive every probe and every proposed rule from the detected manifests, the documented tech stack, and the observed folder layout.
- Do not edit files outside `.agents/context/` as part of this skill.
- Do not remove existing rules without explicit user approval.
- When in doubt about whether something is a real convention or a one-off, ask before adding.
