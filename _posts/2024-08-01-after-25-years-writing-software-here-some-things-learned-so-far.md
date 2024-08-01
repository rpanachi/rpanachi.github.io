---
layout: post
title: "I've been writing software for the last 25 years. Here some things I learned so far"
date: 2024-08-01 00:00:00
categories: software engineering career
excerpt: My strong opinionated advices for newcomers to software industry
disqus: true
archive: true
redirect_from: /2024/08/01/after-25-years-writing-software-here-some-things-learned-so-far.md/
---

> The computer was born to solve problems that did not exist before.<br/>
> –  Bill Gates


## Understand the problem
Processes are secondary, doesn’t matter if it was Agile, Kanban or Waterfall: you just need to have an exactly understanding of the problem to be solved and plan/organize your work around that.
And, of course, be able to adapt. Changes will occur during the execution and it's fine; Flexibility and adaptability are key to effectively solve the right problem.

Protip: First, solve the problem. Then, write the code. (John Johnson)

## Master at least one programming language
Throughout your career, you'll likely work with a dozen programming languages, including popular ones like Java, JavaScript, Python, Go, and Ruby. However, it's essential to choose one language that balances execution speed and development efficiency and master it. Becoming an expert in one language allows you to write cleaner, more efficient code, reducing the time spent debugging and maintaining projects. The cost of development and maintenance often outweighs execution costs, so prioritize languages that streamline your workflow and enhance productivity. Learn it thoroughly, become a master, and be the go-to person for that language. This can be a game-changer in your career.

Protip: become a contributor to any non-hyped and open source language like [Ruby](https://git.ruby-lang.org/ruby.git) or [Go](https://github.com/golang/go).

## Know to negotiate
Negotiation skills are essential. Whether you're discussing project requirements with stakeholders, negotiating salaries, or resolving conflicts within your team, being able to negotiate effectively is a valuable asset.
Equally important is knowing when and how to say "no". Setting boundaries and managing expectations ensures that you can deliver well-aligned solutions without burning out in the process.

Protip: read [The Art Of Saying NO](https://www.amazon.com/Art-Saying-NO-Reclaim-Granted-ebook/dp/B074LZG7KS) book.

## Use Linux, for real
Mastering Linux is fundamental. Be the admin of your machine. Dive deep into commands, get comfortable with the terminal, and embrace text files. This knowledge will be invaluable throughout your career.
Also, once you embrace the Unix philosophy, you'll start to create more cohesive and reusable code by thinking in a modular and efficient manner. By breaking tasks into simpler, independent components that do one thing well, your solutions become more elegant and maintainable.

Protip: use Linux on your personal computer, all the time.

## Understand the business
Learn the business context of your software and your company. Understanding how your work fits into the bigger picture allows you to make more informed decisions, align your efforts with business goals, and communicate more effectively with non-technical stakeholders.
This insight not only enhances your value as a software engineer but also enables you to deliver more value to your company by ensuring that your work directly supports and advances business objectives.

Protip: read [The Product-Minded Enigneer](https://blog.pragmaticengineer.com/the-product-minded-engineer/) article.

## Write simple and boring code
You don't need to follow any strict or complex practice _by the book_ like Clean Code, SOLID, OOP, DRY-ish, functional, etc. Just focus on writing dumb readable and self-explainatory code that could be easily searched with `grep` or any other search mechanism. To be clear, avoid metaprograming or any technique that make your code less readable. A good test is to show your code to junior developers; if they understand it, you've succeeded.

Embrace boring writing practices and prioritize the most explicit coding style you can. Don't need to be fancy here, just keep in mind to be readable and simple as possible, and you'll be good.

Protip: [embrace the boring code-style](https://www.reddit.com/r/programminghorror/comments/16f5roz/i_embraced_the_boring_codestyle/) and [be proud of it](https://dankim.org/posts/boring-programmer-and-proud-of-it/).

## Build trust and reputation by delivering software
Trust is the most valuable soft skill in your career. If people trust you,
you'll progress more easily. Building trust involves consistently delivering
quality work and communicating effectively.
Earn reputation points by shipping working software. When you deliver reliable software, you gain respect, which you can use to negotiate raises, make significant decisions, and lead and inspire other developers.

Protip: read [Ship it!](https://pragprog.com/titles/prj/ship-it/) book.

## Master a database thoroughly
Understanding a database inside and out is crucial. Master at least one popular DBMS; My recommendation is PostgreSQL. Learn how query analysis works, how data types are managed, the better way to use indexes, and some of the database internals. Know what practices to use and the ones to avoid.
A significant cause of low performance or bottleneck is often related to the database or its improper use. This deep understanding will help you optimize your applications and troubleshoot performance issues, ensuring your software runs smoothly and efficiently.

Protip: read [The Art Of PostgreSQL](https://theartofpostgresql.com/) book.

## Be a specialized generalist
You’re not a React, Java or Go developer; your are a problem solver. Your primary role is to solve problems, which sometimes requires stepping out of your comfort zone. Be proficient in the basics of frontend development, documentation, testing, and other areas. You don't need to be an expert in everything, but a well-rounded skill set will make you a more effective problem solver.

Protip: read [The Pragmatic Programmer](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/) book.

## Don't be a hostage to tools
While it's great to have at disposal the latest and greatest tools, fancy IDEs with integrated AI, visual tools to drag and drop block of codes with ease, never stray too far from raw programming.
Master command-line tools and simple editors like Vim (or Emacs), command-line tools like `curl`, `git` and Unix utilities like `grep` and pipe operator. The ability to work efficiently in the terminal is invaluable.
Simpler tools often offer greater flexibility and control, and being proficient in them can enhance your overall effectiveness.

Protip: [use VIM](https://www.youtube.com/watch?v=wlR5gYd6um0).

## Packaging matters
Delivery isn't just about the code itself. Include concise documentation that outlines what your code does, what it doesn’t, evidence of tests, benchmarks, monitoring instructions, and problem-reporting guidelines. The presentation and packaging of your work are crucial, much like how Apple emphasizes the unboxing experience to stand out from competitors.
Remember, you aren’t the end user of your solution, so consider the experience from their perspective and ensure it is as smooth and informative as possible.

Protip: put a neat [README](https://www.freecodecamp.org/news/how-to-write-a-good-readme-file/) file on your projects.

## Test all the fucking time!
The hallmark of a professional developer is thorough testing. Write tests for your code, embrace Test-Driven Development (TDD), and prioritize automated testing over traditional QA processes.
Continuous testing ensures your code is robust and reduces the likelihood of bugs making it to production.

Protip: [test all the fucking time!](https://www.youtube.com/watch?v=iwUR0kOVNs8)

## Focus on production
Production is where it all matters. All other environments like staging or homologation are a waste of time.
Use feature flags and rolling out strategies: deploy your code disabled on production, enable it, monitor it, and move on to the next task.
Other environments or process can be useful for specific scenarios, but they shouldn't detract from the primary goal of delivering reliable software to production.

Protip: read [Testing in Production, the safe way](https://copyconstruct.medium.com/testing-in-production-the-safe-way-18ca102d0ef1) article.

---

In my 25 years of experience, my method has remained largely the same. My techniques have improved, and my toolbelt has grown, but my approach to problem-solving is consistent: identify the problem, set your goal, find solutions, test them, choose the best one for your context, make it work, and ship it.

Enjoy o/
