# design-to-code

A Claude Code plugin that turns any visual source (design URL, attached image, or an already-rendered page) plus explicit implementation intent into a verified, pixel-faithful implementation — through a 5-stage, linear, spec-driven workflow.

---

## Workflow

```mermaid
flowchart TD
    USER(["User: visual source\n+ implementation intent"])

    subgraph S1["① brainstorming-from-design"]
        direction LR
        a1["Ingest visual source\n(playwright / multimodal)"] --> a2["Interview user"] --> a3["Write spec.md\n+ user approval"]
    end

    subgraph S2["② writing-plans"]
        direction LR
        b1["Read spec.md"] --> b2["Explore codebase"] --> b3["Write plan.md\n+ user approval"]
    end

    subgraph S3["③ subagent-driven-development"]
        direction LR
        c1["Dispatch implementer\nsubagent per task"] --> c2["Spec review"] --> c3["Code quality review"]
        c3 -. "fail → retry" .-> c1
    end

    subgraph S4["④ tdd-verify-from-spec"]
        direction LR
        d1["Drive playwright\nagainst running app"] --> d2{"Pass?"}
        d2 -- "fail" --> d3["Dispatch fixer\nsubagent"] --> d1
        d2 -- "pass" --> d4["✅ Record in\nverify.log.md"]
    end

    subgraph S5["⑤ visual-qa-from-design"]
        direction LR
        e1["Load design reference\n(multimodal / playwright)"] --> e2["Screenshot each view"] --> e3{"Visual match?"}
        e3 -- "discrepancy" --> e4["Dispatch visual-fixer\nsubagent"] --> e2
        e3 -- "match" --> e5["✅ Record in\nvisual-qa.md"]
    end

    DONE(["Implementation verified\nfunctionally + visually ✅"])

    USER --> S1
    S1 -->|"spec.md"| S2
    S2 -->|"plan.md"| S3
    S3 -->|"code + progress.md"| S4
    S4 -->|"verify.log.md"| S5
    S5 --> DONE
```

### How each stage works

| # | Skill | What it does | Consumes | Produces |
|---|-------|-------------|----------|----------|
| 1 | `brainstorming-from-design` | Opens design in playwright or reads images multimodally; interviews you question-by-question; writes a user-approved spec | URL / image / in-project page | `spec.md` |
| 2 | `writing-plans` | Explores the codebase, decomposes the spec into subagent-sized tasks with explicit file lists and acceptance criteria | `spec.md` | `plan.md` |
| 3 | `subagent-driven-development` | Dispatches a fresh implementer subagent per task; each task passes spec-review then code-quality-review before moving on | `plan.md` | code + `progress.md` |
| 4 | `tdd-verify-from-spec` | Drives playwright against the running app; checks every acceptance item from spec.md; dispatches fixer subagents on failures | `spec.md` + running app | `verify.log.md` |
| 5 | `visual-qa-from-design` | Screenshots each view, compares multimodally against the original design source, dispatches visual-fixer subagents for pixel-level gaps | `spec.md` design source + running app | `visual-qa.md` |

### Artifacts

All files land under `docs/design-to-code/<YYYY-MM-DD>-<topic>/` in your project:

```
docs/design-to-code/2025-06-01-cart-empty-state/
├── spec.md          ← stage 1 (user-approved; never edited by assistant afterward)
├── plan.md          ← stage 2
├── progress.md      ← stage 3 (one entry per task)
├── verify.log.md    ← stage 4 (one entry per acceptance item)
├── visual-qa.md     ← stage 5 (one entry per view)
└── screenshots/     ← stage 5 screenshots
```

No skill writes into another skill's artifact.

---

## Trigger

The plugin activates when a single message contains **both**:

1. A visual source — a design URL, an image attachment, or a reference to an existing page/component in the current project, **and**
2. Explicit implementation intent — "实现", "做出来", "还原", "照这个写", "implement this", "build this", "extend this page".

Only one of the two conditions? Claude will ask before entering the workflow.

---

## Skipping to a later stage

If upstream artifacts already exist, jump in directly:

```
Have spec.md already?     → invoke design-to-code:writing-plans
Have plan.md already?     → invoke design-to-code:subagent-driven-development
Re-verify existing build? → invoke design-to-code:tdd-verify-from-spec
Re-run visual QA only?    → invoke design-to-code:visual-qa-from-design
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Playwright install fails | Run `npm install -g @playwright/cli@latest` manually; ensure node ≥ 18 |
| Login session lost between verifications | Confirm the browser was opened with `--headed --persistent`; non-persistent sessions lose cookies |
| Fixer loops without converging (stage 4) | Skill pauses at 5 rounds and reports; treat this as a signal to redesign the acceptance item or split it |
| Visual-fixer loops without converging (stage 5) | Skill pauses at 3 rounds and reports; check whether the design reference is clear enough to derive an exact value |

---

## Relationship to superpowers

Inspired by superpowers conventions (subagent-driven-development, test-driven-development, writing-plans) but **self-contained** at runtime. No skill references `superpowers:*` internally.
