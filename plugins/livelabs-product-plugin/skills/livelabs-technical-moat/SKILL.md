---
name: livelabs-technical-moat
description: Analyze LiveLabs technical strategy and defensibility. Use for StreamBridge, Chumo, LiveKit, SRT/WebRTC, AI Producer, spatial video, real-time systems, streaming infrastructure, agentic event pages, data moats, architecture tradeoffs, build-vs-buy, and technology trend positioning.
---

# LiveLabs Technical Moat

Use this when the question is not just “can we build it?” but “does building it this way create advantage?”

## Grounding

Load `references/moat-lens.md` and `references/streambridge-context.md`. For StreamBridge, inspect code/docs before making claims:
- `ARCHITECTURE.md`
- `docs/SRT_ROUTING_ARCHITECTURE.md`
- `docs/features/agentic-event-pages/AGENTIC_EVENT_PAGES.md`
- `services/director/README.md`
- `docs/skills/LIVEKIT.md`

Use current official docs for mutable platform claims, especially LiveKit.

## Analysis Lenses

1. **Latency and reliability**: What hard realtime constraints does the product own?
2. **Workflow control**: Which painful multi-tool workflow becomes one LiveLabs primitive?
3. **Data advantage**: What event/camera/operator/viewer/AI-director data compounds?
4. **Distribution wedge**: Which technical feature creates a demo that spreads?
5. **Replacement cost**: What would be hard to rip out after repeated events?
6. **Stack leverage**: Where does LiveKit/SRT/Rails/Director accelerate us, and where does it commoditize us?
7. **Trend fit**: Smart glasses, multimodal AI, citizen video, spatial interfaces, and trust/authenticity.

## Output

```markdown
## Technical Bet
[one sentence]

## Moat Thesis
[why this compounds if we execute]

## Existing Advantage
- [asset from repo/product/team]

## Commoditized Parts
- [parts we should not overvalue]

## Hard Part Worth Owning
- [specific system/workflow/data loop]

## Build Next
[technical/product move that increases defensibility]

## Risks
- [risk] -> [test]
```
