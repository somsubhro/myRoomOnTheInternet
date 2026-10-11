---
title: "Is AI replacing your engineering instinct?"
date: 2026-10-05
---

Earlier in the spring, I had asked a [question](https://somsubhro.github.io/myRoomOnTheInternet/2026/04/26/ai-assistant.html#:~:text=But%20what%20about%20someone%20who%E2%80%99s%20never%20done%20the%20work%20without%20AI%3F%20If%20your%20first%20experience%20of%20project%20tracking%20is%20an%20AI%20that%20does%20it%20for%20you%2C%20do%20you%20develop%20the%20instincts%20to%20know%20when%20it%E2%80%99s%20tracking%20the%20wrong%20things%3F)): "But what about someone who's never done the work without AI? If your first experience of < insert > is an AI that does it for you, do you develop the instincts to know when it's [doing] the wrong things?"

I have been observing engineering teams embrace AI native software development during the past year+. Engineers who were delivering software from before this shift are doing well. Engineers without the experience of building software before AI are getting hurt. I am worried about that.

Before going further, I will set a shared context for what I mean by AI native software development. The software delivery loop has not changed. We still start from a customer need, develop it into specs or prompts, package that into code, deploy it, operate it, and let what we learn in production feed back into the next increment. What has changed is that agents (and MCPs, steerings, and other vectors) are now in these loops with humans collaborating to deliver consequential production software.

```
Why am I worried about engineers who started their careers in the AI-native era?
│
└─ Because some of them accept their AI agent's reasoning at face value -
   satisficing without a calibrated bar for when "good enough" is actually good enough.
   │
   └─ Why do they accept it at face value?
      │
      └─ Because they haven't built the expert judgement to interrogate the smell,
         correlation, coherence, correctness checks.
         │
         └─ Why haven't they built that judgement?
            │
            └─ Because judgement is forged by making mistakes, and getting corrected by reality or a senior. 
               The AI-native loop hides the mistakes behind plausible, usually-working output.
               It removes the forcing function that builds judgement.
               │
               └─ Why is that forcing function missing?
                  │
                  └─ Because AI tools are optimized to produce an answer, not to
                     surface the engineer's wrong turns. So error signals that 
                     calibrates judgement never occurs.
                     │
                     └─ Why are the tools optimized that way, and used from day one?
                        │
                        └─ Because employers, in a race for productivity and efficiency,
                           chose AI-native development and the speed it promises.
                           │
                           └─ Why are employers racing for productivity in the first place?
                              │
                              └─ Because the market rewards shipping speed over craft —
                                 investors, competitors, and customers all price velocity.
                                 │
                                 └─ Why does the market reward velocity over craft?
                                    │
                                    └─ Because the cost of low judgement is deferred and
                                       diffuse. Those surface later as incidents, rework, and
                                       fragile systems while the benefit of speed is
                                       immediate and legible.
```

The bill for missing judgement comes due later and typically becomes someone else's problem. And the loop reinforces itself. Lacking judgement → can't evaluate AI tool output → defer to the agent → never build judgement → lacking judgement...
       
```

These aren't the root cause, but they deepen the problem:
```
│
├─ Engineer-side agency
│  └─ Some engineers, given capable AI tools, choose not to learn the
│     traditional way — why invest in the hard path when the easy one ships?
│
├─ The "village's experience" trap
│  └─ Engineers trust the model because it's trained on a huge corpus —
│     a village's collective experience. But a statistical consensus of
│     what's common is not the same as what's correct here, now, for this system.
│
└─ Judgement is invisible on a dashboard
   └─ You can measure PRs merged; you can't easily measure disasters avoided.
      Because judgement is tacit — intuition, not artifact — organizations
      under-invest in what they can't put on a scoreboard.
```
  Is any of this a problem?

  Code writing -> correction loop is shortened
  For someone who is curious, they won't have a problem
  How about we learn on the job while using the AI tools
  Will the underlying models get so much better that we won't need to worry about all this
