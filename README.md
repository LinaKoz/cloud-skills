# claude-skills

Lina's custom skills for Claude. Each folder under `skills/` is a self-contained skill with a `SKILL.md` (YAML frontmatter + instructions) and optional `references/`.

| Skill | What it does |
|---|---|
| [ai-coding-interview](skills/ai-coding-interview/SKILL.md) | Pair-programming copilot for live AI-allowed coding interviews: clarify → read context → plan & design review → requirements-based tests → test review → staged implementation → final verification. |
| [system-design-interview](skills/system-design-interview/SKILL.md) | Copilot for live system design interviews: requirements, structure, trade-offs, failure modes — supports your reasoning rather than replacing it. |
| [council](skills/council/SKILL.md) | Adversarial "Council of 5 Experts" red-team review of architectures, docs, decisions. References: `architecture.md`, `general.md`. |
| [harsh-review](skills/harsh-review/SKILL.md) | Claude Code skill: adversarial review of uncommitted changes / a commit / a diff across correctness & concurrency, security, reliability & cost, API contract & test coverage. Every finding is verified empirically (CONFIRMED / PLAUSIBLE) and reported via ReportFindings; review-only unless fixes are requested. Local copy of `~/.claude/skills/harsh-review`. |
| [pr-review-council](skills/pr-review-council/SKILL.md) | Multi-angle PR/diff review with 3 parallel subagents (concurrency & shutdown, auth & secrets, cost & reliability), file:line findings ranked by severity. |

## Using a skill

- **Claude (web/desktop/Cowork):** Settings → Capabilities → Skills → upload the skill folder (zipped).
- **Claude Code:** copy or symlink a folder into `~/.claude/skills/` (personal) or `.claude/skills/` (per project):
  ```bash
  ln -s "$PWD/skills/council" ~/.claude/skills/council
  ```

## Packaging

```bash
cd skills && zip -r ../council.zip council
```

## Editing

The copy in your Claude account is the one Claude actually uses. After editing here, re-upload the skill (or update it through Claude), and keep this repo as the source of truth / history.
