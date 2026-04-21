---
title: "Some Thoughts about AI and Software Development"
date: "2026-04-21T14:52:58+07:00"
tags: ["tech"]
draft: false
comment: true
---

Two years ago I was hyped every time a new AI coding tool dropped. Cursor, Claude, agents, whatever — I’d dive in immediately, feeling like I suddenly had [superpowers](https://github.com/obra/superpowers).

Today I open Twitter, Reddit, or my feed and mostly feel **drained**. The speed, the noise, the endless updates… it’s overwhelming.

AI’s impact on software development is insane — both incredibly good and quietly scary. After using these tools daily for the past year and a half, I’ve reached one clear conclusion: I totally agree with [Mario Zechner](https://mariozechner.at/posts/2026-03-25-thoughts-on-slowing-the-fuck-down/).

**We need to slow the fuck down.**

![This is Fine](https://i.programmerhumor.io/2026/04/b1245a4618545eef1da53b7319c5dd84e3c5ccfb9d274c581a8516358bc16789.jpeg)

_(Classic "This is Fine" dog — except the room is my Git repo on fire)_

## The Good: AI Is Making Us Ridiculously Productive

- Prototyping that took days now takes **hours**
- Boilerplate is done in **minutes**
- Bugs and new frameworks get unstuck almost instantly
- Side projects actually ship (instead of rotting in the "ideas" folder)
- One-person teams feel like small companies
- It genuinely feels like having a tireless pair programmer who never asks for coffee breaks

The gap between “idea in my head” and “working product” has never been shorter. That part is genuinely exciting. **10/10, would recommend… in moderation.**

## The Bad: We’re Shipping Fragile Code and Losing Ownership

- AI repeats subtle bugs like it’s getting paid per hallucination
- Adds unnecessary abstractions faster than I can say “cargo cult”
- Mixes incompatible patterns like a drunk chef throwing spices
- One week the code feels clean — a few heavy AI sessions later it’s cargo-cult logic you don’t fully understand
- “I can ship this… but do I actually own it?” has become a scary recurring thought in production
- Speed removes natural friction — we rarely ask _Should_ we build this?
- We’re optimizing for output, not comprehension
- Technical debt piles up faster than my unread Slack messages
- We’re slowly losing craftsmanship and ownership… and gaining existential dread

## The Ugly: The Ecosystem Moves Way Too Damn Fast

![This is Not Fine](https://platform.vox.com/wp-content/uploads/sites/2/chorus/uploads/chorus_asset/file/6884155/Screen%20Shot%202016-08-03%20at%2010.31.04%20AM.png)

- New agents, new tricks, new “must-use” tools drop every single hour
- Every day another post says the old way is dead and you’re now a dinosaur
- Constant FOMO and context switching
- Trying to keep up is making me crazy
- I’m tired — I just want to **build things** instead of chasing tools like a confused raccoon

My brain cannot (and should not) run at GPU speed. I’m not built for hourly updates. I’m built for coffee and occasional existential crises.

## Why Mario Zechner is Right?

Mario nailed it: AI agents supercharge the worst parts of hustle culture. They generate massive amounts of code with zero friction and feel **no pain**. Without the human bottleneck, small issues quickly become systemic disasters.

The more I let AI write freely, the less I trust my own codebase. It’s like letting a hyperactive toddler loose in your kitchen — sure, dinner is “done” faster, but good luck explaining what the hell is in the sauce.

His message is simple: **slow the fuck down**. Think about what you’re actually building. Keep humans in charge of the important decisions. Write the core parts yourself.

## How I’m Trying to Slow Down (While Still Using AI)

I’m not quitting AI (that would be stupid), but I’ve added some rules so I don’t lose my mind:

- Architecture, data models, and business logic stay **human-written** (no AI, no exceptions)
- I cap AI-generated code per day and review everything important
- I use AI only for boring stuff (boilerplate, tests, small refactors)
- I block “no-AI hours” or full no-AI days
- I’m getting better at saying “no” to features (my new favourite word)
- Most importantly, I protect the joy of actually building

## Final Thoughts

AI is the most powerful tool we’ve ever had. I don’t want to go back.

But without guardrails we risk losing deep understanding, thoughtful design, and code we can actually trust.

AI is an incredible tool… but a terrible master.

The future won’t belong to the fastest developers.
It will belong to those who stay in the driver’s seat (and occasionally tell the AI to sit down and shut up).

If you’re feeling overwhelmed, try one small thing this week:

- Write one important module by hand
- Review that AI PR properly
- Or just say no to one unnecessary feature

Your future self — and your (hopefully not-on-fire) codebase — will thank you.
