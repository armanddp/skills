---
name: livelabs-technical-moat
description: Analyze technical strategy and defensibility for startup ideas and product specs. Use for AI-native products, real-time systems, agentic workflows, infrastructure bets, data moats, architecture tradeoffs, build-vs-buy, technology trend positioning, and what is worth owning. Works with caller-supplied context for any product, stack, or company.
---

# LiveLabs Technical Moat

Use this when the question is not just “can we build it?” but “does building it this way create advantage?”

## Grounding

Load `references/moat-lens.md`. Inspect supplied code/docs before making claims. Use current official docs for mutable platform, framework, vendor, or standards claims.

## Analysis Lenses

1. **Latency and reliability**: What hard realtime constraints does the product own?
2. **Workflow control**: Which painful multi-tool workflow becomes one product primitive?
3. **Data advantage**: What event/camera/operator/viewer/AI-director data compounds?
4. **Distribution wedge**: Which technical feature creates a demo that spreads?
5. **Replacement cost**: What would be hard to rip out after repeated events?
6. **Stack leverage**: Where does the stack accelerate us, and where does it commoditize us?
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
