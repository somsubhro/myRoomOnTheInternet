---
title: "Is AI replacing your engineering instinct?"
date: 2026-10-05
---

In an earlier post, I asked a question that has stayed with me:

*If your first experience of doing something is with AI doing most of the work, do you ever develop the instincts to know when it's getting things wrong?*

Experienced engineers often recognize when something doesn't feel right. A design might look reasonable, the code might pass every test, and yet something makes them uncomfortable. Perhaps it's an unnecessary abstraction, a hidden dependency, a familiar failure mode, or an unwritten understanding of how customers actually use the product. I call this engineering instinct.

But where does it come from?

I believe it develops through years of making decisions, getting things wrong, understanding why, and applying those lessons. For a curious engineer, **the feedback loop is also the learning loop.**

**What happens when AI starts removing much of that feedback loop?**

An engineer can now describe a problem, have an agent design a solution, generate the implementation, and write tests. The result might work perfectly. But did the engineer understand why that design was chosen? Could they recognize a flawed assumption? Would they know when the same solution shouldn't be used?

I think **the ability to produce software is becoming increasingly disconnected from the ability to understand it.**

Of course, we've always built on abstractions. Engineers don't need to understand how a database engine works to use a database. Why should AI be different?

AI doesn't just abstract implementation details. It increasingly makes engineering decisions on our behalf. Evaluating those decisions requires the very judgment we may no longer be developing.

There's a compelling counterargument. AI could accelerate learning. Curious engineers can explore more designs, question more assumptions, and experiment with unfamiliar systems. I believe that too. But curiosity alone doesn't guarantee learning, and delivering working software doesn't necessarily demonstrate understanding.

This creates a problem for engineering leaders. We measure how much faster engineers deliver with AI. But are we paying equal attention to whether they're developing the judgment to evaluate its output?

Perhaps future AI systems will become so reliable that much of today's engineering instinct becomes unnecessary. That's possible. But we're not there yet.

On my team, we still encourage newer engineers to build fundamentals through developer guides and bootcamps, reason through design and code decisions, and participate actively in design discussions and reviews. These conversations force engineers to explain their thinking, challenge assumptions, and answer questions in real time. On-call experience remains equally important. We have AI tools to inspect logs, correlate events, and reason about failures. But understanding an incident's impact, deciding how to respond, and experiencing the consequences of engineering decisions continue to build judgment.

I don't believe restricting AI is the answer. But I also don't know if our current approaches are enough. We're largely relying on the same methods we've always used to develop engineers, while the way they build software is changing dramatically.
