# FULL SPEECH — Lessons from Multi-Agent AI
## Each section matches a slide. Verbal cues in *italics*.

---

### [SLIDE 1 — Title]
*Pause 3 seconds. Let them read.*

Hi everyone. I'm Aviral. Today I'm going to talk about what happens when you push multi-agent AI to its limits. Not the theory — the reality.

---

### [SLIDE 2 — "Push multi-agent to its limits"]

I wanted to test how far multi-agent can go. Not just one Copilot session — but multiple agents working together like a team. A company structure was the test case. CEO, VPs, engineers — all AI agents. Shoutout to Jaypete whose "Corporation" project inspired this.

---

### [SLIDE 3 — Building Blocks]

Before I show you what I built, let's get on the same page. There are three things in multi-agent.

An Agent — that's an LLM with a persona file. It has CAN and CANNOT rules. Think of it like a job description for AI. It runs once, produces output, and dies.

A Sub-Agent — same LLM, but no persona file. Anonymous. No governance. Fire and forget.

And the Main Session — this is the orchestrator. It's the only thing that persists. It spawns agents, passes context between them, and manages everything.

You see these in VS Code Copilot, Copilot CLI, Claude Code, Conductor.

---

### [SLIDE 4 — The Setup / Org Chart]

So here's what I built. Me at the top — one Copilot session — running multiple AI companies. Each company has a CEO, VPs, Dev Managers, ICs, QA. Full org chart.

One session. Multiple companies. Talking to each CEO.

---

### [SLIDE 5 — Agents Have Rules]

And these aren't just random prompts. Every agent has a persona file. Here's the CEO's — Ria Vasquez. She CAN set vision, hire VPs, approve plans. She CANNOT write code, assign IC tasks, or skip QA.

Named. Governed. But the persona reloads fresh every turn — no learning between invocations.

---

### [SLIDE 6 — "Where It Broke"]
*Pause. Let the title breathe.*

---

### [SLIDE 7 — "Every agent dies after one turn"]

Here's the baseline constraint. Every agent — no matter how well-governed — runs once, responds, and dies. One request. One response. Gone. Zero memory.

Everything that follows is a consequence of this.

---

### [SLIDE 8 — CEO Can't Hire]
*Wait for comic panels to animate in.*

I tell CEO Ria: build an agent dashboard. She says: "On it. Routing to VP Eng."

*Wait for panel 2.*

CEO Ria is dead. VP was never deployed.

*Wait for panel 3.*

Nobody called VP. The persona file says "route to VP" — but agents don't have a tool to deploy other agents. They can only suggest. The orchestrator has to do the actual deploying.

---

### [SLIDE 9 — "Fine, I'll Deploy VP Myself"]
*Wait for panels.*

OK so I'll do the hiring myself. I deploy VP Eng and say: "CEO planned the dashboard. What's the plan?"

*Wait for the punchline.*

"...what plan?"

No context was passed. VP has no idea what CEO decided. The CEO's output is sitting in my context — the orchestrator's context — but I didn't forward it.

---

### [SLIDE 10 — No Communication]
*Wait for panels.*

Fine, I'll give everyone the task directly. I deploy IC Backend — "build the status API." I deploy IC Frontend — "build the dashboard UI." Separately. In parallel.

Backend builds GET /agents/status. Frontend calls GET /api/v1/agent-list.

*Wait for bottom text.*

Different endpoints. Different schemas. They never talked. Agents can't communicate with each other — there's no peer-to-peer channel. Everything has to go through the orchestrator.

---

### [SLIDE 11 — "Just pass a shared contract?"]
*Wait for panels.*

Now someone might say — just pass them both the same API spec upfront. Fair point. But what about runtime changes?

Backend adds a "cluster" field mid-build. Frontend has no idea. There's no Teams call. No Slack. No way to tell the other agent.

Upfront contracts help. Runtime coordination doesn't exist.

And here's something Anthropic's engineering team found — agents can't even evaluate their own work reliably. They praise their own output. You need separate agents: one to do the work, one to judge it.

---

### [SLIDE 12 — "So who's been doing all of this?"]
*Pause. Let them think.*

So let me take a step back. The hiring. The context passing. The coordination. Who's been doing all of it?

It wasn't me — not directly.

---

### [SLIDE 13 — The Real Mega-Agent]

*Point to the orchestrator node.*

This. The orchestrator. The main LLM session. It sits between me and every single agent. It does ALL the deploying, ALL the context passing, ALL the coordination.

And look — Copilot CLI, VS Code, Claude Code — it's the same pattern in every tool. There's always one persistent session doing everything.

Every agent below it? One API call. Dies.

---

### [SLIDE 14 — Context Rot]

And what happens to the orchestrator over time? It accumulates everything. Every agent's output flows back into its context window.

*Point to the growing bars.*

Turn 1 — 2K tokens. Turn 3 — 12K. By turn 7, you're at 80K+ tokens. Quality degrades. Anthropic documented this — they call it coherence loss. Models literally lose track of earlier decisions as context fills. Some even start wrapping up work prematurely because they think they're running out of space.

This isn't hypothetical — it happened to this very session while building this presentation.

---

### [SLIDE 15 — "This Isn't Just My Problem"]

Now — you might be thinking, "cool company experiment, but how does this affect me?"

PR reviews. You want one agent to review code, another to analyze CI logs, another to check coverage. Same problems — context overflow, no communication, no nesting.

Code refactoring. Debugging. Anything that needs multiple perspectives hits these same walls.

A developer filed an issue on Claude Code about this exact problem — trying to do PR reviews with nested agents. Same failures we just saw. Anthropic closed it as a duplicate — they know.

---

### [SLIDE 16 — "How I Adapted"]
*Pause.*

Four patterns. Each one fixes a specific failure from Act 2.

---

### [SLIDE 17 — Docs as Message Bus]

*Point to the diagram.*

The orchestrator deploys Agent A. Agent A writes its findings to a file. Then the orchestrator deploys Agent B — and Agent B reads that same file.

No context was passed between them. Just a file path. Agent B builds on Agent A's work through the file system, not through memory.

The repo is the shared memory. I borrowed this from MetaGPT's pub-sub pattern.

---

### [SLIDE 18 — Shared Contracts]

*Point to the diagram.*

The Dev Manager defines an API contract. That contract gets written down. Then both IC Backend and IC Frontend read the same contract.

They still never talk to each other. But they don't need to — they both follow the same spec. The contract is the communication channel.

---

### [SLIDE 19 — Two-Step Pattern]

*Point to the diagram.*

Step 1 — deploy a manager. The manager's job isn't to code — it's to split the work. Who does what.

Step 2 — take that split and deploy focused ICs in parallel. Each IC gets a tight scope and fresh context.

Rule: if a task has more than 3 deliverables, split it. This is how you avoid the mega-agent trap — instead of one agent drowning in context, you have three agents each with a clean slate. Anthropic's engineering team converged on the same pattern — they call it planner, generator, evaluator.

---

### [SLIDE 20 — Deterministic Orchestration]

*Point to the diagram — LLM crossed out, YAML glowing.*

And the biggest fix: replace the LLM orchestrator with code.

The orchestrator doesn't need to be smart. It just needs to route outputs between agents. YAML can do that. Python can do that. You don't need an LLM accumulating 80K tokens to decide "deploy VP next."

This is what Conductor does — developed by Jason and folks at Microsoft. Same orchestration pattern. But the brain is deterministic. It reads YAML, routes agent outputs, never runs out of context. And the DAG shows every step — nothing hidden.

---

### [SLIDE 21 — Where the Industry Is Heading]

So where does this go from here?

*Point to the three cards.*

Today we have sub-agents — one-shot functions. They run, they die. That's what we've been working with this whole talk.

But Claude Code just shipped something experimental called Agent Teams. Teammates can message each other directly. They have a shared task list. They self-coordinate. They're not one-shot anymore — they're persistent services with a mailbox.

And beyond that? Self-organizing teams. Dynamic spawning. No human orchestrator needed. We're not there yet — but that's the direction.

The key insight: sub-agents die because nobody talks to them after the first response. Give them a mailbox, and they persist. Same LLM. Different conversation loop.

---

### [FINAL SLIDE — Closing Quote]

*Let it sit on screen. Pause.*

"Agents are not employees. They're functions that happen to speak English."

Thank you.

---

## TIMING GUIDE
- Act 1 — Slides 1-5: ~2 min
- Act 2 — Slides 6-15: ~4.5 min
- Act 3 — Slides 16-20: ~2.5 min
- Closing — Slides 21-22: ~1 min
- Total: ~10 min

## VERBAL DROPS (say naturally, don't read)
- Anthropic: "Models lose coherence as context fills" → Context Rot slide
- Anthropic: "Agents praise their own work" → Contracts slide
- Anthropic: "Planner, generator, evaluator" → Two-Step slide
- Meta: "This session is live proof — my context is massive right now" → Context Rot slide
