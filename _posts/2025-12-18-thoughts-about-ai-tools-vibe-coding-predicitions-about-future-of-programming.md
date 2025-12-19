---
layout: post
title: My thoughts about AI tools, vibe coding, and some predictions about the future of programming
date: 2025-12-18 00:00:00
categories: [ai, vibe coding, thoughts]
excerpt: AI is not a replacement for thinking. It’s an advanced code generator
disqus: true
archive: true
---

> "AI tools are great if you know what you’re doing."
> — A tired developer


## “Magical tools” aren’t new

I started programming about 25 years ago. Self-taught. A lot of trial and error. When I was lucky, I had access to books. When I wasn’t, I explored every corner of the *Help* menu of Delphi. I learned by breaking things, fixing them, and slowly understanding how everything fit together. The learning curve was high.

At that time, the “best" tools were the ones that promised productivity through visual components: drag-and-drop forms, buttons, grids, data sources, and magically “connected” events. The pitch was always the same: *“You don’t need to write code. Just drag components and connect them.”*

Delphi, Visual Basic, PowerBuilder, and similar tools built entire empires on that promise.

Old-school developers hated it. They said it wasn’t real programming. That it was cheating. That people using those tools didn’t really understand what was happening underneath.

The market didn’t care.

Companies shipped software faster. Businesses made money. And those tools dominated for years. Eventually, they shaped the industry, influenced modern frameworks, and became the foundation of what we now call “rapid application development”.

AI tools feel different — but they really aren’t. They are just the next iteration of “magical tools”.


## AI tools are great if you know what you are doing

If you just ask an AI to *solve a problem for you*, you’re doing it wrong.

AI is not a replacement for thinking. It’s an **advanced code generator**.

The quality of the output is directly proportional to the quality of your prompt — which, in practice, means the quality of your understanding of the problem, the codebase, and the architecture.

A bad prompt looks like this:

> “Add a button that lists people on this page.”

In the best case scenario, the AI will generate something that works. In the worst case, it will dump a pile of spaghetti code that ignores conventions, architecture, separation of concerns, permissions, tests, and long-term maintainability.

A *good* prompt looks like this:

> “Add a button that lists people on this page.  
> Use the `MyButton` component and centralize all fetching/mutation logic into the `api.ts` file.  
> Get the list of people from the `ContactsService` class, but keep all filtering logic inside `PeopleController`.  
> Make sure to verify user permissions to create a new `Person` using the `User.canExecute("action")` method.  
> Write an integration test covering this usage scenario.”

Now you are in control.

You already know *what* needs to be done. You know *where* the logic belongs. You know *how* your system should evolve. You’re just delegating the mechanical part — writing code — to the AI.

That’s not laziness. That’s leverage.


## “You’ll become dependent and forget how to code”

This argument keeps coming back. And yes — it’s partially true.

When you don’t have to worry about syntax, language quirks, or framework boilerplate, you naturally forget some details. You might not remember the exact syntax of a `for` loop in Go, or how Ruby blocks behave in edge cases.

But here’s the uncomfortable truth: **languages are commodities now**.

It really doesn’t matter that much if your backend is written in Go, JavaScript, Ruby, or Python. What matters is that you understand:
- logic
- data flow
- architecture
- patterns
- trade-offs
- constraints
- failure modes

If you have that, the AI can handle the syntax.

We already went through this transition before. We stopped memorizing assembly when high-level languages appeared. We stopped writing raw SQL everywhere when ORMs became popular. We stopped managing servers by hand when cloud platforms emerged.

This is just the next step.


## Junior developers will disappear

AI tools are the new junior developers.

A mid/senior developer, using AI effectively, can be dramatically more productive than a senior leading and coordinating a team of juniors. There’s no onboarding, no context switching, no waiting for pull requests, no repeated explanations of architecture decisions.

More importantly:
- AI doesn’t forget conventions
- AI can instantly generate alternatives  
- AI can refactor relentlessly  
- AI can write tests, docs, and examples on demand  

For companies, the cost/benefit equation changes completely. Instead of hiring multiple juniors and hoping they grow, a smaller team of experienced developers with AI assistance can deliver faster and with more consistency.

That doesn’t mean *people* disappear — it means the traditional entry point to the industry will change radically.


## Generalists will rule

For the vast majority of companies, you no longer need deep specialists in frontend, backend, and database just to build and run a product.

A single developer, armed with AI tools, can:
- design APIs
- build frontend interfaces
- model databases, write migrations
- configure infrastructure
- handle authentication and security practices
- set up CI/CD
- deploy and monitor applications

This is not science fiction. It’s already happening.

Of course, I’m talking about **common software** — the kind that runs most businesses. Not rocket science. Not medical devices. Not highly specialized domains.

If your company operates in a niche area that requires deep domain knowledge, specialists will still be essential — not because of coding itself, but because of the *knowledge* required to make correct decisions.

But for most cases, generalists who understand the whole system will be far more valuable than narrow specialists.


## Software development will still need people

Despite all the hype, software development won’t disappear.

What *will* change is the role of the developer.

Developers will increasingly become the bridge between business intent and technical execution. AI lowers the learning curve, but it doesn’t remove the complexity. Someone still needs to:
- understand what the business actually wants
- translate vague ideas into concrete systems
- make trade-offs
- define boundaries
- ensure quality
- manage risk

The “new programmers” will orchestrate execution. They’ll use AI, vibe coding, text-to-code, diagrams, prompts, and automation to transform business needs into working software — faster than ever before.


## Conclusion

Every generation of developers believes their tools are the last ones that require “real skill”. And every generation is wrong.

We’ve always built abstractions to move faster. AI is just the most powerful abstraction we’ve seen so far.

The core skill has never been typing code. It has always been **thinking clearly**, **designing systems**, and **solving real problems**.

AI doesn’t replace that. It amplifies it.

If you don’t know what you’re doing, AI will happily help you create a mess — faster than ever.  
If you *do* know what you’re doing, AI becomes a force multiplier.

And that’s why I truly believe this:

**AI tools are great if you know what you’re doing.**


### Personal notes

1. This text was partially written using AI. The content is mine, thoughts, examples, topics, and logical structure. I just asked IA to transform my notes into a blog post format. This is why I kept the "AI dash" on it.
2. If you're a junior developer, don't focus on just writing code; you should learn the concepts, software patterns, best practices, and soft skills. Achievements and deeds have more weight on your profile than technical skills purely.
