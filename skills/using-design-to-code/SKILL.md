---
name: using-design-to-code
description: Use at the start of any conversation where the user provides a design URL or image AND expresses implementation intent ("实现", "做出来", "还原", "照这个写", "implement this", "build this"). Explains the 4-stage workflow and dispatches to design-to-code:brainstorming-from-design as the first step. Do NOT activate when URL or image appears without implementation intent.
---

# Using Design-to-Code

This is the entry point for the `design-to-code` workflow. It converts a design source (URL or image) plus explicit implementation intent into a verified implementation through four linear skills:

1. `design-to-code:brainstorming-from-design` — design → `spec.md` (user-approved)
2. `design-to-code:writing-plans` — `spec.md` → `plan.md`
3. `design-to-code:subagent-driven-development` — `plan.md` → implementation, with spec + quality + final review
4. `design-to-code:tdd-verify-from-spec` — drive playwright-cli to verify each acceptance item in `spec.md`

## Plugin-wide discipline (shared HARD-GATE)

- Internal references MUST use `design-to-code:<skill-name>`. References to `superpowers:*` are forbidden.
- Artifact directory is `docs/design-to-code/<YYYY-MM-DD>-<topic>/`. Filenames are fixed: `spec.md`, `plan.md`, `progress.md`, `verify.log.md`.
- Convert this skill's checklist into TodoWrite tasks and execute in order; no skipping.
- `spec.md` is immutable to the assistant; only the user may edit it. `plan.md` is written only by `design-to-code:writing-plans`. `progress.md` is appended only by `design-to-code:subagent-driven-development` (exception: `design-to-code:tdd-verify-from-spec` may append a single "verification complete" section at the very end).
- Do not write artifact files on `main` / `release`. A feature branch or worktree must exist first.

## Activation

When both (a) the user's message contains a design URL or image attachment AND (b) the message expresses implementation intent, immediately dispatch to `design-to-code:brainstorming-from-design`. Do not ask clarifying questions at this layer — that skill handles Q&A.

When only one of the two conditions is present, ask the user whether they want to enter the workflow. Do not auto-dispatch.

## Manual entry from later stages

Users may skip to a later skill when artifacts from earlier stages already exist:

- Have a user-approved `spec.md` → invoke `design-to-code:writing-plans` directly.
- Have a `plan.md` → invoke `design-to-code:subagent-driven-development` directly.
- Implementation already done, want to verify → invoke `design-to-code:tdd-verify-from-spec` directly.

In all cases, the invoked skill assumes its upstream artifacts are present and valid.
