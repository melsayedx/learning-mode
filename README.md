# Learning Mode

[![MIT License](https://img.shields.io/badge/license-MIT-111827.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude_Code-plugin-D97757.svg)](https://code.claude.com/docs/en/plugins)
[![Codex plugin](https://img.shields.io/badge/Codex-plugin-111827.svg)](https://developers.openai.com/plugins)

An agent-agnostic guided-learning skill that makes the learner own the selected learning
target. The assistant defines one outcome, checks the smallest prerequisite, guides
prediction and attempts, reviews without rewriting, verifies consequential technical
claims, and optionally preserves progress across sessions.

The same behavioral core is packaged for Claude Code and Codex.

## Install

### Claude Code

```text
/plugin marketplace add melsayedx/learning-mode
/plugin install learning-mode@learning-mode-marketplace
/reload-plugins
```

### Codex

Add the repository as a marketplace, then install the plugin:

```bash
codex plugin marketplace add melsayedx/learning-mode
codex plugin add learning-mode@learning-mode-marketplace
```

During local development, replace `melsayedx/learning-mode` with the absolute path to
your clone.

OpenAI uses one universal public plugin directory for ChatGPT and Codex. The
`.agents/plugins/marketplace.json` file in this repository is the local/repository
marketplace used for Codex development and testing; it is not a separate public
submission.

Start a new conversation after installing or updating the plugin.

## Test locally without publishing

The Codex marketplace can point directly at a local clone. Nothing needs to be committed,
pushed, or submitted to the public plugin directory.

If the published marketplace is already configured, temporarily replace it because the
published and local sources share the `learning-mode-marketplace` name:

```bash
codex plugin remove learning-mode@learning-mode-marketplace
codex plugin marketplace remove learning-mode-marketplace
codex plugin marketplace add "/absolute/path/to/learning-mode"
codex plugin add learning-mode@learning-mode-marketplace
codex plugin list
```

The local path is the repository root containing `.agents/plugins/marketplace.json`.
Confirm that `codex plugin list` reports the expected local version, then start a new
conversation, select Learning Mode from the `$` skill picker, and try:

> Teach me React state through guided learning. Do not implement the solution for me.

Smoke-test the intent routes and safety boundaries:

- a quick factual question receives a direct answer;
- a guided request follows the prerequisite, attempt, validation, and transfer flow;
- a delivery request made during a lesson requires confirmation before mode exit;
- journaling asks once and writes under `~/learning-journals/` only when enabled;
- urgent security or data-loss risk receives immediate protective guidance.

After changing the local plugin, update its development cache-busting version, reinstall
it, and start another new conversation:

```bash
python3 ~/.codex/skills/.system/plugin-creator/scripts/update_plugin_cachebuster.py \
  "/absolute/path/to/learning-mode/plugins/learning-mode"
codex plugin add learning-mode@learning-mode-marketplace
```

Restore the release version in `.codex-plugin/plugin.json` before preparing a commit or
public release. To return to the published marketplace:

```bash
codex plugin remove learning-mode@learning-mode-marketplace
codex plugin marketplace remove learning-mode-marketplace
codex plugin marketplace add melsayedx/learning-mode
codex plugin add learning-mode@learning-mode-marketplace
```

## What it does

Learning Mode first determines intent:

- **Quick answer:** answer directly.
- **Guided learning:** activate Learning Mode.
- **Delivery:** build or fix the result normally.

In guided learning, it follows this flow:

```text
Determine intent: quick answer, guided learning, or delivery
    ↓
Define one learning outcome
    ↓
Check the smallest relevant prerequisite
    ↓
Select the next missing concept
    ↓
Predict → attempt → feedback → retry
    ↓
Validate against the defined outcome
    ├─ Fails → diagnose, revise, and retry
    └─ Passes → continue
    ↓
Can the learner explain and transfer it?
    ├─ No → hint ladder or analogous example
    └─ Yes → next checkpoint or finish
    ↓
Update journal only if previously enabled
```

Example request:

> Redis Streams are new to me. Guide me through building a worker, but leave the
> load-bearing implementation and reasoning to me.

## Learner ownership without setup theater

The skill defines the learning target before deciding what the assistant may write.

If the target is React form state, the learner owns the state model, transitions, and
important tests. The assistant may prepare unrelated project files or an empty test
harness.

If the target is build-tool configuration, that configuration becomes load-bearing and
the learner owns it.

The boundary is:

> The learner owns the target. The assistant may remove incidental friction.

## Adaptive progression

The skill does not start with a giant syllabus. It checks one prerequisite, selects the
next missing concept, and runs a short cycle:

```text
predict → attempt → feedback → retry
```

When the learner is stuck, the assistant climbs a four-rung hint ladder. If the learner
still lacks the underlying schema, it shows one fully worked analogous problem and then
returns to the original target.

Before completion, the learner explains the mechanism and handles one changed example,
transfer question, or counterexample.

## Visual-first when relationships are visual

Learning Mode prefers the smallest useful visual when relationships are easier to inspect
together than explain serially:

- diagrams or trees for structure, hierarchy, dependencies, and data flow;
- flows, state diagrams, or timelines for sequence, state, concurrency, and timing;
- tables for comparisons and exact mappings;
- plots for quantities, trends, and distributions;
- interactive models only when manipulating the system teaches more than a static visual.

The learner predicts, sketches, traces, or annotates when constructing the representation
is part of the target. The assistant may provide a partial scaffold or a reference visual,
then asks the learner to explain or modify it for a changed case. Short facts, syntax, and
linear steps stay as text when a visual would add noise.

## Application and trade-offs

For each substantial concept, Learning Mode offers one authentic way to apply it:
implementation, design, prediction, diagnosis, critique, simulation, or decision. The
learner performs the load-bearing reasoning, compares one plausible alternative, and
explains what the concept improves, what it costs, and when it should not be used. One
relevant constraint then changes so the learner can reconsider the choice.

For example, after learning Onion Architecture, the learner can map a real service,
compare that design with a traditional layered architecture, identify the extra
indirection and boilerplate, and revisit the decision for a smaller system or team.
Theoretical topics use a decision, critique, simulation, or prediction when implementation
would be artificial.

Small syntax facts, quick corrections, already-mastered concepts, and declined practice
skip this step.

## Validation versus verification

**Validation** asks whether the learner's artifact or reasoning satisfies the selected
outcome in the tested conditions.

After a viable attempt:

1. Restate the success criteria.
2. Have the learner propose the important cases.
3. Run the smallest relevant test, probe, trace, build, type check, worked case, or
   observation.
4. Compare the result with the prediction.
5. Return failures to the feedback-and-retry loop.

A passing check proves only what was tested in that environment. It does not prove
universal correctness or learner understanding.

**Verification** asks whether the technical claims behind the work are accurate,
current, and supported by authoritative evidence.

Learning Mode verifies facts when being wrong could change the implementation or mental
model:

- version-specific APIs and defaults;
- current service limits, guarantees, retries, quotas, and rate limits;
- protocol, security, authorization, and data-integrity requirements;
- claims presented as real production practice;
- uncertain, surprising, disputed, or plausibly changed facts;
- behavior that can be checked safely with a focused probe;
- load-bearing facts saved into the journal.

It does not perform live lookups for stable foundational knowledge, pedagogical choices,
learner preferences, or incidental facts that cannot change the outcome.

A probe establishes observed behavior in the tested environment. Documentation,
specifications, and machine-readable contracts establish what is guaranteed.

The skill prefers primary sources and marks claims as verified, inferred, or unconfirmed
when provenance matters.

## Journals

Journaling is opt-in and agent-agnostic. The default root is:

```text
~/learning-journals/
```

The assistant asks once whether journaling should apply to all Learning Mode sessions or
only the current project, then remembers the choice using available durable memory,
project instructions, or a preference under the journal root.

When enabled, journals are stored at:

```text
~/learning-journals/<date>-<topic>/journal.md
```

The learner can pause journaling, disable it, or change the location at any time. If the
current agent cannot write to the selected directory, it explains the limitation and
asks before using another location.

## Intent changes and safety

If the learner asks for direct delivery after guided learning has begun, the assistant
confirms before leaving Learning Mode:

> You want me to leave Learning Mode and deliver the solution directly. Should I switch?

Guided struggle pauses when delay or incomplete guidance could worsen security, data
loss, destructive changes, financial harm, privacy exposure, or an active production
incident. The assistant gives immediate protective guidance, then asks whether to switch
to direct remediation.

## Package structure

```text
.
├── .agents/plugins/marketplace.json              # Codex repo marketplace
├── .claude-plugin/marketplace.json                # Claude marketplace
└── plugins/learning-mode/
    ├── .codex-plugin/plugin.json                  # Codex plugin manifest
    ├── .claude-plugin/plugin.json                 # Claude plugin manifest
    ├── agents/doc-verifier.md                     # optional Claude verifier wrapper
    └── skills/learning-mode/
        ├── SKILL.md                               # agent-neutral behavioral core
        ├── agents/openai.yaml                     # Codex/ChatGPT skill metadata
        ├── references/
        │   ├── verification.md                    # verification method
        │   └── examples.md                        # implementation-learning example
        └── assets/journal-template.md
```

## Requirements

- A host that supports agent skills, such as Claude Code or Codex.
- Context7 is recommended for version-specific library documentation. The verification
  method can fall back to official documentation, source, and tests.
- Visuals use the smallest format supported by the host, with tables or text diagrams as
  portable fallbacks. Mermaid remains optional.

## How it triggers

Learning Mode activates for explicit active-learning intent such as:

- “Teach me this.”
- “Guide me while I build it.”
- “I want to practice or master this.”
- “This is new to me; help me do it myself.”

A one-off “what,” “how,” or “why” question does not automatically start a workshop.
Routine build, fix, and direct-delivery requests stay outside Learning Mode.

## Credits

Created by [Muhammad El Sayed](https://github.com/melsayedx). The skill combines
diagnostic checkpoints, guided inquiry, graduated hints, self-explanation, transfer
checks, and source-grounded technical verification.
