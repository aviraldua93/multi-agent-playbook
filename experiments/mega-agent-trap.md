# Experiment: The Mega-Agent Trap

## Hypothesis

If I give one agent the entire dashboard feature — backend API, frontend UI, WebSocket streaming, auth, tests, docs — quality will degrade as context fills up.

## Setup

One sub-agent with a single prompt: "Build a complete agent dashboard. Produce ALL deliverables with real code."

Eight deliverables requested:
1. Agent status API
2. Data models
3. Status endpoint
4. Dashboard UI
5. WebSocket streaming
6. Auth + RBAC
7. Tests
8. Docs

## Result

| Deliverable | Quality |
|---|---|
| 1. Agent status API | ✅ Solid — well-structured, correct types |
| 2. Data models | ✅ Solid — matches the API contract |
| 3. Status endpoint | ✅ Solid — proper error handling |
| 4. Dashboard UI | ⚠️ Corners cut — generic component, missing features |
| 5. WebSocket streaming | ⚠️ Half-baked — basic implementation, no reconnection logic |
| 6. Auth + RBAC | ❌ Barely there — placeholder middleware |
| 7. Tests | ❌ "TODO" comments instead of actual tests |
| 8. Docs | ❌ Empty or single-paragraph |

## What This Proves

**Context is RAM, not disk.** As the agent's context fills with earlier deliverables, later deliverables get less attention and lower quality. The agent does an "OK" job on everything but a great job on nothing.

## The Rule

**If a task has more than 3 deliverables, it MUST be split.** Use the two-step pattern: manager scopes → ICs execute in parallel.

## Fix

**Two-step pattern:** Deploy a manager to split the work into 3 IC assignments (backend, frontend, QA). Each IC gets focused scope and fresh context.
