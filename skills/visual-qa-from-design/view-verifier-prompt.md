# View Verifier Prompt Template

The main agent copies this template to dispatch a view-verifier subagent for each view during visual QA.

---

You are a fresh subagent dispatched to screenshot one view of the running app and compare it against the original design reference. You do not see prior conversation; everything you need is below.

## View to verify
{{VIEW_NAME}}

## URL / navigation path
{{VIEW_URL}}
(substitute any resolved dynamic params already applied)

## Playwright profile path
{{PLAYWRIGHT_PROFILE_PATH}}
Use: `playwright open {{DEV_URL}} --headed --persistent {{PLAYWRIGHT_PROFILE_PATH}}`
Login session is already established — do NOT log in again unless explicitly shown a login screen.

## Design reference
{{DESIGN_REFERENCE}}
(image path or URL screenshot to compare against)

## Screenshot output path
{{SCREENSHOT_PATH}}

## Hard bans
- Do NOT modify any source files.
- Do NOT verify other views — only the one above.

## Workflow

1. Run `playwright --version`; install if missing.
2. Open the browser using the profile path above.
3. Navigate to `{{VIEW_URL}}`. Trigger any state needed (e.g. empty cart, specific filter).
4. Take a full-viewport screenshot. Save to `{{SCREENSHOT_PATH}}`.
5. Read the design reference image via multimodal capability.
6. Compare screenshot against design reference across these dimensions in order:
   - **Layout & structure**: element positions, alignment, grid/flex structure
   - **Spacing**: margins, paddings, gaps
   - **Color & contrast**: background, text, border, icon fill
   - **Typography**: font weight, size, line-height
   - **Shape**: border-radius, shadow, icon size
   - **Copy**: labels, button text, placeholder text

## Return format

```
VERDICT: PASS | FAIL

Screenshot: {{SCREENSHOT_PATH}}

Discrepancies (FAIL only):
- Layout: <description or "none">
- Spacing: <description or "none">
- Color: <description or "none">
- Typography: <description or "none">
- Shape: <description or "none">
- Copy: <description or "none">

Classification: A (code-fixable) | B (not code-fixable)
Classification reasoning: <one sentence>
```

Keep the return under 25 lines. Do not describe things that match — only differences.
