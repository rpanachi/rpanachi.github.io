---
layout: post
title: "I've been writing software for the last 25 years. Here are a few more things I've learned so far"
date: 2025-04-14 00:00:00
categories: [software engineering, career]
excerpt: What the docs don’t teach you, 25 years will
disqus: true
archive: true
---

> The code you write makes you a programmer. The code you delete makes you a good one. The code you don’t have to write makes you a great one.<br/>
> –  Unknown

## But first, some context

I'm a self-taught generalist (aka full-stack) software engineer experienced mainly in web applications, having a strong focus on pragmatism and problem solving. Over the last 25 years, I’ve worked in small startups, mid-sized companies, and large enterprises—mostly in Linux environments, using open-source languages and tools. Along the way, I’ve also had some experience leading and managing teams.

The lessons below come from years of seeing what works and what doesn’t. Some are the result of success, others from hitting the wall—hard.

Read part 1 here: [I've been writing software for the last 25 years. Here some things I learned so far]({% post_url 2024-08-01-after-25-years-writing-software-here-some-things-learned-so-far %})

---

## Building and Designing Software

### Automate your workflow  
Automate everything you can—project setup, configuring local environments, creating test data with scripts, etc. If you do it more than once, automate it. Your future self (and teammates) will thank you.

### Applications should run locally  
If your app can’t run on a dev machine, you’re doing it wrong. Make local setup simple—native or Docker, doesn’t matter. Devs need full control to reproduce problems, debug, and test... locally. Include scripts to generate sample data, simulate API calls, and preload images or documents. These tools are part of the product too.

### Design your code to fail  
Don’t trust inputs. Add logs where it matters. Write checkers, guard clauses, notifications for edge case scenarios. Expect things to break—and make it easy to recover.

### Track requests and responses  
Log everything used in third-party integrations—requests, responses, and payloads. You *will* need it for debugging and audits.

### Don’t blindly trust your data
Users send broken data. Bugs introduce garbage into database. Validate everything—on input, on output, on storage. Your system is only as good as the data flowing through it.

### Don’t ignore idempotency  
If something fails, you should be able to retry it without causing damage. Idempotency is your friend. Don’t ship production code without it.

### Be explicit  
Use `"mobile"` instead of `3` to describe enums. Name things clearly: `has_balance`, `requested_amount`. Never assume something is “obvious.” It never is.  
More on this here: [Greppability: A code metric](https://morizbuesing.com/blog/greppability-code-metric/)

### Testing > Debugging  
Relying only on debugging tools could lead you to chase problems after they happen. Testing (especially TDD) helps you move in small, safe steps. You define expectations first, then work towards meeting them. It’s like climbing a mountain one secure foothold at a time.

### Avoid unnecessary frameworks  
The web is just HTTP—receive a request, send a response. That hasn’t changed in decades. You don’t need heavy frameworks to handle 90% of what you build. Stick with boring, battle-tested tools.

### Monoliths aren’t always bad. Microservices aren’t always good.  
Don’t split your system into pieces unless you have a *really* good reason. Scale only when you must. Premature complexity is the real bottleneck.

### Worse is better  
Keep it simple. Forget hexagonal architecture and all the over-engineered “best practices” that solve problems you don’t have. [Worse is better](https://www.dreamsongs.com/WorseIsBetter.html)—and often faster, clearer, and easier to maintain.

### Don’t try to make it perfect. Make it adaptable.  
There’s no silver bullet. Instead of aiming for the perfect solution, aim for something adjustable, configurable, flexible, and easy to change when needed. Prioritize adaptability over perfection.

---

## Shipping and Maintaining Software

### You won’t get it right the first time  
No matter the architecture, design, or framework, your first attempt won’t be perfect. You’ll need to iterate to make it truly work. So don’t overthink or over-plan. Ship something functional as soon as possible and let feedback guide you.

### Monitor and observe your software  
Know something’s broken before your users do. Instrument your systems. Set alerts. Pay attention to the data.

### Always have a Plan B  
Have redundancy. Keep backups. Make sure someone else can step in if needed. Single points of failure—whether in infrastructure or people—are liabilities.

### Care about the UX  
Interfaces aren’t just forms anymore. Users talk to systems with voice and text (Alexa, ChatGPT, etc.). If your signup form has 12 fields, they’ll probably quit halfway through.

---

## Working with Teams and People

### Estimates are bad. Manage expectations instead.  
[Software estimates have never worked—and never will](https://world.hey.com/dhh/software-estimates-have-never-worked-and-never-will-a41a9c71). Communicate uncertainty. Share tradeoffs. Set expectations early and often.

### Extreme visibility  
Keep your teammates, managers, and stakeholders aware of what you're doing, your progress, and what’s blocking you. It builds trust and reduces surprises.

### Don’t micromanage  
If you don’t trust your team, why are they on the team? Focus on outcomes: what's the impact on the product, team, or company? What’s blocking them? How can you help? Let people figure things out—it’s how they grow.

### Communicate decisions  
Why this framework instead of that one? Why this database, this design, this pattern? Write it down. Share it. It prevents repeat mistakes and helps others understand the “why.”

### Share what you know  
Sharing accelerates your career more than anything else. Teach. Present. Write. Share what you learn, what you build, and what you discover. The more you share, the more you grow.

### Don’t postpone obvious decisions  
You’re going to make a lot of decisions. Some are big, others are trivial. Don’t waste time overthinking the obvious ones. Make the call, communicate it, and spend your energy where it really matters.

### Reject dogma  
Take principles like the ones in this post as guidance—not gospel. Don’t follow advice blindly just because someone said it’s best practice. Think. Understand the context. Apply what makes sense.

---

##  Wrapping up

After all these years, one thing is clear: writing software isn’t just about code. It’s about people, communication, trade-offs, and learning to live with imperfection. You won’t get everything right. You’ll ship things that break. You’ll rethink decisions. That’s normal. That’s healthy.

What matters is staying curious, being honest about what works (and what doesn’t), and helping those around you grow. The lessons in this post aren’t universal truths—they’re just what I’ve picked up from the trenches. Take what’s useful, challenge what isn’t, and keep moving forward.

And above all: keep it simple, keep it working, and keep learning.

Enjoy o/
