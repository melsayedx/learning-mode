# Learning Mode

[![MIT License](https://img.shields.io/badge/license-MIT-111827.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude_Code-plugin-D97757.svg)](https://code.claude.com/docs/en/plugins)

A Claude Code skill (plus a companion agent) that turns Claude into a **guide, not an autocomplete**. When you're learning an unfamiliar technology, Learning Mode makes Claude explain, question, and review, but never write the implementation for you. The understanding ends up in your head instead of in pasted code.

The repository targets Claude Code and works across backend, frontend, distributed systems, databases, and algorithms. Its behavioral core is a portable `SKILL.md`.

## Install

In Claude Code:

```text
/plugin marketplace add melsayedx/learning-mode
/plugin install learning-mode@learning-mode-marketplace
/reload-plugins
```

Then ask naturally:

> Redis Streams are new to me. Help me build a worker, but make sure I understand every decision.

## What it feels like

Shortened illustrative exchange:

> **You:** I need Stripe webhooks, but I've never built one.
>
> **Claude:** Before we name the mechanism, how would your server learn that a payment succeeded on Stripe's server?
>
> **You:** Poll the API every few seconds.
>
> **Claude:** At 10,000 customers, what happens to request volume, latency, and rate limits?

The failure motivates the concept. You write the handler. Claude climbs a hint ladder only when you get stuck, verifies technical claims against primary sources, and records what you learned so a later session can resume.

## How this differs from Anthropic's learning output style

[Anthropic's `learning-output-style`](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style) still implements most of a task and asks the user for focused 5–10-line contributions at meaningful decision points. Learning Mode is stricter and activates only when you signal learning intent.

|                                       | Anthropic learning output style          | Learning Mode                                      |
| ------------------------------------- | ---------------------------------------- | -------------------------------------------------- |
| Who writes the solution-bearing code? | Claude, with selected user contributions | The learner                                        |
| Activation                            | Every session while enabled              | Only when the user wants to learn                  |
| Technical claims                      | Educational explanation                  | Primary-source verification with confidence labels |
| Continuity                            | Current conversation                     | Structured journal and cross-session resume        |

## What it's for

- **Learning a new tool by building with it.** A library, framework, protocol, or service you haven't used: you implement it yourself against a real problem, hint by hint — and you end with the production-hardened version, not the tutorial version, with every divergence from the naive approach explained and sourced.
- **Understanding pure concepts.** Topics with nothing to type — object storage, consistency models, how an algorithm really works. The same discipline applies to thinking: you derive, predict, and explain back; Claude elicits and reviews instead of lecturing.
- **Deepening fundamentals you half-know.** The failure-first arc exposes the edges of a mental model fast: you find out precisely where your version of "how this works" stops being true.

## Why this instead of "act as a tutor and don't give me the answer"?

You can prompt any model into tutoring. Four things make this hold up in practice where ad-hoc prompts don't:

1. **A behavioral contract that doesn't decay.** One-off instructions drift as a chat grows — the model "helps" a little more each time you struggle. This skill reloads its rules every session and names the rationalizations it must decline: _"just write it for me, just this once"_, _"I'm in a hurry"_, _"show me an example and I'll adapt it"_, _"I already understand it, just generate it"_. Its line for what Claude may write is one sentence: **if you could paste what Claude wrote and have the task done, it wrote too much.**
2. **Verification instead of confident guessing.** Factual claims about an API, protocol, or production practice are grounded in primary sources before they're stated — official docs, specs, canonical texts, real industry code — with explicit labels separating _verified_ from _reasoned_ from _analogy_. A read-only **doc-verifier** subagent runs the lookups in its own forked context, so your session stays clean and the verdict comes back cited.
3. **Memory that survives the session.** Every session keeps a structured journal as it goes — updated at milestones, like a write-ahead log — and closes with a final pass that formats the entry and verifies its claims. A crashed, compacted, or abandoned session resumes in a fresh conversation from its `in-progress` entry, so you never re-explain what you were learning.
4. **Pedagogy encoded from learning science, not vibes.** The session shape implements specific, well-evidenced techniques — below — rather than a generic "be Socratic" instruction.

Against the broader alternatives: a tutorial is consumption — fluent to read, gone in a week. Flashcards retain facts you already understand. Learning Mode is for _constructing_ understanding; its journals then feed whatever retention routine you run on top.

## The learning science inside

- **Productive failure.** You attempt the obvious naive solution first and feel exactly where it breaks; the technology is introduced _by_ that failure, so the "why" is earned rather than lectured.
- **Guided inquiry, not pure discovery.** Unguided struggle is well documented to fail for novices, so a four-rung hint ladder climbs from "what have you tried?" to a near-complete hint — one rung at a time, never skipping to the bottom.
- **The worked-example floor (expertise reversal).** True novices learn faster from studying a worked solution than from open-ended struggle — so when the ladder bottoms out, Claude walks one fully worked _analogous_ problem, never the target, then returns you to yours.
- **Generation and retrieval practice.** You write every line that counts and you design the tests; each journal stores retrieval questions for later self-testing, because testing beats re-reading for retention.
- **Self-explanation.** Every arc ends with you explaining the idea back in your own words — and the gaps in that explanation are the honest signal of what isn't understood yet.
- **Consolidation.** Once a concept is earned, it's crystallized into a titled mini-wiki entry — a precise definition, why it exists tied to the failure you just felt, and where else it shows up — so the aha resolves into something you keep, whether or not you reached the concept yourself.
- **Fading scaffolding.** Support tracks demonstrated competence, not how much you ask: more hints early, longer struggle once you're self-correcting.
- **Visual construction.** When an idea is clearer as a picture — a request lifecycle, a state machine, a memory layout — Learning Mode asks _you_ to sketch it first and reviews your version, rendering a clean one itself only as the novice floor or for a reference diagram. Diagrams land in the journal as inline Mermaid (no image files to lose), via the dual-coding principle that pairing a visual with words beats words alone.

## What's included

```
plugins/learning-mode/
├── skills/learning-mode/
│   ├── SKILL.md                      # the behavioral core
│   ├── references/
│   │   ├── verification.md           # the source-grounding method (also used by the agent)
│   │   └── examples.md               # a worked failure-first, naive-vs-production arc (Stripe webhooks)
│   └── assets/
│       └── journal-template.md       # the per-session learning-journal template
└── agents/
    └── doc-verifier.md               # read-only subagent that grounds facts in a forked context
```

The **doc-verifier** agent runs documentation lookups and probes in its own context so your main session stays clean, and returns a compact, cited verdict. It is **read-only** (no Edit/Write tools) and uses only `Bash, WebSearch, WebFetch, Read, Grep, Glob`.

## Requirements & companions

- **Claude Code** (for the plugin and companion agent).
- **Context7 `ctx7` CLI** (recommended) — the doc-verifier prefers it for version-specific library docs. Install with `npx ctx7 setup`. If you use Context7 as an MCP server instead of the CLI, add its tool to the agent's `tools` line.
- **Complements [obra/superpowers](https://github.com/obra/superpowers)** — Learning Mode is independent, but its trigger is written so it also engages when a superpowers brainstorming/planning/execution step touches a technology you've flagged as unfamiliar.
- **Diagrams (optional).** Visuals default to inline Mermaid, which renders in GitHub and most Markdown viewers with nothing installed. For live preview or rendered SVG/PNG, connect a Mermaid MCP (the official remote server at `https://mcp.mermaidchart.com/mcp`, or a Claude-Code-native one with live reload) — or install the Mermaid CLI (`npm i -g @mermaid-js/mermaid-cli`). For freeform conceptual sketches, an Excalidraw MCP fits better than Mermaid. The skill prompts you to connect one only if a visual needs it; it never blocks a lesson on a tool.

## Manual install

To install the skill without the plugin marketplace, copy the skill folder into your user skills directory:

```bash
cp -r plugins/learning-mode/skills/learning-mode ~/.claude/skills/learning-mode
```

To also get the verifier agent, copy it into your user agents directory:

```bash
cp plugins/learning-mode/agents/doc-verifier.md ~/.claude/agents/doc-verifier.md
```

Restart Claude Code to load the agent.

## Where journals go

The skill writes each session to `~/.claude/learning-journal/<date>-<topic>/journal.md`. A session folder keeps its journal, diagrams, and rendered artifacts together without putting them in a project repository. If the skill is unsure you want a journal, it asks. An entry left `in-progress` is the resume checkpoint a fresh session restores from; at close, its TL;DR is written only after the factual verification pass.

If you want a specific project to keep its own journals instead, say so in that project's `CLAUDE.md` — e.g. `Learning journals for this repo live in docs/learning/`. `CLAUDE.md` loads every session, so the override is always seen.

**One-time setup (Claude Code):** by default Claude Code can only write inside the current working directory, so writing the journal to `~/.claude/learning-journal/` needs a one-time grant in `~/.claude/settings.json`:

```json
{
  "permissions": {
    "additionalDirectories": ["~/.claude/learning-journal"],
    "allow": ["Edit(~/.claude/learning-journal/**)"]
  }
}
```

Keep the rule scoped to `learning-journal/` only — never allow all of `~/.claude/**`, which holds your settings and hooks. Without this grant, the skill will tell you writing is blocked and ask before falling back to `./.claude/learning-journal/` inside the project.

**Platforms:** works on macOS, Linux, and native Windows (Claude Code resolves `~` to your home — `C:\Users\<you>\.claude\` on Windows). If `~` doesn't expand in your Windows permission rules, use the absolute path instead: `"C:\\Users\\<you>\\.claude\\learning-journal"`. Under WSL, Claude Code behaves as Linux, so journals live in your WSL home.

## How it triggers

Learning Mode engages when you signal you want to _learn_ rather than just ship: when you say something is new to you or it's your first time with it, when you ask to understand or study it, or when you ask how/why something works before building. It deliberately stays out of the way when you just want something built, fixed, or delivered — say "just write it" and it won't take over.

## Credits

Built to integrate with the [superpowers](https://github.com/obra/superpowers) skill framework by Jesse Vincent. The teaching method is failure-first — let the learner try the naive approach and feel it break before the concept is named (productive failure). The verification method draws on the principle that primary sources settle claims and that running code beats reading docs for behavior.
