# FULL SPEECH — Lessons from Multi-Agent AI
## Read this word-for-word. Each section matches a slide.

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

You use these in VS Code Copilot, Copilot CLI, Claude Code, Conductor.

---

### [SLIDE 4 — The Setup / Org Chart]

So here's what I built. Me at the top — one Copilot session — running multiple AI companies. Each company has a CEO, VPs, Dev Managers, ICs, QA. Full org chart.

One session. Multiple companies. Talking to each CEO.

---

### [SLIDE 5 — Agents Have Rules]

And these aren't just random prompts. Every agent has a persona file. Here's the CEO's — Ria Vasquez. She CAN set vision, hire VPs, approve plans. She CANNOT write code, assign IC tasks, or skip QA.

Named. Governed. Persona reloads fresh every turn — no learning.

---

### [SLIDE 6 — Act 2 Title: "Where It Broke"]
*Pause. Let the title breathe.*

---

### [SLIDE 7 — "Every agent dies after one turn"]

Here's the baseline constraint. Every agent — no matter how well-governed — runs once, responds, and dies. One request. One response. Gone. Zero memory. The persona reloads fresh every time.

Everything that follows is a consequence of this.

---

### [SLIDE 8 — CEO Can't Hire]
*Wait for comic panels to animate in.*

I tell CEO Ria: build an agent dashboard. She says: "On it. Routing to VP Eng."

*Wait for panel 2.*

CEO Ria is dead. VP was never deployed.

*Wait for panel 3.*

Nobody hired VP. He's sitting there doing nothing.

Agents can't spawn agents. subagents... CEO can only suggest — not deploy. TODO

---

### [SLIDE 9 — "Fine, I'll Deploy VP Myself"]
*Wait for panels.*

OK, I'll do the hiring myself. I tell CEO to plan the dashboard. She gives me a great plan.

Then I deploy VP Eng and say: "CEO planned the dashboard. What's the plan?"

*Wait for the punchline panel.*

"...what plan?"

No context was passed. VP has no idea what CEO decided.

---

### [SLIDE 10 — No Communication]
*Wait for panels.*

Fine, I'll tell everyone the task directly. I deploy IC Backend — "build the status API." I deploy IC Frontend — "build the dashboard UI."

Backend builds GET /agents/status. Frontend calls GET /api/v1/agent-list.

*Wait for bottom text.*

Different endpoints. Different schemas. They never talked. TODO: agents cant talk to each other. No communication. No coordination. Just two agents doing their own thing.

---

### [SLIDE 11 — "Just pass a shared contract?"]
*Wait for panels.*

Now someone might say — just pass them both the same API spec. Fair point. But what about runtime changes?

Backend adds a "cluster" field mid-build. Frontend has no idea. There's no Teams call. No Slack. No way to tell the other agent.

Upfront contracts help. Runtime coordination doesn't exist.

---

### [SLIDE 12 — "So who's been doing all of this?"]
*Pause. Let them think.*

todo: let me take a step back.. The hiring. The context passing. The coordination. Who's been doing all of it? It wasnt me..

---

### [SLIDE 13 — The Real Mega-Agent]

*Point to the orchestrator node.*

This. The orchestrator. The main LLM session. It sits between me and every single agent. It does ALL the deploying, ALL the context passing, ALL the coordination.

And look — you can see Copilot CLI, VS Code, Claude Code on the right. It's the same in every tool.

Every agent below it? One API call. Dies.

---

### [SLIDE 14 — Context Rot]

And what happens to the orchestrator over time? It accumulates everything. Every agent's output flows back into its context window.

Turn 1 — CEO output — 2K tokens. Turn 2 — add VP — 5K. Turn 3 — TPM — 12K. By turn 7, it's at 80K+ tokens. Quality degrades. This is context rot.

And this isn't hypothetical — it happened in THIS session while building this very presentation.

---

### [SLIDE 15 — "This Isn't Just My Problem"]

Now — you might be thinking, "cool company thing, but how does this affect me?"

PR reviews. You want one agent to review code, another to analyze CI logs, another to check coverage. Same problems — context overflow, no communication, no nesting.

Code refactoring. Debugging. Anything that needs multiple perspectives hits these same walls.

This is from a real Claude Code issue — #4182. A developer trying to review PRs hit the exact same problems we just saw. anthropic closed th issue as duplicate - and just couple days back - jaypete shared a thread about Agent sessions claude todo

---

### [SLIDE 16 — Act 3 Title: "How I Adapted"]
*Pause.*

Four patterns. Each one fixes a specific failure from Act 2.

---

### [SLIDE 17 — Docs as Message Bus]

*Point to the diagram.*

The orchestrator deploys Agent A. Agent A writes its findings to a file. Then the orchestrator deploys Agent B — and Agent B reads that same file.

No context was passed between them. Just a file path. Agent B builds on Agent A's work through the file system, not through memory.

The repo is the shared memory. I borrowed this from MetaGPT's pub-sub pattern.

This fixes agent isolation and amnesia.

---

### [SLIDE 18 — Shared Contracts]

*Point to the diagram.*

The orchestrator tells the Dev Manager to define an API contract. That contract gets written down. Then both IC Backend and IC Frontend read the same contract.

They still never talk to each other. But they don't need to — they both follow the same spec.

This fixes the communication problem. The contract is the communication channel.
todo: 
---

### [SLIDE 19 — Two-Step Pattern]

*Point to the diagram.*

Step 1 — the orchestrator deploys a manager. The manager's job isn't to code — it's to split the work. Who does what.

Step 2 — the orchestrator takes that split and deploys focused ICs in parallel. Each IC gets a tight scope.

Rule: if a task has more than 3 deliverables, you must split it. Managers think about scope. ICs think about code.

This fixes the mega-agent trap and context rot. todo - explain how

---

### [SLIDE 20 — Deterministic Orchestration]

*Point to the diagram — LLM crossed out, YAML glowing.*

And the biggest fix: replace the LLM orchestrator with code.

The orchestrator doesn't need to be smart. It just needs to route outputs between agents. YAML can do that. Python can do that. You don't need an LLM accumulating 80K tokens to decide "deploy VP next."

This is what Conductor does. conductor is being developed by Jason and folks (todo!) Same orchestration pattern. But the brain is deterministic. It reads YAML, routes agent outputs, never runs out of context. And the DAG shows every step — nothing hidden.

167K tokens. 36 cents. 11 agents.

---

### [SLIDE 21 — The Full Company: CEO Says SHIP]

And here's the result. Same dashboard feature. Same agents. But now with Conductor orchestrating.

CEO routes to VP Eng. VP sets technical direction. TPM writes the spec. Dev Manager splits into IC tasks. Backend and Frontend code in parallel. QA validates.

Then the reporting chain flows back up. Dev Manager reports integration status. TPM checks spec compliance. VP gives the org-level assessment. And CEO makes the final call.

*Point to the green text.*

SHIP. All requirements met. 167K tokens. 36 cents. 11 agents. One feature lifecycle — from "build a dashboard" to "ship it."

---

### [SLIDE 22 — Three Rules / Takeaway]

*Slow down. Let each rule land.*

Three rules I walked away with.

One — context is RAM, not disk. It's finite. It's precious. Don't overload agents. Split the work.

Two — the repo is the shared memory. Agents can't remember. Files can. Use docs as your message bus.

Three — don't anthropomorphize (todo) agents. They're not employees. They're stateless functions with good UX. Design for that.

*Pause.*

Agents are not employees. They're functions that happen to speak English.

Thank you.

---

## TIMING GUIDE
- Act 1 (Slides 1-5): ~2.5 min
- Act 2 (Slides 6-15): ~4 min
- Act 3 (Slides 16-21): ~3 min  
- Takeaway (Slide 22): ~0.5 min
- Total: ~10 min
