---
title: "Trying is the strategy"
date: 2026-03-22
---
*this writing was co-authored with AI*

Over the past several months, my team within AWS Cost Management has been running an experiment: adopting AI-native practices in how we build, not just in what we build. 
This post is about what we've learned (thus far). Not a framework. Not a maturity model. Just observations from a team that decided the only way to form real opinions about AI-native development was to actually do it.
The adoption wasn't always organic. There was skepticism — some of it well-founded. There is no single right way to do this. 
But we felt strongly that you can't evaluate a new way of working from the sidelines. 
You have to get your hands dirty, see what works, see what doesn't, and figure out what needs to change. Trying is the strategy.

### The skepticism is real — and it's not unreasonable

Let me be clear about something upfront: the skepticism around AI-assisted development isn't irrational. When someone on your team says "I tried XXXXX and it hallucinated an API that doesn't exist," they're not being a Luddite. They're reporting a real experience. When a senior engineer says "I'm faster without it," they might be right — for now, for their workflow, for the kind of work they do.

The mistake isn't skepticism. The mistake is letting skepticism become a conclusion before it's been tested as a hypothesis.

### There's no one right way to adopt

Many blog posts hand you a neat adoption framework with phases and gates. I'm not going to do that, because it would be dishonest. We didn't follow one. What we did instead was establish a few principles: 

* We would try AI-native practices across real work, not sandboxed experiments. Toy problems produce toy insights.

* We would make adoption voluntary but visible. Nobody was forced to use any specific tool or workflow. But we asked everyone to share what they tried, what worked, what didn't. The visibility mattered more than the mandate.

* We would treat our observations as data, not verdicts. "This didn't work for code reviews" isn't a permanent conclusion — it's a data point from a specific tool, at a specific point in time, applied to a specific kind of review. The tools are changing fast. Our conclusions need to be held loosely.

### What we tried

We didn't start from scratch. Amazon's internal builder tools team maintains a reference of available AI-assisted tooling, organized around the phases of the software development lifecycle. Most of these tools take the form of Agents, Skills, or MCPs — each targeting a specific part of the workflow: design, implementation, testing, review, deployment. We started there, picking up what was available and applying it to real work.

But the off-the-shelf tooling only got us so far. The interesting shift happened when we started codifying our own ways of working through bespoke Agents and Skills. We built code review steering files that encoded our team's review expectations. We built design document review steering that reflected how we actually evaluate technical proposals — not generic best practices, but our bar. For narrative writing, which plays a particular role in Amazon's culture, we developed a power-user skill that provided consistency in how prose quality is adjudicated. The pattern was the same each time: take a workflow where humans were applying implicit standards, make those standards explicit, and give an AI enough context to apply them.

The most impactful example was around requirements. At Amazon, many projects and new launches begin with a PRFAQ — a document that articulates the customer problem and the proposed solution. PRFAQs are valuable for alignment, but there's typically a long gap between having a PRFAQ and having requirements that engineers can actually implement against. We realized this translation step — from PRFAQ to implementable specs — was a bottleneck worth attacking. So we designed a skill to help translate PRFAQs into EARS-format specifications. And here's the part that still feels a little recursive: we used an AI skill to design the skill itself. The result meaningfully shortened the time between "we have a PRFAQ" and "engineers can start building."
