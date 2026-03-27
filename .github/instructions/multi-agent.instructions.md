---
description: "Multi-agent orchestration rules. Auto-loaded by Copilot CLI for every session in this repo."
globs: "**/*"
---

# Multi-Agent Orchestration Rules

## Agent Lifecycle
- Agents and sub-agents are one-shot: one request, one response, dead.
- Only the main session (orchestrator) persists across turns.
- Sub-agents cannot spawn sub-agents — only the orchestrator can deploy.

## Coordination Patterns
- **Docs-as-bus**: Write outputs to files in `docs/`. Next agent reads the file, not the orchestrator's context.
- **Shared contracts**: Define API specs in `docs/api-contract.md` before deploying parallel agents.
- **Two-step**: Manager scopes (step 1) → ICs execute in parallel (step 2). Never >3 deliverables per agent.

## When Deploying Agents
- Always include the relevant `.agent.md` persona in the prompt
- Pass only the context the agent needs — not everything
- If the task is too big, split it first (two-step pattern)
- After agent completes, save its output to `docs/` for the next agent

## Quality Gates
- >3 deliverables = must split
- >50K orchestrator context = consider Conductor (deterministic YAML)
- Parallel agents = shared contract required
- Agent says "route to X" = orchestrator must manually deploy X
