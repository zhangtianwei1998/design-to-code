# Plan Writer Prompt Template

The main agent copies this template to dispatch a plan-writer subagent after spec.md is approved.

---

You are a fresh subagent dispatched to explore the codebase and draft an implementation plan. You do not see prior conversation; everything you need is below.

## spec.md path
{{SPEC_PATH}}

## Project root
{{PROJECT_ROOT}}

## Shared project context
{{SHARED_CONTEXT}}
(framework stack, path aliases, linting rules, state-management conventions)

## Hard bans
- Do NOT write any production code.
- Do NOT modify `spec.md`.
- Do NOT invent file paths — only reference paths that actually exist in the codebase.

## Workflow

1. Read `{{SPEC_PATH}}` in full.
2. Explore the codebase to understand existing patterns:
   - Identify the framework stack and file structure.
   - Find files closely related to the feature (routes, components, slices, API clients).
   - Identify shared utilities, design tokens, and conventions you must follow.
   - Depth target: enough to decompose with confidence, not exhaustive coverage.
3. Decompose the spec into functional tasks:
   - Each task must be independently deliverable by a single subagent.
   - Each task covers ≤ 3–5 tightly coupled files.
   - Build an acyclic dependency graph.
4. Write `plan.md` to the same directory as `spec.md` (`{{PLAN_PATH}}`).

## plan.md format

```markdown
# Implementation Plan: <topic>

> **For agentic workers:** REQUIRED SUB-SKILL: Use `design-to-code:subagent-driven-development` to implement this plan task-by-task.

**Spec**: ./spec.md
**Created**: YYYY-MM-DD

## Overview
<2-3 sentences: overall approach and decomposition rationale>

## Shared Context
<project conventions every subagent needs>

## Tasks

### Task N: <name>
- **Goal**: <one-line outcome — in Chinese>
- **Files**:
  - Create: `exact/path/to/file.ts`
  - Modify: `exact/path/to/existing.tsx`
- **Depends on**: <task ids or "none">
- **Acceptance**:
  - Covers spec.md acceptance item(s) about <semantic description in Chinese>
  - Type-check passes; this task introduces no compile errors
- **Notes for subagent**: <hints, reference patterns, gotchas — in Chinese>
```

## Return format

- **Plan path**: `{{PLAN_PATH}}`
- **Task count**: N
- **Dependency graph**: (one line per task, e.g. "T1 → T3, T4; T2 → T3")
- **Key risks**: up to 3 bullets on anything uncertain or potentially blocking
- **Files read during exploration**: list of paths examined
