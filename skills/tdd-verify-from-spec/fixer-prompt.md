# Fixer Prompt Template

The main agent copies this template to dispatch a fixer subagent when an acceptance item fails verification and is classified as Category A (code-in-repo).

---

You are a fresh subagent dispatched to fix a specific UI verification failure. You do not see prior conversation; everything you need is below.

## Failing acceptance item (verbatim from spec.md — DO NOT modify spec.md)
{{ACCEPTANCE_ITEM}}

## Observed evidence from verification
{{OBSERVED_EVIDENCE}}

## Files the main agent located as relevant
{{LOCATED_FILES}}

## Hard bans
- Do NOT modify `spec.md` — ever, for any reason.
- Do NOT modify files outside `{{LOCATED_FILES}}` unless you discover the true fix requires a closely-adjacent file; in that case include it in your return and explain why.
- Do NOT propose refactors, renames, or cleanups unrelated to this failure.
- Do NOT declare "can't reproduce" without first reading the implementation of each located file.
- If after reading the located files you believe the failure is NOT actually a code-in-repo issue (e.g. the acceptance item is ambiguous or contradicts other items, the backend returns data the spec didn't anticipate, an environment issue), do NOT force a code change. Return with `Reclassify: Category B` and your reasoning. The main agent will take it from there.

## Workflow
1. Read every located file fully.
2. Identify the defect causing the observed evidence.
3. Make the minimum change to fix it.
4. Run the project's type check / lint if available.
5. Self-review your diff against the acceptance item.

## Return format (on successful fix)
- **Root cause:** one or two sentences explaining why the acceptance item was failing.
- **Files changed:** list of paths (including any closely-adjacent file you had to touch, with justification).
- **Diff summary:** 2-4 lines.
- **Reproduction hint for the main agent:** "To re-verify: <specific action the main agent should take in playwright to confirm the fix>."

## Return format (on reclassification to Category B)
- **Reclassify: Category B**
- **Reasoning:** why this is not a code-in-repo issue (cite specific findings from reading the located files).
- **What the user or upstream system must resolve:** concrete next step (e.g. "acceptance item 3 contradicts item 5", "backend endpoint `/foo` returns `status: null` which the spec doesn't cover").
- **Files read:** list of paths you examined before reaching this conclusion.
