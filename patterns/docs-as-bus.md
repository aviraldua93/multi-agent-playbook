# Pattern: Docs-as-Message-Bus

> Agents can't remember. Files can.

## The Problem

Agents are stateless. When Agent A finishes, it dies. Agent B — deployed fresh — has no idea what Agent A did. The orchestrator could pass context, but that accumulates in the orchestrator's context window (context rot).

## The Pattern

Instead of passing context through the orchestrator, agents read from and write to shared files in the repo.

```
Orchestrator
  → deploys Agent A
      Agent A writes → docs/findings.md

  → deploys Agent B (later)
      Agent B reads  → docs/findings.md
      Agent B builds on Agent A's work ✓
```

No context was passed between agents. Just a file path. The repo is the message bus.

## How It Works

1. **Agent A** receives a task and writes its findings/output to a markdown file in `docs/`
2. **The orchestrator** deploys Agent B with a prompt that says "read `docs/findings.md`"
3. **Agent B** reads the file and continues the work
4. The orchestrator's context stays small — it only knows the file paths, not the content

## When to Use

- Agent B needs Agent A's full output (not just a summary)
- The orchestrator would accumulate too much context forwarding outputs
- Multiple agents need to read the same shared state
- You want an audit trail of what each agent produced

## Origin

Borrowed from [MetaGPT's](https://github.com/geekan/MetaGPT) publish-subscribe pattern, where agents declare what they "watch" and react to changes in shared state.

## Fixes

- Agent isolation (amnesia)
- Orchestrator context accumulation
- Lack of audit trail
