# Learning Mode

A Claude Code skill (plus a companion agent) that turns Claude into a **guide, not an autocomplete**. When you're learning an unfamiliar technology, Learning Mode makes Claude explain, question, and review — but never write the implementation for you, so the understanding ends up in your head instead of in pasted code.

It works across any domain: backend, frontend, distributed systems, databases, algorithms.

## What makes it different

- **Hard no-implementation rule.** Claude does not write the code, config, schema, or commands that constitute the solution. You type every line that counts. Claude gives graduated hints, reviews what you wrote, and corrects it Socratically — it never hands back a finished version.
- **Failure-first, so the "why" is earned.** Instead of lecturing, it has you try the obvious naive solution first and feel where it breaks — and *that* failure is what introduces the technology. Then it contrasts the naive implementation with the production approach, grounded in **real, cited industry sources**, not a guess at "best practice." A learner who has felt the failure a tool prevents can adapt it later; one handed only the recipe can't.
- **Verifies facts instead of guessing.** Factual claims about an API, library, protocol, or service are checked against authoritative primary sources (and, when possible, by running a probe) before they're stated, with a clear label for what's verified vs. reasoned vs. an analogy.
- **Offers depth on request.** When a topic has a meaningful lower layer, Claude flags it and *asks* before diving in — no unprompted firehose.
- **Builds a learning journal.** Each session can close with a structured journal entry (why it exists, naive-vs-production, what you built, verified facts, your own-words explanation, retrieval questions) — a record you can review and feed into a spaced-repetition or quiz routine later.

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

- **Claude Code** (for the plugin/agent). The skill alone also works on claude.ai if uploaded as a `.skill`.
- **Context7 `ctx7` CLI** (recommended) — the doc-verifier prefers it for version-specific library docs. Install with `npx ctx7 setup`. If you use Context7 as an MCP server instead of the CLI, add its tool to the agent's `tools` line.
- **Complements [obra/superpowers](https://github.com/obra/superpowers)** — Learning Mode is independent, but its trigger is written so it also engages when a superpowers brainstorming/planning/execution step touches a technology you've flagged as unfamiliar.

## Install

### Option A — As a plugin via this marketplace (recommended; bundles the skill + agent)

In Claude Code:

```
/plugin marketplace add melsayedx/learning-mode-plugin
/plugin install learning-mode@learning-mode-marketplace
```

(Restart Claude Code or run `/reload-plugins` after installing so the agent loads.)

### Option B — As a global skill only (drop-in, no agent)

Copy the skill folder into your user skills directory so it's available in every project:

```
cp -r plugins/learning-mode/skills/learning-mode ~/.claude/skills/learning-mode
```

To also get the verifier agent, copy it into your user agents directory:

```
cp plugins/learning-mode/agents/doc-verifier.md ~/.claude/agents/doc-verifier.md
```

Restart Claude Code to load the agent.

### Option C — As a `.skill` on claude.ai

Use the packaged `learning-mode.skill` (the skill only — the agent is Claude-Code-specific) and upload it in claude.ai under Settings → Capabilities → Skills.

## How it triggers

Learning Mode engages when you signal you want to *learn* rather than just ship: when you say something is new to you or it's your first time with it, when you ask to understand or study it, or when you ask how/why something works before building. It deliberately stays out of the way when you just want something built, fixed, or delivered — say "just write it" and it won't take over.

## Customization

- The journal location is a convention you choose: per-project `docs/learning/<date>-<topic>.md` or a global `~/learning-journal/`. Point your review routine at whichever you pick.

## Credits

Built to integrate with the [superpowers](https://github.com/obra/superpowers) skill framework by Jesse Vincent. The teaching method is failure-first — let the learner try the naive approach and feel it break before the concept is named (productive failure). The verification method draws on the principle that primary sources settle claims and that running code beats reading docs for behavior.
