---
name: writing-plans
description: Use after design-to-code:brainstorming-from-design completes. Reads spec.md and produces plan.md decomposed into functional units. MUST be followed by design-to-code:subagent-driven-development.
---

# Writing Plans

Turn `spec.md` into an implementation `plan.md`, decomposed into functional units, each executable by a single subagent. This skill writes documentation only; no production code.

## Plugin-wide discipline (shared HARD-GATE)

- Internal references MUST use `design-to-code:<skill-name>`. References to `superpowers:*` are forbidden.
- Artifact directory is `docs/design-to-code/<YYYY-MM-DD>-<topic>/`. Filenames are fixed: `spec.md`, `plan.md`, `progress.md`, `verify.log.md`.
- Convert this skill's checklist into TodoWrite tasks and execute in order; no skipping.
- `spec.md` is immutable to the assistant; only the user may edit it. `plan.md` is written only by `design-to-code:writing-plans`. `progress.md` is appended only by `design-to-code:subagent-driven-development` (exception: `design-to-code:tdd-verify-from-spec` may append a single "verification complete" section at the very end).
- Do not write artifact files on `main` / `release`. A feature branch or worktree must exist first.

## Hard gates

- No production code changes; this skill only produces documentation.
- Do NOT generate a second test document. `spec.md`'s acceptance checklist is the single source of truth; `plan.md` *references* it, does not *copy* it.
- Internal references only `design-to-code:*`.

## Checklist

1. **Read `spec.md` in full.** Do not defer this to subagents; the main agent must hold the whole design in mind to decompose well.

2. **Code exploration** — explore the codebase enough to decompose with confidence. Depth is bounded by "enough to split tasks"; do not aim for exhaustive coverage.

3. **Decompose into functional units.** Each task must satisfy:
   - Clear boundary — input/output/changed-files statable in one line.
   - Sized for a single subagent context (typically ≤ 3–5 tightly coupled files).
   - Has a concrete completion criterion, either referencing a `spec.md` acceptance item or a local check (type-check passes, this task introduces no compile errors).
   - Dependencies marked explicitly via `blockedBy`.

   Common decomposition axes (pick what fits; not a mandate): backend contract / types → frontend API slice → business components → route & entry wiring → styling & i18n.

4. **Produce `plan.md`** at `docs/design-to-code/<YYYY-MM-DD>-<topic>/plan.md`:

   ```markdown
   # Implementation Plan: <topic>

   **Spec**: ./spec.md
   **Created**: YYYY-MM-DD

   ## Overview
   <2-3 sentences: overall approach and rationale for this decomposition>

   ## Shared Context
   <Project conventions every subagent needs: framework stack, path
    aliases, linting rules in force, state-management conventions, i18n
    conventions — whatever THIS project requires. The main agent
    decides the contents based on project specifics; no fixed list.>

   ## Tasks

   ### Task 1: <name>
   - **Goal**: <one line>
   - **Files**: <paths in scope>
   - **Depends on**: none
   - **Acceptance**:
     - Satisfies spec.md item: "<quoted>"
     - Type-check passes
   - **Notes for subagent**: <hints, e.g. reference patterns>

   ### Task 2: ...
   ```

5. **Plan self-review** — ensure:
   - Each task is independently deliverable.
   - Dependency graph is acyclic and reasonable.
   - Every `spec.md` acceptance item is covered by at least one task's Acceptance.
   - No scope additions beyond `spec.md`. Remove any.

6. **User approval** — `Read` the `plan.md` into the conversation. On changes, return to step 4.

7. **Hand off** — invoke `design-to-code:subagent-driven-development`, passing the `plan.md` path.

## Artifacts

- `plan.md` (committed to git by the user's project)
