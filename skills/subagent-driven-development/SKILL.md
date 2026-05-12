---
name: subagent-driven-development
description: Use after design-to-code:writing-plans completes. Dispatches implementer subagents per task; each task passes spec-review and code-quality-review before completion; a final-review covers the whole set. Main agent never writes code. MUST be followed by design-to-code:tdd-verify-from-spec.
---

# Subagent-Driven Development

Execute `plan.md` by dispatching fresh subagents per task with two-stage review (spec + code quality) after each, and a final-review pass before hand-off. The main agent orchestrates; it never edits code directly.

## Plugin-wide discipline (shared HARD-GATE)

- Internal references MUST use `design-to-code:<skill-name>`. References to `superpowers:*` are forbidden.
- Artifact directory is `docs/design-to-code/<YYYY-MM-DD>-<topic>/`. Filenames are fixed: `spec.md`, `plan.md`, `progress.md`, `verify.log.md`.
- Convert this skill's checklist into TodoWrite tasks and execute in order; no skipping.
- `spec.md` is immutable to the assistant; only the user may edit it. `plan.md` is written only by `design-to-code:writing-plans`. `progress.md` is appended only by `design-to-code:subagent-driven-development` (exception: `design-to-code:tdd-verify-from-spec` may append a single "verification complete" section at the very end).
- Do not write artifact files on `main` / `release`. A feature branch or worktree must exist first.

## Hard gates

- Main agent MUST NOT edit code directly. All code goes through an implementer subagent.
- Each task must pass both `spec-reviewer` and `code-quality-reviewer` before being marked complete.
- After all tasks complete, a `final-reviewer` pass must pass before hand-off.
- Internal references only `design-to-code:*`.

## Process per task

```
dispatch implementer (./implementer-prompt.md)
  └─ implementer has questions? → main agent answers → redispatch
  └─ implementer implements, self-tests, self-reviews, returns
dispatch spec-reviewer (./spec-reviewer-prompt.md)
  └─ fails? → redispatch implementer with feedback → re-review
dispatch code-quality-reviewer (./code-quality-reviewer-prompt.md)
  └─ fails? → redispatch implementer with feedback → re-review
mark task completed in TodoWrite; append to progress.md
```

## Final review

After all tasks are complete:

```
dispatch final-reviewer (./final-reviewer-prompt.md)
  └─ passes? → hand off to design-to-code:tdd-verify-from-spec
  └─ fails? → locate the offending task(s) and re-enter per-task loop
```

## Failure thresholds

If the same task fails the same reviewer stage for **2 consecutive rounds**, pause and report to the user. Do not silently continue.

## Concurrency

Multiple ready tasks (no open `blockedBy`) whose `Files` do not overlap may be dispatched in parallel (implementers in one message with multiple Agent tool calls). Review stages remain serial to keep feedback tractable.

## Prompt files

- `./implementer-prompt.md` — sent to implementer subagents
- `./spec-reviewer-prompt.md` — sent to spec-compliance reviewer subagents
- `./code-quality-reviewer-prompt.md` — sent to code-quality reviewer subagents
- `./final-reviewer-prompt.md` — sent to the final reviewer subagent after all tasks complete

## Artifacts

- `progress.md` (committed to git by the user's project). Per-task entry:

  ```markdown
  ## Task N: <name>
  - Status: completed
  - Subagent attempts: <implementer rounds>
  - Spec-review rounds: <n>
  - Quality-review rounds: <n>
  - Files changed: <list>
  - Acceptance: ✅ <each item>
  - Notes: <deviations worth recording>
  ```
