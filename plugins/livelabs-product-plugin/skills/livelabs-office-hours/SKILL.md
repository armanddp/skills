---
name: livelabs-office-hours
description: Challenge LiveLabs product ideas before building. Use for StreamBridge, Chumo, realtime video, LiveKit, AI producer, event-network, or GTM ideas when the user asks for office hours, idea critique, founder review, narrow wedge, demand reality, or whether something is worth building.
---

# LiveLabs Office Hours

Use this as the fast adversarial product review for LiveLabs. The job is not to produce a polished doc first. The job is to find the sharpest true wedge.

## Grounding

Before challenging the idea, load only what is needed:
- StreamBridge context: `references/streambridge-context.md`
- Current repo docs if the idea touches product reality: `/Users/m1/source/livelabs/streambridge-app/docs/features/v1/V1_PRODUCT_PRD.md`, `docs/strategy/positioning-statement.md`, `ARCHITECTURE.md`
- SPC/network context if the idea touches vision/fundraising: Notion page `2f8faf58ca458146b6f5e6edcfb86128`

## Workflow

Run a tight office-hours loop. Ask one question at a time when live with the user. If writing from existing context, answer each question from evidence and mark weak assumptions.

1. **Demand reality**: Who urgently needs this now? What did they try yesterday?
2. **Status quo workaround**: What manual workflow, spreadsheet, OBS/vMix setup, WhatsApp group, or vendor stack do they tolerate today?
3. **Desperate specificity**: Name the exact buyer/user, event type, clock pressure, and budget owner.
4. **Narrowest paid wedge**: What can be shipped in days that someone would pay for or pilot this month?
5. **Live evidence**: What have we observed in pilots, support, Discord, repo telemetry, or customer conversations?
6. **Future fit**: If this works, how does it compound toward human cameras + AI curation + spatial broadcast network?

## Output

Keep it short:

```markdown
## Verdict
[Build / test / kill / defer] because [one sentence].

## Sharpest Wedge
- User:
- Moment:
- Current workaround:
- Paid/pilotable offer:

## What Must Be True
- [assumption] -> [fastest validation]

## Product Move
[the smallest thing to build or test next]

## Pushback
[the uncomfortable truth]
```

If the idea is too broad, force it down to one event type, one operator workflow, and one measurable outcome.
