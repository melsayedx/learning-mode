<!--
This is a fill-in template the learning-mode skill instantiates at the close of a
learning session. Copy it to a new dated file (see "Where this goes") and replace every
<placeholder>. The frontmatter keys are a STABLE CONTRACT: a downstream review or quiz
routine reads them, so keep the key names exactly as written.

Where this goes — pick ONE location and keep it consistent so a review routine can scan it:
  Per project:  docs/learning/<YYYY-MM-DD>-<topic-slug>.md
  Global:       ~/learning-journal/<YYYY-MM-DD>-<topic-slug>.md
-->

---
topic: <short title, e.g. "Stripe webhooks">
domain: <backend | frontend | distributed-systems | databases | algorithms | other>
date: <YYYY-MM-DD>
status: <in-progress | grasped | needs-review>
tags: [<keyword>, <keyword>]
sources: [<url-or-source>, <url-or-source>]
---

## In one line
<What this session was about.>

## Why it exists
<The problem it solves — the general-model beat, in one short paragraph.>

## Where else it shows up
<Other real use cases for this concept — the breadth from the mini-wiki entry, so it's
recorded as a general pattern, not a one-off. Mark anything unverified.>

## Naive vs production
<The key contrasts learned. For each: the naive approach, the failure it hits, and the
production fix — with the *why*, and a source for the production claim.>

- <naive approach> → fails because <failure> → production: <fix> — *why*: <reason> [<source>]

## What I built
<What *I* actually implemented or worked through this session — not what Claude wrote.>

## Verified facts (with sources)
<Each fact labeled, so the review routine knows what is checkable truth versus a teaching
device. Mirrors the skill's confidence labels.>

- [verified] <fact> — <primary source>
- [reasoned] <fact derived by reasoning, not stated in a doc> — <the reasoning>
- [unconfirmed] <fact I could not verify> — <what would settle it>
- [analogy] <any mental model used> — *teaching device, not literal*

## In my own words
<My explanation of the core idea, written by me, without looking it up. Gaps here are the
honest signal of what I don't yet understand — leave them visible rather than smoothing
them over.>

## Deep dives
- Taken: <lower-layer topics I went into>
- Deferred: <lower-layer topics I chose to skip — candidates for a later session>

## Open questions
<Anything still unresolved, and what would answer it.>

## Going further — advanced threads
<Advanced questions or topics beyond what this session covered — what to look up to go
deeper into this subject. Not the unresolved questions above; these are the next rung
toward mastery, posed as prompts to investigate rather than answered here.>

- <advanced thread / question>
- <advanced thread / question>

## Edge cases & tests I considered
<The test cases and edge cases I generated for what I built.>

## Retrieval questions for later
<3–5 questions to test myself on in a future session. This is what a spaced-repetition or
quiz routine consumes — phrase them as questions, not answers.>

1. <question>
2. <question>
3. <question>

## Next step
<Where to pick this up next time.>
