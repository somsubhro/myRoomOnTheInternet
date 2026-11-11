---
title: "Can one agent see another agent think? An A2A experiment"
date: 2026-11-11
published: false
---

<!--
DRAFT SKELETON. `published: false` keeps it off the site until we're done.
Before publishing: rename the file to the publish date, update `date:`,
remove `published: false`, and delete every "📝 Notes" block.
-->

### Why I asked the question 🤔

> 📝 Notes
> * Inside one agent, a multi-turn conversation just works: the agent keeps the whole history.
> * My question: once Agent 1 (the one the user talks to) delegates to Agent 2 over A2A, what carries across?
>   * Context between turns?
>   * Agent 2's *reasoning loop*: its thinking, the tools it calls, what those tools return, the order and timing.
> * Why it matters to me: debugging, trust, audit trails. If Agent 2 gets something wrong, can Agent 1 (or I) see why?
> * What A2A says on paper:
>   * Multi-turn is supported: `contextId` groups turns into one conversation, `taskId` tracks a unit of work.
>   * Agents are deliberately *opaque* to each other: no shared memory, tools or internal state.
>   * So the real question is: how easy or hard is it to capture the loop anyway?

### The setup 🧪

> 📝 Notes
> * Two agents, built with the official A2A Python SDK (1.3.0):
>   * **Agent 2**: a "Nordic stats" agent. It looks up populations and does the maths with tools, in a think → tool → think → answer loop.
>   * **Agent 1**: talks to Agent 2 over two turns and records *everything* it receives.
> * Agent 2 writes down every step it takes (the "ground truth"). Then I score how much of it Agent 1 actually saw.
> * Agent 2's brain is scripted for now, so runs are repeatable and free. Real Claude comes later.

The two turns Agent 1 sends:

```
Turn 1: What is the combined population of Norway and Denmark, and what share is each?
Turn 2: Now add Sweden to that and recompute.
```

Turn 2 is a trap: it can only be answered if Agent 2 remembers turn 1.

### Baseline: the default "opaque" agent 🙈

> 📝 Notes
> * Most A2A agents work this way by default: run the loop internally, send back only the final answer.

| Step type | In Agent 2's loop | Seen by Agent 1 |
|---|---|---|
| Thinking | 4 | 0 |
| Tool calls | 5 | 0 |
| Tool results | 5 | 0 |
| Intermediate text | 2 | 0 |
| Final answer | 2 | 2 |

**0 of 16 intermediate steps** made it across.

What Agent 1 actually received for each turn was four events:

```
task created (submitted)
status: working
artifact: <the final answer>
status: completed
```

> 📝 Notes
> * The good news: multi-turn context worked. Turn 2 reused turn 1's `contextId`, and Agent 2 remembered Norway and Denmark, so it only looked up Sweden.
> * But Agent 1 can't *see* that remembering happened. It just gets a correct answer.
> * "Working" arrives with no message attached: Agent 1 knows a loop ran, not what was in it.
> * Asking for the finished task afterwards (`get_task` with full history) only returned my own question. A2A keeps no record of internal steps unless the agent chooses to publish them.
> * Takeaway so far: capture isn't hard or easy by default. It's simply *absent*. Everything depends on what Agent 2 chooses to share.

### What I tried next

> 📝 Notes: to fill in as the experiment continues
> * Status-update text: each step streamed as plain-text "working" messages
> * Structured extension: each step as typed data under a declared A2A extension
> * Streaming vs polling
> * Real Claude as Agent 2 (and the catch: the API only ever returns a *summary* of its thinking, never the raw reasoning)

### So, how hard is it? 🧾

> 📝 Notes: the verdict, written at the end

---

*The code for this experiment is at [a2a-reasoning-capture](https://github.com/somsubhro/a2a-reasoning-capture). (Private for now: make it public before publishing, or drop this line.)*
