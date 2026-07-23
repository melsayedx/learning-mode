<!--
Fill-in template the learning-mode skill opens early in a session and updates at
milestones; the close is a verified final pass. The skill defines the full lifecycle —
including that the TL;DR is written last, after verification. Replace every
<placeholder>. The frontmatter keys are a STABLE CONTRACT: a downstream review or quiz
routine reads them, so keep the key names exactly as written.

Where this goes — one global root, one folder per session so a journal keeps its diagrams
and artifacts beside it:
  ~/.claude/learning-journal/<YYYY-MM-DD>-<topic-slug>/journal.md
  (a project may override the root via a CLAUDE.md line)
-->

---
topic: <short title, e.g. "Stripe webhooks">
domain: <backend | frontend | distributed-systems | databases | algorithms | other>
date: <YYYY-MM-DD>
status: <in-progress | grasped | needs-review>
tags: [<keyword>, <keyword>]
sources: [<url-or-source>, <url-or-source>]
---

## TL;DR
<!-- Filled at the close, after the verify sweep — 2-4 sentences; see the skill's closing pass. -->
<placeholder — filled at close>

## In one line
<What this session was about — one sentence; seeds the frontmatter and the TL;DR.>


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

## Diagrams
<Visuals that helped — preferably the ones I drew. Inline ```mermaid blocks; rendered
images linked relatively with a caption.>

```mermaid
%% <e.g. the flow / state machine I worked out — replace with the real diagram, or delete this section if none>
```

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
