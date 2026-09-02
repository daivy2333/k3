## OpenSpec workflow rules

Before planning, implementing, reviewing, or updating project documentation:

1. Read `CLAUDE.md` completely.
2. Treat its OpenSpec roles, Gates, BDD, TDD, verification, and editing rules as mandatory.
3. Load only the skill references required by the active task.
4. Do not copy those rules into this file; `CLAUDE.md` is their single source.
5. When the platform automatically resumes pending work, re-check the nearest authorization, capability, or stop boundary before continuing.

## Skill entry point

- Codex: skills resolve from `.agents/skills/<name>/SKILL.md` (symlinked to `.claude/skills/`).
- OpenCode: prefers `.agents/skills/`, falls back to `.claude/skills/`.
- Claude Code: skills resolve from `.claude/skills/<name>/SKILL.md` directly.

The skill content is identical across all three platforms; do not edit per-platform copies.
