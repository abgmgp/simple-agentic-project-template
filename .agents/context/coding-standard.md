# General Rules

- Prefer editing existing modules over creating parallel ones; consult the structure/architecture reference file in `.agents/context/` for clear guidelines.
- Optimization policies should be followed only if it does not break the existing functionality and established guidelines.
- Do not go out of requested scope. Be concise and clear with the delivery and fixing. Only do what is being asked unless stated.
- Always ask the user for additional instructions if something is unclear or needs to be checked.
- Error handling should be proper and consistent, and aligned with the globalized implementation.
- Code format should follow the tech stack's preferred structure and design principles.

[list goes on...]

# Skill Preference

- Always run `scaffold-module` skill when adding a new module.
- Mentions relating to `creating a pr` or `creating a pull request` will be executed with the `pr` skill.
- For Github remotes, run the `ci-git` skill if the user wants to test a build for the release workflow.