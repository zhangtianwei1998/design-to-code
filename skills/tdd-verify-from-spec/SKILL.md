---
name: tdd-verify-from-spec
description: Use after design-to-code:subagent-driven-development completes. Main agent drives playwright-cli (headed, persistent) to verify each acceptance item in spec.md; failures dispatch fixer subagents. MUST be followed by design-to-code:visual-qa-from-design.
---

# TDD Verify from Spec

The main agent drives playwright-cli against the running app to verify each acceptance item in `spec.md`. Failures dispatch fixer subagents; the loop runs until all items pass or thresholds are hit. After all items are verified, this skill hands off to `design-to-code:visual-qa-from-design` for pixel-level visual fidelity checking.

## Plugin-wide discipline (shared HARD-GATE)

- Internal references MUST use `design-to-code:<skill-name>`. References to `superpowers:*` are forbidden.
- Artifact directory is `docs/design-to-code/<YYYY-MM-DD>-<topic>/`. Filenames are fixed: `spec.md`, `plan.md`, `progress.md`, `verify.log.md`.
- Convert this skill's checklist into TodoWrite tasks and execute in order; no skipping.
- `spec.md` is immutable to the assistant; only the user may edit it. `plan.md` is written only by `design-to-code:writing-plans`. `progress.md` is appended only by `design-to-code:subagent-driven-development`. `verify.log.md` is written only by this skill. No skill writes into another skill's artifact.
- Do not write artifact files on `main` / `release`. A feature branch or worktree must exist first.

## Hard gates

- playwright-cli MUST be driven by the main agent (login session stays on the main agent's browser instance).
- Code fixes MUST go through a fixer subagent; the main agent does not edit code.
- `spec.md` is immutable here. The assistant MUST NOT weaken or mutate acceptance items to make verification pass.
- Every acceptance item is recorded as pass/fail in `verify.log.md`.
- Every failure MUST be classified as either Category A (code-in-repo issue that a fixer could plausibly resolve) or Category B (out-of-repo issue: test case itself ambiguous or wrong, backend data anomaly, environment/dependency issue, or underlying `spec.md` defect). The classification determines the failure threshold (see below).
- Internal references only `design-to-code:*`.

## Process

1. **Pre-flight** — ensure the dev server is running (if not, ask the user how to start or start it per project conventions). Ensure `@playwright/cli` is installed (install on miss via `npm install -g @playwright/cli@latest`).

2. **Launch browser** — `playwright open <dev-url> --headed --persistent`. Prompt the user to log in if not already authenticated.

3. **Per-item verification loop** — for each acceptance item in `spec.md`, in order:
   - Drive playwright to execute the described interaction and make the described assertion (DOM inspection + screenshot reasoning).
   - On pass: record ✅ in `verify.log.md` and proceed.
   - On fail:
     1. Record ❌ with the observed evidence.
     2. **Classify** the failure as Category A or Category B (see next section).
     3. For Category A: dispatch a fixer subagent (see `./fixer-prompt.md`). When the fixer returns, wait briefly if HMR, then re-run the SAME acceptance item.
     4. For Category B: do NOT dispatch a fixer. Record the category and reasoning in `verify.log.md`. Optionally gather a little more evidence (e.g. one extra run to rule out flake); do not attempt a code change.

4. **Completion** — after all items pass (or are recorded as Category B), invoke `design-to-code:visual-qa-from-design` to run pixel-level visual fidelity verification against the original design source. Pass the `spec.md` path so the visual-qa skill can locate the design reference.

## Failure classification and thresholds

- **Category A — code-in-repo**: defect looks fixable by editing files in the current repo (component, slice, API call, styling, routing, etc.). Threshold: **5 consecutive rounds** on the same acceptance item pauses the loop and reports to the user.
- **Category B — out-of-repo**: test case itself ambiguous or wrong, backend data anomaly, environment/dependency issue, `spec.md` defect. Threshold: **2 consecutive rounds** on the same acceptance item pauses the loop and reports to the user. Do NOT dispatch a fixer for Category B failures.
- When uncertain about classification, treat as Category B (fail-fast) rather than burning rounds on Category A.
- An item may change category across rounds; use the threshold matching the current round's category.

## Prompt files

- `./verification-loop-prompt.md` — main agent's self-reference for driving playwright
- `./fixer-prompt.md` — sent to fixer subagents (Category A failures only)

## Integration

**Required workflow skills:**
- **design-to-code:subagent-driven-development** — runs before this skill; produces the implementation.
- **design-to-code:visual-qa-from-design** — runs after this skill; checks pixel-level visual fidelity against the original design.

## Artifacts

- `verify.log.md` (committed to git by the user's project). Per-item entry:

  ```markdown
  ## Acceptance N: <item>
  - Attempts: <count>
  - Result: ✅ / ❌
  - Category (only on ❌): A (code-in-repo) / B (out-of-repo)
  - Category reasoning (only on ❌): <one-line justification>
  - Evidence: <screenshot path or DOM excerpt>
  - Fixes applied: <files changed by fixer, if any>
  ```
