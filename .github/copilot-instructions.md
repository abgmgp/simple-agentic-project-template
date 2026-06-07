# GitHub Copilot Instructions

All agent-facing context lives in `.agents/`. Read those files before generating non-trivial changes; this file is just a pointer.

## Required reading

- `.agents/README.md` — index of instructions. read first before anything else.

## Skill playbooks

When a request matches one of the following, follow the corresponding playbook under `.agents/skills/<name>/SKILL.md`:

## House rules

- `.agents/` is the single source of truth. Do not duplicate context into this file or other pointer files.
