---
name: brainstorming-from-design
description: Use when the user provides a design URL or image AND expresses intent to implement it. Creates spec.md via playwright/image inspection, grill-me style Q&A, and user approval. MUST be followed by design-to-code:writing-plans.
---

# Brainstorming from Design

Transform a design URL or image into a user-approved `spec.md`, then hand off to `design-to-code:writing-plans`. This skill owns the "design → spec" stage only; no code is written here.

## Plugin-wide discipline (shared HARD-GATE)

- Internal references MUST use `design-to-code:<skill-name>`. References to `superpowers:*` are forbidden.
- Artifact directory is `docs/design-to-code/<YYYY-MM-DD>-<topic>/`. Filenames are fixed: `spec.md`, `plan.md`, `progress.md`, `verify.log.md`.
- Convert this skill's checklist into TodoWrite tasks and execute in order; no skipping.
- `spec.md` is immutable to the assistant; only the user may edit it. `plan.md` is written only by `design-to-code:writing-plans`. `progress.md` is appended only by `design-to-code:subagent-driven-development` (exception: `design-to-code:tdd-verify-from-spec` may append a single "verification complete" section at the very end).
- Do not write artifact files on `main` / `release`. A feature branch or worktree must exist first.

## Hard gates

- No production code changes in this skill.
- No checklist skipping.
- Each checklist item becomes a TodoWrite task, executed in order.
- Internal references only `design-to-code:*`.

## Process flow

The 8-step checklist below walks from project exploration → visual ingestion → grill-me Q&A → feature confirmation → spec writing → self-review → user review → hand-off. Do not combine, reorder, or skip steps.

## Checklist

1. **Explore project context** — read `CLAUDE.md`, recent commits, related feature directories; identify framework stack.

2. **Ingest visual source**
   - If a **design URL**:
     - Run `playwright --version`; on failure, `npm install -g @playwright/cli@latest`.
     - Run `playwright open <url> --headed --persistent`. Headed + persistent are REQUIRED; without them login cannot be carried.
     - Prompt the user to finish any login flow in the opened browser.
     - Read the design content via screenshots and DOM inspection.
   - If an **image attachment**: read the image directly using the multimodal capability. Do NOT launch playwright.

3. **Grill-me Q&A** — drive the decision tree to shared understanding.
   - **Methodology (from mattpocock `grill-me`):**
     - One question at a time.
     - Walk the decision tree systematically; later branches may change based on earlier answers.
     - Every question carries a recommended answer with rationale.
     - Prefer grepping / reading the project over asking the user; only ask for true intent or business judgment.
     - Prefer multiple choice over open-ended.
     - Do not stop early to save tokens; stop only when understanding is genuinely shared.
   - **Branches that MUST be covered (in order):**
     1. Interface layer — new backend endpoints? Existing ones gaining fields or being restructured? Contract source (OpenAPI / already implemented / not yet written)? Backward-compat requirement?
     2. Entry point & location — which page, route, component hosts the feature? Trigger (button, menu, right-click, URL param, hotkey)? Permission / gating / role constraints?
     3. Interaction details — loading / empty / error states? Confirm, undo, toast, redirect, inline update?
     4. Data & state — new state location (Redux slice / local / URL query)? Persistence (redux-persist whitelist)? Concurrency / optimistic update?
     5. i18n / styling — does copy need zh-CN and en-US? antd theme or `antd-style` custom?
     6. Non-goals — explicitly confirm what is NOT being built.
   - After each branch, emit a short summary of that branch and let the user confirm before moving on.

4. **Feature-point confirmation** — summarize the understood feature points as bullets; the user confirms or corrects each.

5. **Write `spec.md`** to `docs/design-to-code/<YYYY-MM-DD>-<topic>/spec.md`. Sections:
   - Design source (URL or image path)
   - Feature points (with entry, interaction, data)
   - Interface impact (added / modified / none)
   - Acceptance checklist — phrased for playwright verification; each item must have a concrete assertion
   - Non-goals

6. **Spec self-review** — inline scan for placeholders, contradictions, scope drift, ambiguity; fix inline.

7. **User review** — `Read` the `spec.md` into the conversation and ask the user to approve. If changes are requested, return to step 5.

8. **Hand off** — invoke `design-to-code:writing-plans`, passing the `spec.md` path.

## Artifacts

- `spec.md` (committed to git by the user's project)
