---
name: scaffold-module
description: Scaffold a new module that matches the project's established module shape, derived from the detected stack and existing modules
---

# Instructions

Create a new module that fits the project's existing conventions. The shape of a "module" is whatever the project already does — a frontend feature folder, a backend service/controller set, a CLI command, a library package, etc. Do not assume a stack or layer.

## 1. Detect the stack and the established module shape

- Read `.agents/context/tech-stack.md` and `.agents/context/project-structure.md` if they exist. They are the primary source of truth for what a module looks like here.
- If `.agents/context/` names a module-setup guide (e.g. `module-setup-guidelines.md`), read it and treat it as authoritative — its rules override the inferences below.
- Inspect at least two existing modules of the same kind the user is asking to create. Identify:
  - Folder layout (where files live, naming conventions).
  - Entry points (controllers, routes, command registrations, exported APIs, page/route registrations).
  - Companion files (DTOs/requests/responses, validators, tests, migrations, types, schemas, fixtures).
  - DI/registration touch points (`Program.cs` `AddScoped<...>`, router config, plugin manifest, package exports, feature-flag map).
  - Cross-cutting wiring (auth attributes, middleware, logging, error handling, i18n keys).
- Note the project's coding-standard rules from `.agents/context/coding-standard.md` and apply them verbatim (response envelope, route template, auth opt-in pattern, naming, etc.).

If no existing modules of the requested kind exist, ask the user to point at the closest analog rather than inventing a shape.

## 2. Gather the request

Confirm with the user (or extract from the prompt) before generating:

- Module name and a one-line purpose.
- Which surface it exposes (HTTP endpoints, UI route, CLI command, library API, scheduled job, …).
- Inputs/outputs at the boundary, in whatever form the existing modules document them (DTOs, schemas, types).
- Any deviations from the established pattern (explicit user instruction required).

## 3. Plan the scaffold

Produce a short plan listing each file to create and each existing file to touch (DI registrations, route maps, exports, navigation, fixtures). Group by phase so the user can stop or redirect mid-way:

1. Skeleton files matching the established layout.
2. Wiring into composition points (DI, routing, exports).
3. Boundary types (requests/responses/DTOs/schemas/types).
4. Tests at whatever level the existing modules have them.

Show the plan and wait for approval before writing files.

## 4. Implement in phases

Apply each phase, then pass back through to verify cleanliness before moving on:

- Match naming, casing, and folder placement to siblings — do not introduce a parallel convention.
- Reuse shared building blocks (components, helpers, base classes, utilities) where they already exist; only globalize new ones with discretion and only if the established standards encourage it.
- Honor the response envelope, error handling, and auth pattern from `coding-standard.md` exactly.
- For UI work specifically: reuse existing design tokens (colors, typography, spacing), shared components, and module-flow conventions only when the project actually has them. For non-UI work, ignore those concerns entirely.

After each phase, run whatever the project uses for fast feedback (type-check, build, lint, tests — only what's configured), report failures, and fix them before continuing.

## 5. Report

List every file created and every file touched, the composition points wired, and any items the user explicitly opted to defer.

## Guardrails

- Do not introduce new architectural patterns, libraries, or layers without explicit user instruction — match what's already there.
- Do not assume frontend or backend conventions. Apply UI-specific concerns only when the project has a UI; apply server/middleware concerns only when the project has a server.
- Do not skip phases or write all files at once — phased passes catch mismatches early.
- When the established module shape and the user's request conflict, surface the conflict and ask before proceeding.
