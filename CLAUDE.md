# Multi-Agent AI Playbook — Project Rules

This repo contains operational patterns for multi-agent AI. When working on multi-agent projects, follow these rules:

## Core Constraints
- Every agent (and sub-agent) runs once, responds, and dies. Zero memory between invocations.
- Only the orchestrator (main session) persists. It accumulates context every turn.
- Sub-agents cannot spawn sub-agents. Only the orchestrator can deploy.
- Agents cannot communicate peer-to-peer. All coordination goes through the orchestrator or shared files.

## Patterns (Always Apply)
1. **Docs-as-Bus**: Agents read from and write to shared files. The repo is the shared memory. Never rely on the orchestrator to forward context — write it to a file.
2. **Shared Contracts**: When deploying parallel agents, define the API contract first. Both agents read the same spec. They never talk directly.
3. **Two-Step (SCOPE → EXECUTE)**: Never give one agent more than 3 deliverables. Step 1: deploy a manager to split the work. Step 2: deploy ICs in parallel with focused scope.
4. **Deterministic Orchestration**: For workflows with >5 agents, use Conductor (YAML-based) instead of LLM orchestration. The orchestrator doesn't need to be smart — it just needs to route.

## Anti-Patterns (Never Do)
- ❌ **Mega-agent**: One agent with 10 tasks → context rot, quality collapse
- ❌ **Raw output forwarding**: Passing unstructured agent output to the next agent → noise
- ❌ **Expecting peer communication**: Agents cannot talk to each other — design for isolation
- ❌ **Deep hierarchy**: Every layer = another LLM call + context pass. Keep it flat.

## Agent Persona Files
When creating agents, every `.agent.md` file must include:
- **CAN**: What this agent is allowed to do
- **CANNOT**: What this agent must NOT do (most important section)
- **Reports to**: Who this agent's output goes to
- **Context**: What files/docs this agent should read

## Quality Gates
- If a task has >3 deliverables → MUST split
- If orchestrator context exceeds ~50K tokens → consider Conductor
- If agents produce incompatible outputs → add a shared contract
- If an agent needs another agent's output → use docs-as-bus, not orchestrator forwarding
