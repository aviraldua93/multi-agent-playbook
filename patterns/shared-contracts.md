# Pattern: Shared Contracts

> ICs don't talk. They read the same spec.

## The Problem

When two agents work on the same feature independently, they produce incompatible implementations. IC Backend builds `GET /agents/status`, IC Frontend calls `GET /api/v1/agent-list`. Different endpoints, different schemas. They never coordinated.

Even with upfront contracts, runtime changes can't be communicated. If Backend adds a `cluster` field mid-build, Frontend has no way to know.

## The Pattern

A manager agent defines an API contract. Both ICs read the same contract. They never talk to each other — but they don't need to.

```
Orchestrator
  → deploys Dev Manager
      Dev Manager writes → docs/api-contract.md
      
  → deploys IC Backend
      IC Backend reads → docs/api-contract.md
      IC Backend implements the contract
      
  → deploys IC Frontend
      IC Frontend reads → docs/api-contract.md
      IC Frontend calls the contract
```

## Key Insight

The contract IS the communication channel. Agents don't need peer-to-peer messaging if they align through a shared document. This is how real companies work too — teams coordinate through specs, not hallway conversations (most of the time).

## Limitations

- Contracts are static. If an agent needs to change the contract mid-build, other agents won't know.
- Requires the contract to be comprehensive upfront. Underspecified contracts lead to mismatches.
- Works best when the work is truly parallel and independent.

## Fixes

- No peer communication between agents
- API mismatches in parallel work
- Partial fix for runtime coordination (upfront contracts help, but can't solve runtime changes)
