# Pattern: Orchestrator Self-Governance

> The orchestrator is the real mega-agent. Govern it or it governs you.

## The Problem

Every pattern in this playbook governs sub-agents — but the orchestrator (main LLM session) has zero governance. It defaults to:

- Reading full sub-agent output into its own context
- Holding all accumulated results in memory
- Making every routing decision itself
- Growing its context window until quality degrades

The orchestrator IS the mega-agent anti-pattern, hiding in plain sight.

## The Pattern

The orchestrator follows three rules:

### Rule 1: Never Absorb — Route Through Files

```
❌ WRONG:
  Orchestrator deploys Agent A
  Agent A returns 5K tokens of findings
  Orchestrator reads ALL of it into context
  Orchestrator deploys Agent B with "here's what A found: [5K tokens]"

✅ RIGHT:
  Orchestrator deploys Agent A with: "write findings to docs/wave1-findings.md"
  Agent A writes → docs/wave1-findings.md
  Orchestrator deploys Agent B with: "read docs/wave1-findings.md"
  Orchestrator's context: just the file path (~20 tokens)
```

**The orchestrator should know WHAT was produced, not the CONTENT.**
Read summaries (1-2 sentences). Never read full outputs.

### Rule 2: Declare the Bus Upfront

Before deploying ANY wave, the orchestrator writes a manifest:

```markdown
# docs/wave1-manifest.md

## Agents
- Agent A → writes: docs/wave1-research.md
- Agent B → writes: docs/wave1-audit.md
- Agent C → writes: docs/wave1-gaps.md

## Wave 2 reads from:
- docs/wave1-research.md (for Agent D)
- docs/wave1-audit.md (for Agent E)
```

Every agent knows where to write. Every downstream agent knows where to read. The orchestrator just connects the dots.

### Rule 3: Budget Your Own Context

The orchestrator tracks its own context usage:

```
Turn 1:  System prompt + user request           ~8K tokens
Turn 2:  Deploy Wave 1 (4 agents)               +2K tokens
Turn 3:  Read summaries (NOT full output)        +500 tokens
Turn 4:  Deploy Wave 2 (3 agents)               +1.5K tokens
Turn 5:  Read summaries                          +400 tokens
...
BUDGET: Stay under 50K accumulated. If approaching, checkpoint and summarize.
```

## Enforcement Checklist

Before every deployment, the orchestrator asks:

- [ ] Did I tell the agent WHERE to write its output?
- [ ] Am I reading a summary or the full output? (If full → stop)
- [ ] Does the next agent read from a file or from my context? (If context → rewrite)
- [ ] How many tokens have I accumulated? (If >50K → checkpoint)

## When to Use

Always. This isn't optional — it's the cost of being an LLM orchestrator.

## Fixes

- Orchestrator context rot (the #1 failure mode in long sessions)
- Invisible accumulation (orchestrator silently degrades)
- Raw output forwarding (agents get noisy context)
