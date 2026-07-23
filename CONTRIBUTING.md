# Contributing to Learning Mode

Thanks for wanting to improve this. A few ground rules keep the skill effective.

## Before you open a PR

**Open an issue first for any behavioral change** — anything that adds, removes, or rewords a rule in `SKILL.md` or the references. The skill is a prompt: every instruction competes with every other instruction for the model's attention, so additions must earn their place. "It would be nice if" is not enough; "sessions currently fail at X, this fixes it" is.

Doc fixes, typos, and README improvements can go straight to a PR.

## Design principles (changes must respect these)

- **One rule, one home.** A rule lives in exactly one file. Cross-references point; they don't repeat.
- **Rule + one-line rationale.** Instructions with a stated _why_ are followed more reliably — keep the why, cut the elaboration.
- **Construction over consumption.** Anything that makes Claude hand the learner a finished artifact (code, diagrams, explanations they didn't work for) violates the core rule. If your feature makes learning more convenient but less active, it will be declined.
- **Claims get verified.** Factual statements in references must carry a source; anything unverifiable is labeled, not asserted.

## What's especially welcome

- **Worked examples for new domains** (`references/`) — a concept-mode arc (e.g. a distributed-systems topic) following the same beats as the Stripe example.
- **Bug reports with transcripts** — the skill's failure modes (writing code it shouldn't, skipping the journal, silent misplacement of files) are easiest to fix from a real session excerpt.
- **Translations** of the README.

## Testing a behavioral change

Describe how you tested: a before/after session excerpt showing the behavior difference. Untested behavioral PRs will be asked for one.
