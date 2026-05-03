---
name: livelabs-product-shape
description: Shape LiveLabs product direction into lean specs, MVP docs, prototype briefs, and buildable product decisions. Use for StreamBridge, Chumo, Director, AI Producer, LiveKit/video features, event pages, producer surfaces, roadmap choices, or turning research/office-hours output into a focused product plan.
---

# LiveLabs Product Shape

Use this after an idea has survived office hours or product research. Keep the output lean enough to build from immediately.

## Grounding

Load `references/shape-template.md` and `references/streambridge-context.md`. For StreamBridge, read the relevant current docs before inventing product behavior:
- `/Users/m1/source/livelabs/streambridge-app/docs/features/v1/V1_PRODUCT_PRD.md`
- `/Users/m1/source/livelabs/streambridge-app/docs/features/v1/V1_CAPABILITY_MATRIX.md`
- `/Users/m1/source/livelabs/streambridge-app/docs/features/v1/PRODUCER_SURFACE_SPEC.md`
- `/Users/m1/source/livelabs/streambridge-app/ARCHITECTURE.md`

## Principles

- Move from vision to one live workflow.
- Prefer real cameras, real pages, real controls over mock surfaces.
- Treat Director as orchestration, not chatbot garnish.
- Specs should be concise, opinionated, and implementation-aware.
- If a requirement does not help “describe the event, invite the cameras, go live,” challenge it.

## Workflow

1. **Frame the bet**: What user outcome and business outcome are we trying to create?
2. **Choose the wedge**: One ICP, one event type, one workflow, one live moment.
3. **Map the journey**: From first prompt to live output.
4. **Bind to primitives**: Which existing StreamBridge objects, APIs, Director tools, LiveKit rooms, event-page primitives, or producer controls are used?
5. **Cut scope**: List what is explicitly not v1.
6. **Define proof**: What would make this obviously working in a demo/pilot?

## Output

Use the template in `references/shape-template.md`. Keep the main spec under two pages unless the user asks for more.
