---
title: "The language and barriers between agents"
date: 2026-11-11
---

{% include a2a/styles.html %}

I’ve been working with agents a lot this year, and one question kept bugging me. Inside a single agent, a multi-turn conversation works: the agent keeps the whole history (limited by its context window and memory) and builds on it. But what happens when the agent you’re talking to hands part of the job to another agent? [A2A (Agent2Agent) protocol](https://a2a-protocol.org) is one approach for this use case.

### What I wanted to find out

First, I wanted to understand the protocol itself. What does A2A carry between the agents? How does a multi-turn conversation work across them? Who keeps track of what was said?

Second, I wanted to know what this means for evals. To evaluate an agent properly, I need its traces. I need to see what it reasoned, which tools it called, and what those tools returned. A final answer alone tells me whether the agent was right. It does not tell me why.

Third, I wanted to know if teams can build agents that work together without ever talking to each other. That needs a contract about how each agent behaves. In traditional software, API contracts largely solve this. I wanted to see if A2A does the same.

### The setup

```
User  ⇄  Agent 1 (the agent users interact with)
              ⇅  A2A
         Agent 2 (a specialist agent: Nordic population stats)
```

The system has two agents. Agent 1 (the agent users interact with) receives every user message. When it needs specialist work, it delegates to Agent 2 (a specialist agent) over A2A. A2A is designed so that agents are opaque to each other. By default, Agent 2 does not share its memory, its tools or its internal state with Agent 1. So I expected that Agent 2's traces would not be visible from Agent 1's side. I want to know how hard it is to set up an eval across both agents, and what extra scaffolding that takes.

I used the official A2A Python SDK (version 1.3.0). Agent 2 answers questions about Nordic populations. It has two tools: `lookup_fact` and `calculate`. It works in a loop. It thinks, calls a tool, reads the result, and repeats until it can answer.

Agent 2 writes every step to a log. That log is the full trace an eval would want - this is an expected trace setup also referred to as the agent's trajectory. Agent 1 records everything it receives (this setup is not a norm - I wanted this to prove my hypothesis), including the raw HTTP traffic. Comparing the two shows exactly what crosses between them.

In the first three experiments, Agent 2 follows a script instead of using a language model. I did this so the results are repeatable and cost nothing. In the fourth experiment, Agent 2 runs on Claude.

<div class="rc-card">
  <p class="rc-card-title">Results so far</p>
  <div class="rc-grid rc-grid--2x2">
    <div class="rc-tile"><span class="rc-tag">e01</span><div class="rc-num">0 / 16</div><div class="rc-label">steps of the specialist agent's work were visible to the agent that called it</div></div>
    <div class="rc-tile"><span class="rc-tag">e02</span><div class="rc-num">33×</div><div class="rc-label">more data was sent when the calling agent resent the history instead of reusing the conversation ID</div></div>
    <div class="rc-tile"><span class="rc-tag">e03</span><div class="rc-num">11</div><div class="rc-label">wrong answers came back as “completed” from an agent with no memory</div></div>
    <div class="rc-tile"><span class="rc-tag">e04</span><div class="rc-num">11 / 12</div><div class="rc-label">times Claude without memory asked for help instead of guessing. One wrong answer still got through.</div></div>
  </div>
</div>

### Experiment 1: what does Agent 1 see of Agent 2's work?

Agent 1 asked Agent 2 two questions in the same conversation:

```
Turn 1: What is the combined population of Norway and Denmark, and what share is each?
Turn 2: Now add Sweden to that and recompute.
```

The second question only makes sense if Agent 2 remembers the first.

<div class="rc-card">
  <p class="rc-card-title">Experiment 1 · default A2A behavior</p>
  <div class="rc-grid">
    <div class="rc-tile"><div class="rc-num">16</div><div class="rc-label">intermediate steps in Agent 2's trace</div></div>
    <div class="rc-tile"><div class="rc-num">0</div><div class="rc-label">of those steps reached Agent 1</div></div>
    <div class="rc-tile"><div class="rc-num">2 / 2</div><div class="rc-label">final answers were correct</div></div>
  </div>
</div>

As expected, none of the trace reached Agent 1. For each question, Agent 1 received four events:

```
task created (submitted)
status: working   (no message)
artifact: <the final answer>
status: completed
```

This is the **basic shape of an A2A exchange**. Each request becomes a task. The task moves through states, and the answer arrives as an artifact. I also asked Agent 2 for the finished task with its full history. It returned only my own question.

Agent 2 remembered the first question because I wrote it to keep a record of each conversation. Experiment 2 looks at this more closely. On the second question, it looked up Sweden and reused the figures it already had. Agent 1 cannot see that this happened. It only sees a correct answer.

**For an eval run from Agent 1's side, this means one thing. It can grade Agent 2's final answers. It cannot grade how Agent 2 reached them.**

### Experiment 2: multi-turn conversations

Next I wanted to see how a longer conversation works across the two agents. This also matters for evals. An eval of a conversation needs to know what context and traces each agent had at each turn.

The user sent 17 messages to Agent 1. Some were small talk, which Agent 1 answered itself. The other 13 needed Agent 2. They built on each other: "add Sweden", "remove Denmark", "compare with the very first total". One request was vague: "add the remaining Nordic country". For that one, Agent 2 had to ask the user a question. A2A has a task state for this, called `input-required`.

I tested two ways for Agent 1 to pass context to Agent 2:

* **Stateful.** Agent 1 reuses the same `contextId` (from A2A protocol) and sends only the new request. Agent 2 remembers the conversation because I coded it to keep a record for each `contextId`.
* **Stateless.** Agent 1 starts a new conversation every time and pastes the full history into the message. Agent 2 does not need to remember anything.

{% include a2a/growth-chart.html %}

<div class="rc-card">
  <p class="rc-card-title">Experiment 2 · stateful vs stateless</p>
  <div class="rc-grid rc-grid--2x2">
    <div class="rc-tile"><div class="rc-num">13 / 13</div><div class="rc-label">follow-ups were answered correctly by both approaches</div></div>
    <div class="rc-tile"><div class="rc-num">437 vs 14,622</div><div class="rc-label">characters sent in total (stateful vs stateless)</div></div>
    <div class="rc-tile"><div class="rc-num">108</div><div class="rc-label">earlier requests Agent 2 had to re-read in stateless mode</div></div>
    <div class="rc-tile"><div class="rc-num">7 vs 1,011</div><div class="rc-label">characters needed to send the one-word answer “Finland”</div></div>
  </div>
</div>

Both approaches answered every follow-up correctly. The difference was cost.

In stateless mode, every message carries the whole conversation so far. The messages get bigger every turn. By turn 16, Agent 1 sent 2,055 characters to deliver a 37-character question. Agent 2 also had to read the small talk, because Agent 1 pasted everything.

In my setup, Agent 2 also sent the whole message back. The first event of every task contains the task's message history. My Agent 2 stored the incoming message in that history, and Agent 1 never limited it. Setting `history_length=0` on the request removes this echo. Stateless mode still has to send the full history, though.

The clarifying question showed the biggest gap. In stateful mode, Agent 1 answered on the same paused task with one word. In stateless mode, Agent 1 had to open a new task with the full history. The paused task was never resumed.

#### Who keeps the memory?

Stateful mode worked because Agent 2 remembered the conversation. I checked where that memory comes from.

| Layer | What it provides | Does it remember the conversation? |
|---|---|---|
| A2A protocol | A `contextId` that groups messages into one conversation | No |
| A2A SDK | ID generation and a log of tasks | No, not automatically |
| My Agent 2 | A record of each conversation, stored under its `contextId` | Yes, because I built it |

The protocol only provides the ID. The memory exists because I wrote Agent 2 to keep it. Agent 1 cannot check whether that memory exists.

**For an eval, this matters. If Agent 2 forgets, the eval needs some way to detect it. The next experiment tested whether the protocol provides one.**

### Experiment 3: Agent 2 without memory

I ran the same 17 turns in stateful mode. This time Agent 2 forgot everything before each request. I tested two versions. One crashed when it lacked context. The other never crashed and always returned an answer.

| | With memory | No memory, crashes | No memory, never crashes |
|---|---|---|---|
| Same `contextId`, valid task IDs | Yes | Yes | Yes |
| Errors seen by Agent 1 | 0 | 6 | **0** |
| Correct follow-ups | 12 / 12 | 0 / 12 | 0 / 12 |
| Wrong answers reported as `completed` | 0 | 5 | **11** |

The version that never crashed returned 11 wrong answers. Every one was marked `completed`. Agent 1 saw no errors. After "Now add Sweden", it replied:

> Sweden 10,550,000. Combined: 10,550,000. Shares: Sweden 100.0%.

The answer is well formed and wrong. The crashing version did produce errors. They came from empty lists in my code, not from anything that detected the missing memory.

This taught me something about A2A's task states (in hindsight this is kind of obvious). They describe progress. `completed` means the task finished. It does not mean the answer is right. So an eval cannot use task state as a pass or fail signal.

One of my own checks also failed here. It passed an answer that actually showed the memory loss. I come back to grading below.

### Experiment 4: a real model as Agent 2

A scripted agent only fails in the ways I wrote it to fail. So I replaced Agent 2 with Claude Opus 5.5. It had the same two tools and one more: `ask_user`, which maps to A2A's `input-required` state. I ran two versions. One kept the conversation history. The other started with no history on every request. The run cost $0.41.

<div class="rc-card">
  <p class="rc-card-title">Experiment 4 · Claude Opus 5.5 without memory</p>
  <div class="rc-grid rc-grid--2x2">
    <div class="rc-tile"><div class="rc-num">11 / 12</div><div class="rc-label">follow-ups where it paused to ask a question instead of guessing</div></div>
    <div class="rc-tile"><div class="rc-num">1</div><div class="rc-label">wrong answer was still reported as “completed”</div></div>
    <div class="rc-tile"><div class="rc-num">7</div><div class="rc-label">separate requests that Agent 1 chained into one paused task</div></div>
    <div class="rc-tile"><div class="rc-num">3.7×</div><div class="rc-label">more input tokens used by the version with memory</div></div>
  </div>
</div>

Without memory, Claude usually said it lacked context. In 11 of 12 follow-ups it asked a question instead of answering:

> I don't have a working set of countries yet. Which countries should Sweden be added to?

One wrong answer still got through. Claude had asked which country to add. The user replied "Finland". Without memory, Claude did not know what it had asked. It replied:

> Your working set is now just Finland, with a population of 5,600,000 (2024 estimate). That's also the combined total, so Finland's share is 100.0%.

The task was marked `completed`.

The questions also caused a problem in Agent 1. My Agent 1 assumed that the user's next message always answered Agent 2's open question. So it sent "Thanks, that's really helpful" to Agent 2 as an answer. Seven separate requests ended up in one paused task. A2A tells Agent 1 that Agent 2 needs input. Agent 1 has to decide whether the next user message is that input. That decision is part of Agent 1, so an eval has to cover it too.

Two more results matter for evals:

* **Memory costs tokens.** The version with memory re-read its full history on every request. It used 3.7 times the input tokens over 13 turns.
* **Exact text matching graded wrongly again.** My check marked a correct answer as wrong. Claude wrote "Finland with 5,600,000" and the check looked for "Finland 5,600,000". In experiment 3, a check passed a wrong answer. Both grading errors came from matching exact text. This is not something I am worried about because a probabilistic system's grader need to have leeway for probability in answers and not have exact text matching as I had in my bootstrap setup.

### What I learned about A2A

* **Each request is a task.** A task moves through states such as `working`, `input-required` and `completed`. The answer arrives as an artifact.
* **A `contextId` groups tasks into a conversation.** It is only a label. Any memory behind it is something the receiving agent has to build.
* **`input-required` lets the receiving agent ask a question.** The calling agent answers on the same task. Deciding which user message is the answer is up to the calling agent.
* **Task states report progress, not correctness.** A wrong answer and a right answer both end as `completed`. (again, kind of obvious!)
* **The receiving agent's work stays private.** Only its messages and artifacts cross over. This is by design.

### Can separate teams keep a contract between agents?

In traditional software, a client could mostly rely on an API model or a function signature. It told the client what to send and what comes back. I wanted to know if A2A gives separate teams something similar.

For behavior, it does not. In the [A2A specification](https://a2a-protocol.org/latest/specification/), an agent describes itself with an Agent Card. The card lists the agent's skills in free text, the media types it accepts and returns, and how to authenticate. It describes what the agent can do, not how it behaves. Memory is an example. Agents "MAY use the `contextId` to maintain internal state". Keeping memory is optional, and nothing on the card says whether an agent does it.

Extensions are the one way to agree on more. Two teams can define one and mark it as required. But the protocol does not check that an agent keeps to it. Teams that build agents separately still have to agree on behavior outside A2A.

### So, how hard is the eval?

Without extra scaffolding, one eval setup cannot cover both agents effectively. From Agent 1's side, an eval can see the user's messages, Agent 1's replies, and Agent 2's final answers. It cannot see Agent 2's trace. That follows from how A2A is designed, and the experiments confirm it.

To evaluate both agents together, I would need to add:

* **A trace channel for Agent 2.** Agent 2's owner has to export its trace somewhere the eval can read it. The `contextId` and `taskId` can link it to Agent 1's side. The A2A SDK includes OpenTelemetry hooks, but I have not yet tested whether a trace carries across both agents.
* **Grading based on meaning.** Exact text matching misjudged answers in both directions.
* **Checks on Agent 1's routing.** Agent 1 decides when to call Agent 2 and what each user message means. That is where experiment 4 went wrong.
* **Checks for memory loss.** A2A's task states do not reveal it.

**Note:** I have been part of teams that built full eval suite with LLM-as-judge graders, and I know how hard it is to calibrate for accuracy and match human judgement. It is the real work and notes on grading/grader here is anything but thorough.

### A note on production systems

These experiments used two agents and one conversation. A production agent is likely to rely on many specialist agents. Every added agent adds another opaque boundary. The eval gets harder with each one.

Take cost optimization on AWS as an example. One specialist agent finds the cost optimization recommendations. A second agent finds the current state of the resources to be optimized. A third agent carries out the recommended changes. Each hand-off between them has the same problems I saw with two agents. Context can be lost at a hand-off. An agent's memory can fail without any error. A wrong answer can still come back as `completed`. The third agent changes real infrastructure. So a mistake early in the chain can become a real change in a customer's account.

To evaluate or debug a system like this, I need to follow one request through every agent. This is operational traceability. Each agent has to export its trace. Every trace has to be linked to the same request. In real production systems this gets harder again. The data passing between agents may be encrypted. It may include customer data that must meet strict protection standards. Those standards limit what a trace can contain. They also limit where traces are stored and who can read them. The traces an eval needs are the same records that data protection rules restrict.

None of this means such systems should not be built. It means that building consequential software is real work. A2A connects the agents. Making the whole system traceable and testable is still the job of the people who build it.

---

*The code, results and per-experiment notes are at [a2a-reasoning-capture](https://github.com/somsubhro/a2a-reasoning-capture). Each experiment can be re-run with `uv run a2a-exp all`. Each one is tagged in git, so you can check out the code as it was at that point.*
