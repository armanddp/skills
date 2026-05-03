---
name: livelabs-product-research
description: Run lean but deep product research for startup ideas and product bets. Use when researching ICPs, communities, Reddit, X, Hacker News, YouTube, competitors, customer pain, market wedges, workflows, or message language for product decisions. Works with caller-supplied context for any product, market, or company.
---

# LiveLabs Product Research

Use this to turn messy market curiosity into a narrow ICP, a painful workflow, and language that sounds like the customer.

## Grounding

Load `references/research-playbook.md`. Then gather current evidence from web/community sources and from the repo/docs when the user provides them.

Prefer primary/current sources:
- community threads: Reddit, X, HN, Discord/forum posts, YouTube comments when accessible
- product/docs: competitor docs, changelogs, support pages, pricing, job posts
- supplied product context: specs, docs, code, pitch docs, sales notes, support tickets, analytics, interview notes

## Research Passes

1. **Problem map**: What jobs are people trying to do in the target market/workflow?
2. **Pain language**: Capture exact phrases. Preserve wording. Do not translate too early.
3. **ICP candidates**: Rank by urgency, reachable community, budget, existing workaround, and unfair advantage.
4. **Alternatives**: Map actual alternatives: tools, agencies, spreadsheets, internal teams, manual workarounds, incumbents, and doing nothing.
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
- Why us:
- Demo promise:

## Risks
- [risk] -> [validation]
```

Do not end with a generic market summary. End with a product decision or research gap.
