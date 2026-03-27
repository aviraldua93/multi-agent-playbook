# Experiment: The Amnesia Test

## Hypothesis

If I deploy CEO Ria to plan a feature, then deploy VP Eng as a fresh agent and ask "what did the CEO decide?" — VP Eng will have zero knowledge.

## Setup

1. Deploy sub-agent as CEO Ria: "Build an agent dashboard. Plan it."
2. CEO produces: "Real-time status, agent cards, WebSocket updates. Routing to VP Eng."
3. Deploy a DIFFERENT sub-agent as VP Eng: "CEO planned the dashboard. What's the plan?"
4. No CEO output was included in VP Eng's prompt.

## Result

VP Eng responded: **"...what plan? I have no context about any CEO decision."**

Total amnesia. The information never flowed down because:
- CEO ran, produced output, and **died**
- VP Eng is a completely fresh LLM invocation
- No state transferred between them
- They share nothing — not memory, not context, not even a file

## What This Proves

The CEO → VP → IC hierarchy is fiction at runtime. Each agent is alone in the universe. The hierarchy only works if the orchestrator manually passes context from one to the next.

## Fix

**Docs-as-message-bus:** CEO writes plan to `docs/execution-plan.md`. VP Eng reads that file. No context passing through the orchestrator needed.
