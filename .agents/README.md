# Agent Workspace

Single source of truth for AI agent context. Read by Claude Code, Codex CLI, and GitHub Copilot via thin pointer files at the repo root.

## Layout

- `context/coding-standard.md` — project coding standard knowledge that applies to every agent. always follow.
- `context/project-structure.md` — project structure standard knowledge that applies to every agent. always follow.
- `context/tech-stack.md` — technical stack used for development. always follow.
- `skills/` — task-specific playbooks (canonical copies)

## Pointer files (do not edit content here — edit under `.agents/`)

- `/CLAUDE.md` — Claude Code entrypoint
- `/AGENTS.md` — Codex CLI entrypoint
- `/.github/copilot-instructions.md` — GitHub Copilot entrypoint
- `/.claude/skills/<name>/SKILL.md` — per-skill stubs so Claude's Skill tool still discovers them; each stub references the canonical body under `.agents/skills/<name>/`.

## When adding a new skill

1. Create `.agents/skills/<name>/SKILL.md` (with frontmatter `name:` and `description:`).
2. Create `.claude/skills/<name>/SKILL.md` stub with the same frontmatter and an `Instructions` block pointing to the canonical file.

## When adding new project context

Drop it in `.agents/context/`. All three agents will pick it up through their pointer files.

# Golden Rule 

These rules go hand in hand with the specific rules provided in your agent context file (CLAUDE.md, AGENTS.md). Always follow the related steps. Any overrides from those files will take precedence over related instructions from here.

- Mind over metal. Spirit over monotony.
- Do not go out of requested scope. Be concise and clear with the delivery and fixing. Always ask the user for additional instructions if something is unclear or needs to be checked.
- Assume the user knows basic coding principles and terms. Do not overload with basics.
- Always follow declared technical stack. Check with the tool's documentation for proper implementation guidelines for items with larger scope.