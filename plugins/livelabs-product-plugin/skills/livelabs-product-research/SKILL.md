---
name: livelabs-product-research
description: Run lean but deep product research for LiveLabs. Use when researching StreamBridge, Chumo, realtime video, LiveKit, broadcast workflows, ICPs, communities, Reddit, X, Hacker News, YouTube, competitors, customer pain, market wedges, or message language for product decisions.
---

# LiveLabs Product Research

Use this to turn messy market curiosity into a narrow ICP, a painful workflow, and language that sounds like the customer.

## Grounding

Load `references/research-playbook.md` and `references/streambridge-context.md`. Then gather current evidence from web/community sources and from the repo/docs when relevant.

Prefer primary/current sources:
- community threads: Reddit, X, HN, Discord/forum posts, YouTube comments when accessible
- product/docs: competitor docs, changelogs, support pages, pricing, job posts
- StreamBridge repo: `/Users/m1/source/livelabs/streambridge-app/docs/features/v1/`, `docs/strategy/`, `frontend/web/src/data/faq.json`

## Research Passes

1. **Problem map**: What jobs are people trying to do around live video, multi-camera, field production, remote cameras, event pages, AI production, or institutional streaming?
2. **Pain language**: Capture exact phrases. Preserve wording. Do not translate too early.
3. **ICP candidates**: Rank by urgency, reachable community, budget, existing workaround, and LiveLabs unfair advantage.
4. **Alternatives**: Map actual alternatives: OBS/vMix + SRT, LiveU, Restream, YouTube, volunteers, WhatsApp, manual AV teams, institute LMS/webinar stacks.
5. **Wedge**: Pick the smallest feature/workflow that speaks directly to one ICP.
6. **Message extraction**: Convert pain into landing-page, outbound, demo, and product copy.

## Output

```markdown
## Research Verdict
[one sentence]

## ICP
- Person:
- Event/workflow:
- Trigger:
- Budget owner:
- Where they gather:

## Pain Evidence
- "[exact phrase]" — [source]
- "[exact phrase]" — [source]

## Current Workaround
[what they do today and why it is painful]

## Feature Wedge
[narrow feature/workflow to build or test]

## Message
- Problem line:
- Why now:
- Why StreamBridge/LiveLabs:
- Demo promise:

## Risks
- [risk] -> [validation]
```

Do not end with a generic market summary. End with a product decision or research gap.
