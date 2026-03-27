# Multi-Agent AI Playbook

> What I learned running 120+ AI agents organized like a company. Not theory — operational experience with real failures, real patterns, and real code.

[![Conductor Issues Filed](https://img.shields.io/badge/Conductor_Issues_Filed-3-blue)](https://github.com/microsoft/conductor/issues?q=author%3Aaviraldua93)
[![Claude Code Reference](https://img.shields.io/badge/Claude_Code_Issue-4182-purple)](https://github.com/anthropics/claude-code/issues/4182)

---

## The Experiment

I wanted to push multi-agent AI to its limits. Not just one Copilot session — but multiple agents working together like a real software company: CEO, VPs, Dev Managers, TPMs, ICs, QA. Full org charts, persona files, CAN/CANNOT boundaries.

**The result:** Everything broke in predictable ways. Then I figured out patterns that made it work. This playbook documents both.

**Case study:** [AgentOS](https://github.com/aviraldua93/agentos) — an agent dashboard built entirely by AI agents using these patterns.

---

## The Core Problem: Agent Isolation

Every agent — no matter how well-governed — runs once, responds, and dies. Zero memory. The persona reloads fresh every time. This creates three symptoms:

### 1. Agents Can't Spawn Agents

CEO Ria's persona file says "route to VP Eng." But she has no tool to deploy VP Eng. She can only produce text suggesting it. The orchestrator (main session) must read that text and manually deploy the next agent.

```
CEO: "Routing to VP Eng..."  →  just text output
VP Eng: never deployed (nobody read the suggestion)
```

**The persona file is a suggestion, not an action.** Only the orchestrator can deploy.

### 2. Agents Can't Remember Each Other

Deploy CEO → she produces a plan. Deploy VP Eng → "what plan?" Total amnesia. No context was passed because:
- CEO ran, produced output, and **died**
- VP Eng is a completely fresh invocation
- No state transferred between them

### 3. Agents Can't Talk to Each Other

IC Backend builds `GET /agents/status`. IC Frontend calls `GET /api/v1/agent-list`. Same feature, completely different APIs. They never talked — there's no peer-to-peer communication channel.

Even with upfront contracts, runtime changes can't be communicated. If Backend adds a `cluster` field mid-build, Frontend has no idea. There's no Teams call. No Slack. No way to tell.

---

## The Hidden Mega-Agent: The Orchestrator

All coordination falls on one entity — the **orchestrator** (the main LLM session). It does ALL the deploying, ALL the context passing, ALL the coordination.

```
Owner (you)
  └── Orchestrator (main LLM session)  ← THE REAL MEGA-AGENT
        ├── CEO        ← one API call, dies
        ├── VP Eng     ← one API call, dies
        ├── DevMgr     ← one API call, dies
        ├── IC Backend ← one API call, dies
        └── IC Frontend← one API call, dies
```

**The orchestrator accumulates everything.** Every sub-agent result flows back into its context window. 10 sub-agents = 30K+ tokens added. Context fills → quality degrades. This is **context rot**.

And the worst part? **The orchestrator is invisible in every tool.** Not on Conductor's DAG. Not in Mission Control. Nobody's watching the brain.

This isn't just a Copilot CLI problem:

| Tool | Orchestrator | Accumulates context? |
|---|---|---|
| Copilot CLI | Main session (LLM) | Yes |
| Claude Code | Main session (LLM) | Yes |
| CrewAI | Supervisor agent (LLM) | Yes |
| AutoGen | GroupChatManager (LLM) | Yes |
| **Conductor** | **Python script** | **No — stateless routing** |

---

## Patterns That Work

### Pattern 1: Docs-as-Message-Bus

**Fixes:** Agent isolation, amnesia

Agents can't remember. Files can. Instead of passing context through the orchestrator, agents read from and write to shared documents in the repo.

```
Agent A → writes to docs/findings.md
Agent B → reads docs/findings.md (no context passed, just the file path)
Result: Agent B builds on Agent A's work ✓
```

The repo IS the shared memory. Borrowed from [MetaGPT's](https://github.com/geekan/MetaGPT) pub-sub pattern.

### Pattern 2: Shared Contracts

**Fixes:** No communication, runtime coordination

ICs don't talk to each other. They read the same API spec. The Dev Manager defines a contract, both ICs follow it.

```
Dev Manager defines: GET /agents/status → { agents: [...] }
IC Backend implements it.
IC Frontend calls it.
They never speak. It works.
```

### Pattern 3: Two-Step Pattern (SCOPE → EXECUTE)

**Fixes:** Mega-agent trap, context rot

Never give one agent more than 3 deliverables. Split every big task:

```
Step 1: Manager → "split the work" → returns: who does what
Step 2: ICs → execute in parallel → each gets focused scope only
Rule: >3 deliverables = must split
```

### Pattern 4: Deterministic Orchestration

**Fixes:** Orchestrator overload, invisible brain

Replace the LLM orchestrator with code. The orchestrator doesn't need to be smart — it just needs to route outputs between agents. YAML can do that.

[Conductor](https://github.com/microsoft/conductor) does exactly this: reads YAML, routes agent outputs, never accumulates context. The DAG shows every step — nothing hidden.

```yaml
# Conductor workflow — deterministic, visible, stateless
agents:
  - name: ceo
    routes:
      - to: vp_team
  - name: vp_engineering
    input:
      - ceo.output  # context passed explicitly in YAML
```

---

## Real-World Comparison

| | Copilot CLI | Conductor | Claude Code Agent Teams |
|---|---|---|---|
| Orchestrator | LLM (non-deterministic) | Python (deterministic) | Lead agent (LLM) |
| Can adapt? | Yes | No (fixed YAML) | Yes |
| Context limit? | Yes — degrades | No — stateless | Yes, but distributed |
| Agents visible? | Sub-agents invisible | All visible on DAG | All visible |
| Peer communication? | No | No | Yes (mailbox) |
| Dynamic spawning? | Yes (main session only) | No | Yes (lead spawns) |

---

## Where the Industry Is Heading

**Agent Teams** (Claude Code) solves the core problems:
- Teammates message each other **directly** — not just report to orchestrator
- Shared task list with dependencies and self-coordination
- Each teammate is a full independent session with its own context window
- Sub-agents become persistent services (they have a mailbox, so they wait for messages instead of dying)

The evolution: **Sub-agents** (one-shot functions) → **Agent Teams** (persistent services) → **?** (autonomous coordination)

Related: [Claude Code Issue #4182](https://github.com/anthropics/claude-code/issues/4182) — "Sub-Agent Task Tool Not Exposed When Launching Nested Agents" (8 upvotes, confirmed by Anthropic)

---

## Issues Filed from Real Failures

During this work, I filed 3 issues on [Microsoft Conductor](https://github.com/microsoft/conductor) from actual workflow failures:

| Issue | Problem | Status |
|---|---|---|
| [#64](https://github.com/microsoft/conductor/issues/64) | LLMs return wrong types for output schemas (dict instead of string) | Open |
| [#65](https://github.com/microsoft/conductor/issues/65) | Same agent can't both assign work and review results (no staged re-invocation) | Open |
| [#66](https://github.com/microsoft/conductor/issues/66) | Agents can't dynamically spawn sub-agents at runtime | Open |

---

## Key Insights

1. **The hierarchy is a prompt design pattern, not management.** Every agent dies after one turn. CEO→VP→IC doesn't layer at runtime. It's one orchestrator dispatching stateless workers.

2. **Context is RAM, not disk.** Finite. Precious. Don't overload agents. Every layer in the hierarchy = another LLM call, another context pass, another point of failure.

3. **Agents are not employees.** They're stateless functions with good UX. Design for that. The `.agent.md` file gives them identity and governance, but NOT persistence.

4. **The repo is the shared memory.** Agents can't remember. Files can. Docs-as-message-bus works.

5. **You don't need hierarchy — you need coordination.** Flat + orchestrator + shared docs works better than deep hierarchy. Every layer is overhead.

6. **The orchestrator is the single point of failure AND it's invisible.** The most critical agent in the system, and no tool shows its status.

7. **Sub-agents die because nobody talks to them** — not because they technically can't persist. A mailbox (Agent Teams pattern) is all it takes.

---

## Case Study

**[AgentOS](https://github.com/aviraldua93/agentos)** — an agent status dashboard, built using these patterns:
- Specs generated by a 13-agent Conductor workflow
- Code written by sub-agents reading from `docs/` (docs-as-bus)
- `.agent.md` persona files for every role
- Working Conductor workflows you can run

---

## Repository Structure

```
multi-agent-playbook/
├── README.md              ← You are here
├── patterns/              ← Detailed pattern docs
│   ├── docs-as-bus.md
│   ├── shared-contracts.md
│   ├── two-step-pattern.md
│   └── deterministic-orchestration.md
├── experiments/           ← Documented experiments
│   ├── amnesia-test.md
│   ├── mega-agent-trap.md
│   └── communication-gap.md
├── workflows/             ← Conductor YAML files
│   ├── company-v3.yaml    ← 13-agent org chart
│   └── parallel-research.yaml
└── talks/                 ← Presentation materials
    ├── slides.html
    └── speech.md
```

---

## Tools Used

- **[GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli)** — Terminal-based multi-agent with sub-agents
- **[Microsoft Conductor](https://github.com/microsoft/conductor)** — YAML-based deterministic multi-agent workflows
- **[Mission Control](https://github.com/builderz-labs/mission-control)** — Agent fleet management dashboard
- **[Claude Code](https://code.claude.com)** — Agent Teams for peer-to-peer coordination

---

*Built during a single overnight session preparing for an ADX Share & Learn presentation. The presentation itself was built collaboratively with an AI orchestrator — and yes, the orchestrator ran out of context multiple times. That's the whole point.*
