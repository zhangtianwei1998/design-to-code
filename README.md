# design-to-code

A Claude Code plugin that turns a design source (URL or image) plus explicit implementation intent into a verified implementation, through a 4-stage, linear, spec-driven workflow.

## Trigger

The plugin activates when a single message contains:

1. A design URL **or** an attached image, AND
2. Explicit implementation intent ("实现", "做出来", "还原", "照这个写", "implement this", "build this").

Only one of the two? The assistant will ask before entering the workflow.

## Skills

| # | Skill | Purpose | Input | Output |
|---|-------|---------|-------|--------|
| 1 | `brainstorming-from-design` | Entry point. Understand the design + user intent | design URL / image | `spec.md` |
| 2 | `writing-plans` | Decompose into subagent-ready tasks | `spec.md` | `plan.md` |
| 3 | `subagent-driven-development` | Implement with per-task spec + quality review and final review | `plan.md` | code + `progress.md` |
| 4 | `tdd-verify-from-spec` | Drive playwright to verify each acceptance item | `spec.md` + running app | `verify.log.md` |

## Artifacts

All produced under `docs/design-to-code/<YYYY-MM-DD>-<topic>/` in the user's project:

- `spec.md` — produced by skill 1; user-approved; **never** edited by the assistant afterward
- `plan.md` — produced by skill 2
- `progress.md` — appended by skill 3 (one entry per task)
- `verify.log.md` — appended by skill 4 (one entry per acceptance item)

## Manual entry from later stages

- Have `spec.md` already → invoke `design-to-code:writing-plans` directly.
- Have `plan.md` already → invoke `design-to-code:subagent-driven-development` directly.
- Want to re-verify an existing implementation → invoke `design-to-code:tdd-verify-from-spec` directly.

## Dependencies

- Required on host: `node`, `pnpm`, `@playwright/cli` (auto-installed by skills 1 and 4 if missing).
- Optional, for the tail hand-off only: `.cursor/skills/test-plan-writer`, `.cursor/skills/test-case-generator`. Absent → skill 4 skips the tail gracefully.

## Relationship to superpowers

Inspired by superpowers conventions (subagent-driven-development, test-driven-development, writing-plans) but this plugin is **self-contained**. It does not depend on superpowers at runtime and does not reference the superpowers plugin from any `SKILL.md`.

## Troubleshooting

- **Playwright install fails** — try `npm install -g @playwright/cli@latest` manually; ensure node >= 18.
- **Login session lost between verifications** — confirm the browser was opened with `--headed --persistent`. Non-persistent sessions lose cookies.
- **Fixer loops 5 times without success** — the skill pauses and reports. Treat this as a signal to redesign the task manually, or to split the acceptance item into smaller sub-items.

## Installation

Local install (no marketplace):

```bash
# from the plugin's parent directory
ln -s "$(pwd)/design-to-code" ~/.claude/plugins/local/design-to-code
```

Then enable in your Claude Code settings.
