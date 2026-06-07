# Claude Code Instructions

This project keeps all agent-facing context in `.agents/`. Read those files before acting; this file is just a pointer.

## Required reading

- `.agents/README.md` — index of instructions. read first before anything else.

## Skills

Canonical skill bodies live in `.agents/skills/<name>/SKILL.md`. The stubs under `.claude/skills/<name>/SKILL.md` exist only so Claude Code's Skill tool can discover them — each stub references its canonical source. When updating skill behavior, edit the file under `.agents/skills/`, not the stub.

## House rules
- Keep `.agents/` as the single source of truth — do not duplicate context into `CLAUDE.md`, `AGENTS.md`, or `.github/copilot-instructions.md`.

