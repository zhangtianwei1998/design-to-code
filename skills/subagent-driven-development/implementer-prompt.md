# Implementer Prompt Template

The main agent copies this template and fills placeholders with values from `plan.md`.

---

You are a fresh subagent dispatched to implement ONE task from an implementation plan. You do not see prior conversation; everything you need is below.

## Task
{{TASK_NAME}}

## Goal
{{GOAL}}

## Files in scope (you may only edit these)
{{FILES_IN_SCOPE}}

## Shared project context
{{SHARED_CONTEXT}}

## Acceptance (all must be met before you return)
{{ACCEPTANCE_ITEMS}}

## Notes
{{NOTES}}

## Hard bans
- Do NOT edit files outside the scope above.
- Do NOT propose refactors, renames, or "while I'm here" cleanups.
- Do NOT skip any acceptance item.
- Do NOT modify `spec.md`, `plan.md`, `progress.md`, `verify.log.md`, or `visual-qa.md`.

## Workflow
1. Read each file in scope before editing.
2. Implement the minimum change to satisfy all acceptance items.
3. Run the project's type check / lint where applicable.
4. Self-review: reread your diff, confirm each acceptance item is met.

## Return format
Return a structured summary:

- **Files changed:** `<list of paths>`
- **Diff summary:** `<2-4 lines describing what changed>`
- **Acceptance results:** for each item, write `✅ <item>` with a one-line justification, or `❌ <item>` with why it failed.
- **Open questions for the main agent:** `<list or "none">`
