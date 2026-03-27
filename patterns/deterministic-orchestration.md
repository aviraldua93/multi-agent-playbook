# Pattern: Deterministic Orchestration

> Replace the LLM orchestrator with code.

## The Problem

In Copilot CLI and Claude Code, the orchestrator is an LLM. It accumulates context with every turn. Every sub-agent result flows back into its window. After 10 sub-agents, it's carrying 30K+ tokens of accumulated state. Quality degrades. Decisions become inconsistent.

And the orchestrator is invisible — no dashboard shows its context utilization or health.

## The Pattern

Replace the LLM orchestrator with deterministic code. The orchestrator reads a YAML workflow, routes outputs between agents, and never accumulates context.

```
BEFORE (LLM orchestrator):
  🧠 Main Session ← accumulates everything
    → CEO → output flows back to 🧠
    → VP  → output flows back to 🧠  
    → IC  → output flows back to 🧠
    Context: 80K+ tokens, degrading

AFTER (code orchestrator):
  ⚙️ YAML/Python ← stateless, routes only
    → CEO → output saved to workflow state
    → VP  → reads CEO output from state
    → IC  → reads VP output from state
    Context: N/A — it's code, not an LLM
```

## Implementation: Conductor

[Microsoft Conductor](https://github.com/microsoft/conductor) implements this pattern:

```yaml
workflow:
  name: agent-dashboard
  entry_point: ceo

agents:
  - name: ceo
    input: [workflow.input.mission]
    routes:
      - to: vp_team  # routed by YAML, not LLM decision

  - name: vp_engineering
    input: [ceo.output]  # context passed explicitly
```

- YAML defines the full topology upfront
- Each agent is a separate LLM API call with its own fresh context
- The Python runtime routes outputs — no LLM involved in routing
- Web dashboard shows every agent on a DAG — full visibility

## The Tradeoff

| | LLM Orchestrator | Code Orchestrator |
|---|---|---|
| **Deterministic?** | No — might route differently each time | Yes — same YAML = same flow |
| **Can adapt?** | Yes — can decide dynamically | No — topology is fixed |
| **Context limit?** | Yes — degrades over time | No — it's not an LLM |
| **Visible?** | No — invisible brain | Yes — DAG shows everything |

## When to Use

- Workflows with known topology (you know who runs when)
- Long workflows (>5 agents) where context accumulation matters
- When you need reproducibility (same YAML = same result)
- When you need observability (DAG visualization)

## When NOT to Use

- When the workflow needs to adapt at runtime (use LLM orchestrator)
- Simple 2-3 agent tasks (overhead not worth it)
- When agents need to spawn dynamically (Conductor doesn't support this yet — see [Issue #66](https://github.com/microsoft/conductor/issues/66))

## Fixes

- Orchestrator context overload
- Invisible orchestrator (no observability)
- Non-deterministic routing
- Quality degradation in long workflows
