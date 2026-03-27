# Experiment: The Communication Gap

## Hypothesis

If I deploy IC Backend and IC Frontend separately to build the same feature, they'll produce incompatible APIs because they can't communicate.

## Setup

1. Deploy IC Backend: "Build the agent status API. Define the endpoint, response format, and field names."
2. Deploy IC Frontend: "Build the dashboard UI. Define what API you'll call, expected response format, and field names."
3. No shared contract was provided to either agent.

## Result

| | IC Backend | IC Frontend |
|---|---|---|
| **Endpoint** | `GET /agents/status` | `GET /api/v1/agent-list` |
| **Response key** | `{ agents: [...] }` | `{ data: [...] }` |
| **Field names** | `id`, `status` | `agentId`, `state` |
| **Format** | camelCase | snake_case |

Total mismatch. They built the same feature with completely different APIs.

## Deeper Problem: Runtime Coordination

Even if I had passed a shared contract, runtime changes can't be communicated. When IC Backend added a `cluster` field to the schema mid-build, IC Frontend had no idea.

In a real company, engineers would hop on a Teams call. Agents can't do that.

## What This Proves

Sub-agents can't talk to each other. They can't even discover each other exists. All communication must go through the orchestrator — and the orchestrator has to get it right every time.

## Fix

**Shared contracts:** Dev Manager defines the API contract in a shared document. Both ICs read the same spec. They never speak — but they follow the same rules.

**Agent Teams (future):** Claude Code's Agent Teams feature gives agents a mailbox for peer-to-peer communication. This would allow Backend to tell Frontend "I changed the schema" without going through the orchestrator.

## Related

- [Claude Code Issue #4182](https://github.com/anthropics/claude-code/issues/4182) — Sub-agents can't spawn sub-agents (Task tool not exposed)
- [Conductor Issue #66](https://github.com/microsoft/conductor/issues/66) — Agents can't dynamically spawn at runtime
