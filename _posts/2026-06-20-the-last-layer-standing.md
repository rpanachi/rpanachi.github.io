---
layout: post
title: The Last Layer Standing
date: 2026-06-20 00:00:00
categories: [ai, software engineering, career, thoughts]
excerpt: "The technology never kills you. Standing still does."
archive: true
redirect_from:
  - /2026/06/20/the-last-layer-standing/
  - /the-last-layer-standing
---

> "It is unsinkable. God himself could not sink this ship."<br/>
> — Cal Hockley, *Titanic* (1997)

I'm somewhere in the second half of my career. If things go as planned, I'll retire from this industry in the next ten or fifteen years.

Lately I keep wondering what software will even look like by then. AI has already changed everything — and it's only getting started. So what comes after?

When I want to imagine the future, I look at my past. I've been doing this long enough to have watched the industry tear itself down and rebuild from scratch more than once. The details change every time. The pattern doesn't.

## The industry never sits still

When I started writing code, the internet didn't exist yet — not for most of us, anyway. The industry ran on desktop software: office suites, productivity tools, business systems, ERPs, CRMs. You shipped on floppies and CDs, and a "deploy" meant someone installing it by hand.

And then came the internet. Everything changed. The client-server model, a new programming paradigm, a new kind of usability, an entire category of problems nobody had solved yet. But the same internet that broke everything also connected us: suddenly we could share ideas, read each other's articles, steal each other's tricks, and ask strangers for help. We learned faster because we learned together.

Then infrastructure became the bottleneck. Running software meant buying racks of expensive hardware and hiring a dedicated team to babysit it. It was slow, it was costly, and it didn't scale.

And then came AWS. Everything changed again. That pile of money companies had sunk into hardware turned into scrap almost overnight. A small team could suddenly rent world-class infrastructure for pocket change, and a whole class of scalability problems vanished behind a single click.

And then came the SaaS economy. Why build authentication, payments, search, or messaging yourself when you could rent each one and wire them together in an afternoon? Whole categories of software that used to take a team a year shrank to a few lines of integration and a monthly invoice. Building started to look less like engineering from scratch and more like assembling parts someone else maintained.

Notice the rhythm. Every decade or so, something arrives that resets the board. Each time, the skills that felt essential become optional, and the ones that felt optional become survival.

## And then came AI

Before, learning to do something new took weeks. You picked up a new language or framework, read a mountain of documentation, fumbled through examples, broke things, fixed them, and two or three weeks later you had that fancy integration working. Today I describe it in a paragraph and have a working draft in thirty minutes.

That speed-up isn't a one-off. It shows up everywhere:

- A CRUD app that used to need a team of five — backend, frontend, database, a little DevOps, someone to glue it all together — can now be scaffolded by one person in an afternoon.
- Boilerplate that took a day writes itself in seconds: models, migrations, endpoints, forms, validation, tests.
- The documentation safari — twelve browser tabs and a pilgrimage to Stack Overflow — collapses into a single question.
- The glue code between two APIs that nobody wants to write gets written instantly.
- Translating a service from one language to another, generating tests for legacy code, drafting the first version of a CI/CD pipeline — all of it, faster and cheaper than ever.

The economics are simply different now. Work that justified a team and a quarter can be done by one experienced person in a sprint.

## Software is becoming a commodity

Follow the trend to its end and the conclusion is uncomfortable but hard to dodge: software development can be almost entirely automated. Not assisted — automated. Each wave so far abstracted away a layer we used to sweat over, and the last layer standing is the code itself.

We're already partway there. A non-technical person can ship a real feature just by describing what they want — but only because a developer built the machine that makes it possible: the architecture, the infrastructure, the security model, the guardrails. Someone defined the rules of the game so others could play it without knowing them. Soon even that layer gets abstracted away. The same way AWS turned hardware into something you rent by the minute, building software becomes a feature of a platform — you describe the rules, and it writes, tests, and runs the code underneath. If that sounds far-fetched, remember that "rent a supercomputer with a credit card" sounded far-fetched too.

And no company gets to sit this one out. Every one of those waves left bodies on the path — outfits that owned their era and were too comfortable, too slow, or too afraid to follow it into the next. Borland owned developer tools and dissolved into a footnote; Kodak invented the digital camera and then buried it. The rule never changes: when the ground moves, you move with it or you become history. This wave won't spare anyone who stands still either.

## So, are developers history?

That's the companies. But what about us — the people who actually write the software? If building really can be automated, is the developer, as we know the role, about to become history?

My honest answer: the *code-typist* version of the job is. The person whose value was mostly turning tickets into syntax is in for a hard decade. That part of the work is exactly what the machine is best at.

But "developer" was never really about typing. The new developer is the person who uses creativity, judgment, and tools — AI included — to create value and solve real problems. Does that person need to be deeply technical? Well… *it depends* (says the senior developer).

Because here's the thing the hype skips: there is an ocean of legacy out there. Gigantic codebases, decades old, quietly running the world's banks, hospitals, logistics, and governments. Nobody is going to throw all of that away and migrate to some shiny describe-it-and-it-appears platform. Some of them *can't* — when the software *is* the business, you don't get to start over.

Those systems need people who actually understand them — experienced, technical, stubborn people who can read a mess, hold the whole thing in their head, and make the call.

So no — skilled people don't disappear. The seats reserved for genuinely capable professionals are never empty, because businesses still need humans to run them. What disappears is the comfortable middle: the role that was only ever about producing code.

## What AI still can't do

It's worth being concrete about why those seats stay filled. For all its speed, AI still can't do the things the job actually rewards.

It doesn't know *what* to build. Hand it a vague, contradictory, half-formed idea — which is what real requirements always are — and it can't tell you which parts matter, which are wrong, and which the stakeholder forgot to mention. That's still a human problem.

It doesn't understand *why*. It has no model of your business, your customers, your constraints, or the three years of decisions that explain why the code looks the way it does. It optimizes the question you asked, not the one you should have asked.

It drowns in real legacy. A clean greenfield CRUD is one thing. A fifteen-year-old codebase full of implicit rules, undocumented quirks, and load-bearing hacks is another. AI will confidently "fix" something and quietly break three things you didn't know were connected.

It produces plausible-but-wrong code. The output always *looks* right. Telling the difference between code that works and code that merely compiles still takes someone who actually understands the system. That someone is you.

And it doesn't carry responsibility. When the deploy takes down production at 2 a.m., the AI doesn't get paged. It doesn't make trade-offs it can be held to, doesn't navigate the politics of a reorg, doesn't decide what risk is acceptable. Accountability has no API.

This is the gap. And it's exactly where the value moved.

## So here's my plan

Enough analysis. Here's the personal part — what I'm actually going to do for the next ten or fifteen years to make sure I'm still holding a seat when the music stops.

**Move up the stack, deliberately.** The machine is coming for the typing, so I'm going to do less of it. I'll put my hours where AI is weakest: architecture, system design, the trade-offs and the calls that carry consequences. Deciding *what* to build and *why* is the job now. The *how* is becoming a prompt.

**Use AI harder than anyone around me.** I'm not going to fight the tide — I'm going to surf it. I treat these tools the way I once treated the compiler and the cloud: something to master until it's an extension of my hands. The aim is blunt — be the person who ships what used to take five, because I turned AI into leverage instead of treating it as a threat.

**Take the hard, ugly, high-stakes systems.** Clean greenfield projects are exactly what gets commoditized first. So I'll go the other way — toward the gnarly legacy, the mission-critical, the can't-just-restart-it systems where someone has to understand the whole machine and answer for it when it breaks. That seat doesn't vanish. I intend to be in it.

**Stay glued to the outcome, not the code.** I'm paid to solve problems and create value, not to emit lines. So I'll keep moving closer to the business, the customer, and the money — the context no model can infer from a ticket. Whoever understands the *problem* best is the last to be automated away.

None of this is guaranteed. Nothing is, anymore. But a plan beats hoping the tornado skips your house.

## Conclusion

Strip away the specifics, and the second half comes down to four things I try not to forget:

**AI is a tool.** A spectacularly powerful, genuinely impressive one — but a tool, in the same lineage as the compiler, the cloud, and the framework. It isn't magic, and it isn't the end of the story. It's the next chapter.

**Tools exist so that someone can make something.** Nobody remembers the hammer; they remember the house. The value was never in the instrument — it was in what you built and the problem you solved. Stay focused on the outcome, not the tool of the moment.

**Every tool has to be mastered.** The developers who thrived through each shift weren't the ones with the most to protect; they were the ones who learned the new thing fastest. The ones who refused — who insisted the old way was the only *real* way — became the bodies on the path. Keep learning, or become a case study.

**Read the signals and adapt.** Watch the environment. Notice what's becoming a commodity and move before the floor you're standing on disappears. Relevance isn't a position you hold; it's something you keep earning.

I've watched this industry demolish and rebuild itself several times now, and the lesson is always the same. The technology that arrives isn't what kills you. Standing still is.

I've had to reinvent how I work every decade. Here we go again.
