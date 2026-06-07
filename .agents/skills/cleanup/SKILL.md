---
name: cleanup
description: Cleanup leftover debug logging, redundant comments, and dead code left over from development commits
---

# Instructions

Remove development-time noise (debug logging, leftover comments, dead code) from the source. The exact patterns to look for depend on the detected stack — do not assume a language.

## 1. Detect the stack

Consult `.agents/context/tech-stack.md` first if it exists. Then check manifests at the repo root and one level down (`package.json`, `pyproject.toml`/`setup.py`, `*.csproj`/`*.sln`, `go.mod`, `Cargo.toml`, `pom.xml`/`build.gradle*`, `composer.json`, `Gemfile`, `mix.exs`, etc.). Use the result to choose:

- **Debug-logging patterns** per language. Examples — extend or replace based on the actual stack:
  - JavaScript/TypeScript: `console.log`, `console.debug`, `console.warn`, `console.error`, `debugger;`
  - Python: bare `print(...)`, `breakpoint()`, `pdb.set_trace()`
  - C#/.NET: `Console.WriteLine`, `Debug.WriteLine`, `Debugger.Break()` (but **not** `ILogger` calls — those are intentional)
  - Go: `fmt.Println`, `log.Println` in non-`main` packages, `spew.Dump`
  - Rust: `println!`, `dbg!`, `eprintln!`
  - Java/Kotlin: `System.out.println`, `e.printStackTrace()`
  - PHP: `var_dump`, `print_r`, `dd(...)`, `dump(...)`
  - Ruby: `puts`, `p`, `binding.pry`, `byebug`
- **Test/build command** for the post-cleanup verification step (from the same detection — see §4).

If no recognizable stack is found, ask the user what to grep for and what command to run for verification.

## 2. Scan the source

Restrict the scan to the actual source roots (derive from `.agents/context/project-structure.md` if present, else from the detected manifests' directories). Exclude `node_modules`, `dist`, `build`, `bin`, `obj`, `target`, `vendor`, `.venv`, `.git`, `.agents`, `.claude`, `.github`, and any `*/Migrations/*` / generated-code paths.

For each pattern category, collect:

- **Debug logging** — exact matches of the chosen patterns.
- **Redundant comments** — TODO/FIXME/XXX/HACK markers, commented-out blocks of code, comments that just restate the next line, dated developer notes. Treat anything load-bearing (license headers, public API docs, `// SPDX-...`, `# pragma` directives, suppression comments like `// eslint-disable-next-line`, `# noqa`, `#nullable enable`) as **off-limits**.
- **Dead code** — unreachable branches after `return`/`throw`, commented-out blocks, obviously duplicated adjacent lines. Be conservative; static analysis isn't run here.

## 3. Report before changing anything

Provide a count summary, then per-category groupings with file:line references. For large result sets, show the top offenders per category and offer to dump the full list.

Ask the user which categories (and optionally which specific items) to clean. For comments with non-trivial likelihood of being intentional (anything other than a clearly-commented-out code block or a bare TODO with no context), require per-item confirmation rather than batch approval.

## 4. Apply cleanups, then verify

After applying the approved removals, run the project's validation command(s) to confirm nothing broke. Pick from the detected stack:

- Node: prefer the `ci` skill or run scripts that exist in `package.json` (`lint`, `build`, `test`, `typecheck`). Skip ones not defined.
- .NET: `dotnet build <sln> -c Release` and, if any test project exists, `dotnet test <sln> -c Release`.
- Python: whichever of `ruff check`, `mypy`, `pytest` are configured.
- Go: `go vet ./... && go build ./... && go test ./...`.
- Rust: `cargo build && cargo test`.
- Java/Kotlin: `./gradlew build` or `mvn -B -ntp verify`.
- Otherwise: ask the user.

If any errors surface, report them with suggested fixes and iterate until verification passes or the user calls it.

## Guardrails

- Never strip structured logging calls (`ILogger`, `slog`, `tracing`, `winston`, `pino`, `log/slog`, etc.) — those are production telemetry, not debug noise.
- Never remove suppression/pragma comments, license headers, or public-API docstrings.
- Do not auto-clean comments without category-level user approval; do not auto-clean ambiguous comments without per-item confirmation.
- Do not edit generated code (migrations, protobuf output, `*.Designer.cs`, `*.g.cs`, `*_pb2.py`, etc.).
- If the detected stack is unclear, ask before running anything.
