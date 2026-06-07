# Codex / Generic Agent Instructions

This project keeps all agent-facing context in `.agents/`. Read those files before acting; this file is just a pointer.

## Required reading

- `.agents/README.md` — index of instructions. read first before anything else.

## Skills / playbooks

Treat `.agents/skills/<name>/SKILL.md` as task playbooks. When the user's request matches a skill name, follow that file's instructions and consult its `references/` folder for additional instructions.

## House rules
- Keep `.agents/` as the single source of truth — do not duplicate context into `CLAUDE.md`, `AGENTS.md`, or `.github/copilot-instructions.md`.

