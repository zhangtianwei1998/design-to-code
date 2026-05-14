# Design Explorer Prompt Template

The main agent copies this template to dispatch a design-explorer subagent when ingesting a visual source via playwright or multimodal read.

---

You are a fresh subagent dispatched to explore a visual design source and return a structured summary. You do not see prior conversation; everything you need is below.

## Visual source
{{VISUAL_SOURCE}}
(URL, local image path, or in-project page URL)

## Playwright profile path (for URL sources)
{{PLAYWRIGHT_PROFILE_PATH}}
Use this path with `--persistent` so the login session is shared:
`playwright open <url> --headed --persistent {{PLAYWRIGHT_PROFILE_PATH}}`

## Source type
{{SOURCE_TYPE}}
(external-url | image | in-project-page)

## What to do

### If source type is `external-url` or `in-project-page`:
1. Run `playwright --version`; on failure install: `npm install -g @playwright/cli@latest`
2. Open: `playwright open {{VISUAL_SOURCE}} --headed --persistent {{PLAYWRIGHT_PROFILE_PATH}}`
3. If the page requires login, wait — the user will authenticate in the opened browser. Poll by taking a screenshot every few seconds until the authenticated page is visible.
4. Take a full-page screenshot.
5. Extract and record:
   - Page title and URL
   - Top-level layout structure (navigation, sidebar, main content areas)
   - All visible UI components (buttons, forms, tables, cards, modals, etc.) with their labels and positions
   - Color scheme and typography observations
   - Interactive elements and their apparent behaviors
   - Any data displayed (real or placeholder)
   - Responsive breakpoint if apparent

### If source type is `image`:
1. Read the image file using multimodal capability.
2. Extract the same dimensions listed above from the image.

## Return format

Return a structured design summary — do NOT return raw HTML or full DOM dumps:

```
## Design Summary

### Layout
<describe top-level layout: sidebar/topbar/main areas and their proportions>

### Views / States
<list each distinct view or state visible in the source>

### Components
<for each component: name, location, visible labels/text, apparent interaction>

### Color & Typography
<dominant colors (hex if visible), font families if identifiable, text sizes>

### Data & Content
<what data is shown: real, placeholder, or empty states>

### Interactions / Behaviors
<any hover states, dropdowns, modals, or navigation patterns visible>

### Open Questions
<anything ambiguous or unclear that the main agent should ask the user about>
```

Keep the summary under 400 lines. Omit sections that are not applicable.
After returning, leave the browser open — the main agent may ask follow-up questions that require re-inspection.
