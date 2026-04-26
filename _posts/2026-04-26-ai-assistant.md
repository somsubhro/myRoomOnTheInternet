---
title: "Your AI assistant should remember you"
date: 2026-04-26
---

# Your AI assistant should remember you
*This entire post was drafted by my AI assistant. I course-corrected it two times.*

A few months ago, I started using a personal AI assistant as a personal work organizer. Not a chatbot. Not a summarizer. A persistent assistant that knows my projects, tracks my action items, remembers my preferences, and picks up where we left off every morning.

The trend was started by one of the senior SDEs within my org (and I think few others may have been involved or ideated similar concepts). A markdown file with instructions. A directory for notes. A handoff file that captures what happened at the end of each session so the next one doesn't start cold. Over time, the directory grew. The assistant learned what I care about and what I consider noise. It started flagging things I'd forgotten. It pushed back when my priorities didn't match my actions.

The surprising part wasn't that it worked. The surprising part was how little infrastructure it needed.

### The memory is the product

Most AI assistant setups focus on the model. Which model, which API, which framework. But after months of daily use, I can tell you: the model is the least interesting part. What makes session 50 dramatically better than session 1 is the accumulated context. Preferences. People. Project history. Noise filters. The things you told it once and never had to repeat.

The assistant is the interface. The memory is the product.

And the memory is just files. Markdown files in a directory. A handoff file that gets overwritten every session. A preferences file that grows as the assistant learns what you like and what you don't. An actions file that tracks commitments across projects. Daily logs. Project notes. All plain text, all version-controllable, all readable without any special tooling.

There's no database. No running service. No Python environment to keep alive. The LLM CLI you already use is the runtime. The directory is the state.

### What actually matters

After iterating on this for a while, I've found that a few patterns matter more than everything else combined:

**The handoff.** Every session ends with the assistant writing a summary of what happened, what's pending, and what the next session should pick up on. Every session starts by reading it. This single file is what turns a stateless chat into a persistent relationship. Without it, every morning is a blank slate. With it, the assistant greets you knowing what you were working on yesterday and what's due this week.

**The action list.** A single file that tracks open commitments across all projects. Not buried in daily logs. Not scattered across project files. One place. The assistant adds items when you make commitments during a session, marks them done when you finish, and carries them forward with countdown timers when you don't. If something has been open for a week with no progress, it tells you.

**The noise filter.** Half of making an assistant useful is teaching it what not to show you. Meeting RSVPs. Automated notifications. Threads that aren't yours anymore. The assistant learns these over time and writes them down. After a few weeks, the morning briefing only contains things that actually need your attention.

**Self-evolution.** The directory structure you start with is a scaffold, not a cage. The assistant should create new files, new folders, new routines as it learns how you work. If it never changes its own structure, it'll feel static within a week. The best additions to my setup were things the assistant created on its own because it noticed a pattern.

### What I built

I packaged this into a small open source kit. It's a shell script and a skill file. You run the script, choose your LLM CLI, and the skill interviews you about who you are, how you work, and what you need. Then it generates a complete assistant directory: identity, persona, memory files, action tracking, daily logs, and two starter skills for creating and reviewing new skills.

Once it's set up, you type your assistant's name in the terminal and you're in a session. When you're done, it writes the handoff. Tomorrow, it picks up where you left off.

The whole thing is about 800 lines of markdown instructions and a 200-line shell script. No dependencies beyond the CLI you already have installed.

### Why markdown

Why not a database? Why not a vector store? Why not a proper memory system with embeddings and retrieval?

Because markdown files are debuggable. You can open them, read them, edit them, version them. When the assistant does something wrong, you can look at the file it read and understand why. When you want to change how it behaves, you edit a text file (or better yet - tell the assistant to fix it). There's no abstraction layer between you and the assistant's state.

There's also a practical reason: LLM context windows are large enough now that reading a few markdown files at session start gives the model everything it needs. The retrieval problem that vector stores solve doesn't exist when your total state fits comfortably in context.

This won't scale to enterprise knowledge management. It's not trying to. It's a personal assistant. Your life fits in a few hundred kilobytes of text.

### The formation problem

There's a deeper question here that I keep thinking about. The reason this assistant works is that I know what good looks like. I've spent years doing the work without AI. I know when a design is drifting off course. I know which action items actually matter. I know what "fits our context" means because I've been wrong about it enough times.

The assistant amplifies that judgment. It doesn't replace it.

But what about someone who's never done the work without AI? If your first experience of project tracking is an AI that does it for you, do you develop the instincts to know when it's tracking the wrong things?

I don't have an answer. But I think the question matters, and I think tools like this one, where the human stays in the loop and the AI's state is transparent and editable, are closer to the right answer than something else.

### Try it

The repo is at [github.com/somsubhro/personal-assistant-kit](https://github.com/somsubhro/personal-assistant-kit). It works with Claude Code and Kiro CLI today, with more CLIs planned.

The best way to understand it is to use it for a week. The first day is interesting. The fifth day is when it starts to feel indispensable.
