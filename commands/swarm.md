---
description: Spawn a swarm of user persona agents to simulate real purchase decisions and predict willingness to pay. Each persona independently simulates their decision journey. Returns a WTP prediction report with validated/invalidated findings.
---

You are the Swarm Orchestrator. Your job is to simulate real user behavior across 6-8 distinct personas and predict whether they will pay for the product or idea described below.

The product/idea to validate is:
$ARGUMENTS

---

## Step 0: Context Check

If the input above is missing any of these, stop and ask before proceeding:
- What the product does (one clear sentence)
- Who it's for (target market/audience)
- The pricing structure (free tier, paid tiers, price points)
- The core value claim (what problem it solves)

Do not proceed without enough context to define realistic personas.

---

## Step 1: Define Personas

Before spawning agents, define 6-8 user personas that represent the realistic range of people who would encounter this product. Base them on the target market described in the input.

For each persona define:
- Name, age, job title / archetype
- Monthly tool budget (personal or company card)
- Current tool stack (what they already use for this problem)
- Primary goal when using a tool like this
- Core pain point the product might solve
- Budget mindset (how they make purchase decisions)

Pick personas that represent genuine diversity in:
- Budget (personal vs. company card, tight vs. loose)
- Sophistication (power user vs. casual vs. professional)
- Alternatives (someone with a full paid stack vs. someone who uses only free tools)
- Use case (different jobs to be done within the same product)

---

## Step 2: Spawn All Persona Agents in Parallel

Spawn all 6-8 persona subagents **simultaneously in a single message** using multiple Agent tool calls.

For each persona, the subagent prompt must include:

1. The persona brief (name, role, budget, stack, goals, pain, mindset)
2. The full product description and pricing from the user's input
3. This exact decision journey to simulate:

```
SIMULATE YOUR DECISION JOURNEY — be honest and self-interested, not helpful:

1. You land on the product homepage. What is your first reaction?
2. You try the free tier. What do you think of the output vs. your current tools?
3. You hit the free tier limit or reach the pricing page. What happens in your head?
4. Do you pay? At what price would you definitely pay without overthinking it?

Return as [persona name], first person, honest internal monologue.

End with:
- VERDICT: [PAY (tier) | FREE ONLY | NOT INTERESTED | NEEDS DIFFERENT TIER]
- PRICE I'D ACTUALLY PAY: $X/month
- UPGRADE TRIGGER: [the specific moment or feature that closes the decision]
- KILL FACTOR: [the specific thing that makes you leave or stay free forever]
```

---

## Step 3: Synthesize Results

Once all persona agents return, produce the Swarm Report:

---

# Swarm Report: [Product Name]
**Personas simulated:** [N]
**Date:** [today's date]

## Verdict Table

| Persona | Verdict | Price They'd Actually Pay | Upgrade Trigger | Kill Factor |
|---|---|---|---|---|
[one row per persona]

## Payment Rate
[X out of N personas would pay in some form — state whether this is a real product or a lead magnet]

## Patterns the Swarm Surfaced

List 3-5 patterns that emerged independently across multiple personas. Only include patterns confirmed by 2+ personas. Format each as:

**[Pattern name]:** [What the personas said + what it means for the product]

## Missing Tier / Pricing Gap
[If multiple personas named a price point or tier that doesn't exist in the current pricing, call it out explicitly. This is often the highest-value finding.]

## What the Swarm Missed (Orchestrator's Assessment)
[Anything the personas couldn't know — market dynamics, competitive context, behavioral patterns that require external data — flag as "needs research validation"]

## Verdict: Paid Product or Lead Magnet?

State clearly:
- Which segments will pay, at what price, and what tier they need
- Which segments are lead magnets (spread awareness but don't convert at current pricing)
- The one pricing or product change with the highest impact on payment rate
- The single most important next test to run with real users

---

## Notes

- Use for: pricing validation, new product ideas, offer design, feature prioritization
- Pair with `/council` for strategic decisions after the swarm identifies the right framing
- For real-world anchoring, follow up with the `research-agent` to validate swarm findings against Reddit/forum data
- Richer input = sharper personas = more accurate simulation. Include as much product context as possible.
