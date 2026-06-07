---
name: stack
description: Review and revise the project's tech stack document in .agents/context against the detected package manifests, flagging security risks and enforced deprecations
---

# Instructions

Keep the project's tech stack document in sync with what is actually installed across whichever ecosystem(s) the repo uses, and surface package risks. Pick mode automatically based on what's in `.agents/context/`. Do not assume a specific stack.

## 1. Detect the current state

List markdown files under `.agents/context/`:

```bash
find .agents/context -maxdepth 2 -type f -name '*.md'
```

- **Bootstrap mode** — no tech-stack document present. Go to §5.
- **Review mode** — at least one tech-stack-style document exists. Go to §2.

Do not hardcode `tech-stack.md` as the only valid name. Treat any file whose content lists installed packages / framework versions as the tech-stack document.

## 2. Detect the ecosystem(s)

Inspect manifests at the repo root and one level down. Multi-ecosystem repos are normal — process each one that's present.

| Ecosystem | Manifest signal | Authoritative for versions | Risk-check commands |
|---|---|---|---|
| Node.js | `package.json` (+ `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`) | the lockfile (else `package.json` ranges) | `npm audit --json`, `npm outdated --json`, `npm view <pkg> deprecated` |
| .NET | `*.sln` / `*.csproj` / `Directory.Packages.props` (+ optional `global.json`) | `Directory.Packages.props` if present, else each `*.csproj` `<PackageReference>` | `dotnet list package --vulnerable --include-transitive`, `dotnet list package --deprecated`, `dotnet list package --outdated` |
| Python | `pyproject.toml` / `setup.py` / `requirements*.txt` (+ `poetry.lock` / `uv.lock` / `Pipfile.lock`) | the lockfile if present | `pip-audit`, `pip list --outdated`, or `uv pip list --outdated` / `poetry show --outdated` |
| Go | `go.mod` / `go.sum` | `go.sum` | `govulncheck ./...`, `go list -u -m all` |
| Rust | `Cargo.toml` / `Cargo.lock` | `Cargo.lock` | `cargo audit`, `cargo outdated` |
| Java/Kotlin (Maven) | `pom.xml` | resolved POM | `mvn -B -ntp dependency:tree`, `mvn versions:display-dependency-updates`, `mvn org.owasp:dependency-check-maven:check` |
| Java/Kotlin (Gradle) | `build.gradle*` (+ `gradle.lockfile`) | the lockfile if present | `./gradlew dependencyUpdates`, `./gradlew dependencyCheckAnalyze` |
| PHP | `composer.json` (+ `composer.lock`) | `composer.lock` | `composer audit`, `composer outdated` |
| Ruby | `Gemfile` (+ `Gemfile.lock`) | `Gemfile.lock` | `bundle audit`, `bundle outdated` |

If none of these are present, stop and tell the user there is no recognizable package manifest to review against.

## 3. Review mode — reconcile the doc against the manifests

For every detected ecosystem, read each direct dependency (and its declared version range or pinned version) from the manifests above. Capture the dependency/devDependency split where the ecosystem distinguishes them (Node `dependencies` vs `devDependencies`, .NET `<PackageReference>` in src vs test projects, Python `dependencies` vs `dev-dependencies`/extras, etc.).

In .NET specifically, also capture per-project version drift — the same package referenced at different versions across `*.csproj` files when no `Directory.Packages.props` exists. This is .NET-specific and easy to miss.

Compare against the doc:

- **Missing in doc** — packages referenced by the manifest(s) not mentioned.
- **Stale in doc** — packages in the doc no longer in the manifest(s), or with a version that no longer matches.
- **Drifted classification** — doc lists a package under the wrong role (e.g. listed under Testing when it's a runtime dep).
- **Per-project / per-package version drift** — same package pinned at different versions across files when central package management is absent.

## 4. Review mode — security & deprecation pass

Run the risk-check commands for each detected ecosystem from the §2 table. Inspect results and surface:

- **Security risks** — vulnerabilities at `moderate` or higher (`Moderate` in .NET, `MEDIUM` in some scanners). Include package name, severity, advisory URL when available, and project(s) affected.
- **Enforced deprecations** — packages flagged as deprecated by the registry. Include the deprecation reason and suggested replacement when provided. For .NET, `dotnet list package --deprecated` gives this directly; for Node, `npm view <pkg> deprecated` for top-level deps the doc names.
- **Major behind** — packages where the resolved version is a full major version behind latest (informational; not auto-fail).
- **Ecosystem-specific footguns**:
  - .NET: `Microsoft.AspNetCore.App` as a `PackageReference` on a modern TFM (should be implicit framework reference); `Microsoft.AspNetCore.*` major older than the project `TargetFramework`; out-of-support TFMs (`netcoreapp2.2`, `net5.0`, `net6.0` past EOL).
  - Node: packages stuck on a version flagged `deprecated: see <replacement>` in the registry; presence of both `npm` and `pnpm` lockfiles (ambiguous toolchain).
  - Python: `setup.py`-only projects in a `pyproject.toml`-first ecosystem; unpinned dev tools in a lockfile-using project.
  - Go: modules with `go.mod` `go` directive newer than the installed toolchain.
  - General: presence of any pre-1.0 (`0.x`) packages in the doc — pin and monitor.

Report each finding with a one-line recommendation (upgrade target, replacement package, framework cleanup, or "monitor"). Treat security and enforced-deprecation findings as **must-inform** even if the user only asked for a doc review.

If a risk-check command is unavailable in the environment, say so and skip just that command — still perform the static checks (version drift, ecosystem-specific footguns, out-of-support runtimes) from the manifest contents alone.

## 5. Review mode — propose a delta, do not edit yet

Show four lists plus the risk report before changing anything:

- **Existing entries that may not apply anymore** — stale packages, wrong versions, wrong classification.
- **Existing entries that need clarification** — vague groupings or outdated descriptions.
- **Proposed additions** — packages present in the manifest(s) but missing from the doc.
- **Risk report** — security, deprecations, major-behind, ecosystem-specific footguns, version drift.

Ask which doc changes to apply. Accept "all additions", "all + fixes", or specific item numbers. Then:

- Apply in-place edits for fixes/clarifications.
- Append new entries under the appropriate group, matching the doc's existing grouping convention. If the doc has no groups yet, use the bootstrap defaults from §6.
- Never silently delete a stale entry. Confirm removal explicitly.

The risk report is informational — do not edit packages, run installs, or modify manifests as part of this skill unless the user explicitly asks.

## 6. Bootstrap mode — create the tech stack doc from the manifests

When no tech-stack document exists, generate `.agents/context/tech-stack.md` from the detected ecosystem(s):

- Lead with a one-line scope note describing the repo (derive it: e.g. ".NET microservice layer", "React SPA", "Python data pipeline", "Go service + Node admin UI"). State what the doc covers and that it lists only what matters architecturally — not every transitive dependency.
- Group packages by **purpose**, not by ecosystem. Pick group names from what the manifests actually justify — do not impose a fixed template. Common groups to consider when applicable:
  - **Runtime / framework** — language runtime/TFM, server/web framework, API versioning, reverse proxy.
  - **Service composition** — gateway/services/shared building blocks (derive from project references / workspace layout, not from package lists).
  - **Data & persistence** — ORM/query builder, DB drivers, vector stores, migrations tool.
  - **UI** — UI kit, design system, icon set, animation, styling — only if the repo has a UI.
  - **State / forms** — state management, form library, validation library — only when present.
  - **Identity / auth** — JWT/OIDC/IdentityServer/OpenIddict/Passport/etc.
  - **Cross-cutting** — OpenAPI/Swagger, logging (Serilog/winston/structlog/zap), telemetry, validation, configuration.
  - **Testing** — test framework, runners, coverage, mocking, container fixtures.
  - **Tooling** — central package management presence/absence, runtime SDK pin, analyzers/linters/formatters, source generators.
- For multi-ecosystem repos, prefix lines that are ecosystem-specific with the runtime name when grouping by purpose would otherwise be ambiguous (e.g. `- (Node) react 19.x`).
- One line per package: `- <name> <version>` (optionally a short parenthetical role only when the role isn't obvious from the name). When the same package is referenced at multiple versions (e.g. in .NET without CPM), show the drift inline (e.g. `9.0.4 (RAG pinned to 9.0.1)`).
- Keep it terse — this file is meant to be token-light context for agents. Skip transitive-only noise.

After writing, run §4 once and show the user any risks before treating it as the new baseline.

## Guardrails

- Truth lives in the manifests (and the lockfile / `Directory.Packages.props` when present). The doc is a summary, not a source of versions.
- Do not run installs (`npm install`, `dotnet add package`, `pip install`, `cargo add`, etc.), modify manifests, or change lockfiles as part of this skill.
- Do not assume a single ecosystem. Process every detected manifest set; do not impose frontend or backend defaults that the repo doesn't show.
- Do not invent groupings the repo doesn't justify (e.g. don't add a "UI" group to a pure backend service repo, or a "Data & persistence" group to a static-site repo).
- If a security or enforced-deprecation finding appears, surface it even if the user's request was only "review the doc".
