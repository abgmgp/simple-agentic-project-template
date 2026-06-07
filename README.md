# Simple Agentic Project Template

This template serves as a minimal baseline for AI-powered projects. It provides a single source of truth for agent context (`.agents/`) and thin pointer files so that all agents read the same instructions, standards, and skill playbooks. Currently supports Claude Code, Codex CLI, and GitHub Copilot.

## Purpose

Most AI-assisted repos get messy when the maintainers shift tools: each one gets its own reference file and the rules become harder to maintain over time, especially in larger, well-defined projects. This template fixes that by:

- Keeping all agent-facing context in the `.agents/` directory.
- Having the main directory exposed to each agent via small pointer files at the repo root (`CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`).
- Shipping a starter set of skills (`ci`, `cleanup`, `code-standard`, `passcheck`, `pr`, `project-structure`, `scaffold-module`, `stack`) that allow maintainers for further customization. Details for each skill will be discussed below.

Use it as the starting point for any new AI-assisted project where you want consistent agent behavior from day one.

## What's Inside

```
.agents/
  README.md                 # index — agents read this first
  context/                  # project context files
  skills/<name>/SKILL.md 
.claude/
  skills/<name>/SKILL.md    # Claude-specific prompts that point back to the default skill directory
  settings.local.json
.github/
  copilot-instructions.md   # Github Copilot pointer
  workflows/
CLAUDE.md                   # Claude Code pointer
AGENTS.md                   # Codex CLI pointer
```

## Included Skills

The template ships with a starter set of skill playbooks under `.agents/skills/`. Some of the skills provided are for general development use case, while others are for template and context maintenance. The skills are setup to automatically support most mdodern frontend/backend tech stacks for general usage. 

- **`ci`** - Creates a GitHub Actions workflow at `.github/workflows/ci.yml` that runs on push and pull request activity against the `main` or `master` branch.
- **`cleanup`** - Scans the source for leftover debug calls, redundant comments, and dead code from development commits, then removes them with user confirmation and validates with a test build.
- **`code-standard`** - Reviews and revises the coding standard documents under `.agents/context/` against the actual source, or creates one when none exists.
- **`passcheck`** - Audits the source for code that does not comply with the standards and architecture documented under `.agents/context/`.
- **`pr`** - Opens a pull request to `main`/`master` on GitHub or Azure workflows with a summary, implemented changes, and a test checklist.
- **`project-structure`** - Reviews and revises the project structure/architecture document under `.agents/context/` to match the actual source layout, or creates a new one if no template is detected.
- **`scaffold-module`** - Scaffolds a new module from a description, following the established module setup guidelines and the project's existing architecture conventions.
- **`stack`** — Reviews and revises the tech stack document under `.agents/context/` against currently used packages and dependencies, flagging security risks and deprecated defaults (if any).

## How to Use

1. **Clone the template.**
   ```bash
   git clone https://github.com/<your-org>/simple-agentic-project-template.git my-new-project
   cd my-new-project
   rm -rf .git && git init
   ```

3. **Fill in `.agents/context/`.**
   - `coding-standard.md` - language/style rules your agents must follow.
   - `project-structure.md` - directory layout and module boundaries.
   - `tech-stack.md` - frameworks, libraries, versions.

4. **Trim or add skills.** Delete any skill under `.agents/skills/` you don't need, and remove its stub under `.claude/skills/`. To add one:
   - Create `.agents/skills/<name>/SKILL.md` with property (`name:`, `description:`).
   - Create `.claude/skills/<name>/SKILL.md` stub pointing at the default file.

5. **(Optional) Setup the context files.** If migrating for the first time, run the following skills in order: `stack`, `project-structure`, then `code-standard` for last. Once completed tweak the generated files according to your preference.

6. **(Optional) Modify the scaffold-module reference.** If you want your scaffolding skill to be more specific with your current development process, edit the `.agents/skills/scaffold-module/references/module-setup-guidelines.md` to your intended use case.

7. **Start working.** Open the project in Claude Code, Codex CLI, or Copilot CLI. They should pick up the pointer file and load the `.agents/` context automatically.

## Contributions

For feedback and additional contributions, please feel free to raise an issue or create a pull request. 