# Pattern: Two-Step (SCOPE → EXECUTE)

> Managers scope. ICs execute. Never both at once.

## The Problem

Give one agent 10 deliverables and quality degrades catastrophically. The first 3 are solid. By deliverable 7, it's cutting corners. By 10, it's outputting garbage. This is "context rot" — the agent's context fills up and quality collapses.

## The Pattern

Every deployment is two steps:

```
Step 1: SCOPE — Deploy a manager with a scoping prompt
  "You are Dev Manager. Here's the feature. Break it into assignments.
   Return: who does what, what each person's deliverable is."
  → Manager returns a WORK SPLIT (not the work itself)

Step 2: EXECUTE — Deploy ICs in parallel based on the split
  For each IC in the work split:
    → Include ONLY the context they need
    → Deploy via separate agent call
  → Aggregate results
```

## The Rule

**If a task has more than 3 deliverables, it MUST be split.**

Managers think about scope. ICs think about code. Never overload one agent with both.

## Example

```yaml
# Conductor workflow — two-step pattern
agents:
  - name: dev_manager
    prompt: "Split this feature into IC assignments..."
    output:
      backend_assignment: string
      frontend_assignment: string
    routes:
      - to: ic_team  # parallel group

  - name: ic_backend
    input:
      - dev_manager.output.backend_assignment  # focused scope only
    
  - name: ic_frontend  
    input:
      - dev_manager.output.frontend_assignment  # focused scope only
```

## Why This Works

- Managers think about SCOPE, not implementation → better work splits
- ICs get FOCUSED prompts with minimal context → better code
- Parallel execution → faster delivery
- Each agent stays within context budget → no quality degradation

## Fixes

- Mega-agent trap (one agent doing everything)
- Context rot (quality degradation as context fills)
- Orchestrator overload (smaller outputs to accumulate)
