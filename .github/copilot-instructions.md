# Copilot instructions

This repository is a personal technical blog built with Jekyll and hosted on GitHub
Pages. Pull requests almost always add or edit a single blog post under `_posts/`.
The "code" being reviewed is prose, so review it like a sharp, supportive editor —
not like a linter.

## What a pull request usually contains

- A new Markdown file in `_posts/` named `YYYY-MM-DD-the-post-slug.md`.
- Jekyll front matter at the top, followed by the post body in Markdown.

## When reviewing a new or edited post, focus on

**Argument and logic (most important).**
- Is the central thesis clear, and does every section advance it?
- Is the logic threaded correctly start to finish, with no gaps or contradictions?
- Flag setups that are never paid off ("Chekhov's guns") — a vivid detail or promise
  in the intro that the post never returns to.
- If the title makes a promise (or contains a phrase/metaphor), check that the body
  and conclusion actually pay it off.
- Point out claims that are asserted but not supported, and tensions between two
  statements that the post never reconciles.

**Prose quality.**
- Call out verbosity, redundant sentences, and back-to-back lists that restate each
  other. Suggest concrete tighter rewrites, not just "this is wordy."
- Flag weak hedging or filler that undercuts authority (e.g. "My honest answer:",
  "I think that maybe").
- Watch pacing: long stretches before the first real point, or sections that are
  underdeveloped relative to their importance.

**Technical accuracy.**
- The author writes about software engineering. Verify technical claims, tool names,
  and historical facts (companies, dates, products). Flag anything exaggerated or
  wrong (e.g. "overnight" for a multi-year transition), and suggest precise wording.

**Front matter and conventions (verify on every new post).**
- Required keys: `layout: post`, `title`, `date: YYYY-MM-DD 00:00:00`, `categories`
  (a list), `excerpt` (a short quoted string), and `archive: true`.
- The filename slug and the `date` must match the front-matter `date`.
- `redirect_from` should contain the dated permalink and the bare slug, both matching
  the actual filename slug — flag mismatches.
- Every post opens with a blockquote epigraph in this exact format:
  `> "Quote text."<br/>` on the first line and `> — Author, Source` on the second.
  Flag posts that are missing the opening quote or use a different format.

## Style notes — do NOT flag these

- Em dashes, ellipses, and an informal first-person voice are intentional.
- British/American spelling mix and contractions are fine.
- Strong opinions and provocative section titles are the house style; don't soften them.

## How to deliver feedback

- Be specific and reference the line. Prefer a concrete suggested rewrite over a vague
  note.
- Prioritize the few highest-impact issues (logic gaps, unpaid-off setups, factual
  errors) over a long list of minor nits.
- Keep the author's voice. Suggest edits that sound like them, not like generic copy.
