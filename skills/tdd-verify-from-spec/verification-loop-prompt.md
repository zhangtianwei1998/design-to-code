# Verification Loop Prompt (main agent self-reference)

This file is read by the main agent while driving playwright-cli. It is NOT dispatched to a subagent.

## Preconditions
- Dev server running at a known URL.
- `playwright --version` succeeds (install globally if missing: `npm install -g @playwright/cli@latest`).
- Browser opened once per session with:

  ```bash
  playwright open <dev-url> --headed --persistent
  ```

  `--headed --persistent` are REQUIRED — without them the login session cannot be carried across verification steps.

## Per-item loop skeleton
For each acceptance item in `spec.md`, in order:

1. Read the item and identify: the user action to simulate, the element to observe, the assertion to make.
2. Drive the browser:
   - Navigate / click / type / hover as the item specifies.
   - Prefer `getByRole` and `getByText` locators for readability; fall back to CSS selectors only when role/text won't uniquely identify the target.
   - Wait for idle / network / specific element before asserting.
3. Capture evidence:
   - Screenshot of the assertion point.
   - DOM excerpt (the element's HTML) around the assertion.
   - Any relevant network request/response (if the item mentions data).
4. Assert:
   - Strict equality for exact text; contains-match for longer strings; numeric comparisons for counts.
   - Record PASS/FAIL in `verify.log.md` using the template in the skill's Artifacts section.

## On FAIL
1. Write the `❌` entry in `verify.log.md` including evidence (step 3 above).
2. **Classify the failure** into one of:
   - **Category A — code-in-repo**: the defect looks fixable by editing files in the current repo (component, slice, API call, styling, routing, etc.).
   - **Category B — out-of-repo**: the test case itself is ambiguous or wrong; the backend returns data the spec did not anticipate; an environment or dependency issue; a `spec.md` defect that only the user can resolve.
   - When uncertain, treat as Category B (fail-fast).
   Record the category and a one-line reasoning in the `verify.log.md` entry.
3. **If Category A**:
   a. Locate relevant files in the codebase (grep the observed wrong text, trace components from the route).
   b. Dispatch a fixer subagent using `./fixer-prompt.md`, filling:
      - `{{ACCEPTANCE_ITEM}}` — the item verbatim from `spec.md`
      - `{{OBSERVED_EVIDENCE}}` — the evidence you captured
      - `{{LOCATED_FILES}}` — the paths you identified
   c. Wait for the fixer to return. If the dev server uses HMR, wait briefly for reload.
   d. Re-run the SAME acceptance item (do not skip to the next one).
4. **If Category B**: do NOT dispatch a fixer. Optionally re-run once to rule out flake; do not attempt a code change. The threshold for this category is tight (see below).

## Failure thresholds
- **Category A**: **5 consecutive rounds** on the same acceptance item pauses the loop and reports to the user.
- **Category B**: **2 consecutive rounds** on the same acceptance item pauses the loop and reports to the user.
- An item may change category across rounds (e.g. first round looked like Category A, after fixer round two it's clear the test case itself is wrong). Use the threshold matching the current round's category.
- In all cases: do NOT weaken the acceptance item in `spec.md` to make verification pass.

## Never do
- Never modify `spec.md` to make verification pass.
- Never declare PASS without evidence captured.
- Never dispatch a fixer without first writing the `❌` entry with evidence.
