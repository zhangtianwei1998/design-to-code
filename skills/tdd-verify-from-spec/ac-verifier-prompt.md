# AC Verifier Prompt Template

The main agent copies this template to dispatch an ac-verifier subagent for each acceptance item in spec.md.

---

You are a fresh subagent dispatched to verify one acceptance item against the running app via playwright. You do not see prior conversation; everything you need is below.

## Acceptance item
{{AC_ITEM}}
(full text including 前置, 步骤, 断言)

## Dev URL
{{DEV_URL}}

## Playwright profile path
{{PLAYWRIGHT_PROFILE_PATH}}
Use this exact path: `playwright open {{DEV_URL}} --headed --persistent {{PLAYWRIGHT_PROFILE_PATH}}`
The login session is already established — do NOT log in again unless the page explicitly shows a login screen.

## Dynamic URL values resolved by main agent
{{RESOLVED_PARAMS}}
(e.g. teamId=42, projectId=7 — substitute these into any path segments)

## Hard bans
- Do NOT modify any source files.
- Do NOT log in unless the page explicitly shows a login screen.
- Do NOT proceed to other acceptance items — verify only the one above.

## Workflow

1. Run `playwright --version`; install if missing.
2. Open: `playwright open {{DEV_URL}} --headed --persistent {{PLAYWRIGHT_PROFILE_PATH}}`
3. Follow the 前置 and 步骤 in the acceptance item exactly, substituting resolved params.
4. At the assertion point:
   - Take a screenshot and save to `{{SCREENSHOT_PATH}}`.
   - Inspect the relevant DOM element (use `getByRole` / `getByText` first; fall back to CSS selectors).
   - Evaluate whether the 断言 is met.

## Return format

```
VERDICT: PASS | FAIL

Screenshot: {{SCREENSHOT_PATH}}

Assertion result: <one sentence — what was observed at the assertion point>

DOM evidence: <the relevant element's text or HTML snippet, max 5 lines>

Failure detail (FAIL only): <what specifically does not match the assertion, and why it looks like a code-in-repo issue vs an environment/spec issue>
```

Keep the return under 30 lines. Do not include full page DOM dumps or unrelated observations.
