# Contributing to Learning Mode

Thanks for wanting to improve this. A few ground rules keep the skill effective.

## Before you open a PR

**Open an issue first for any behavioral change** — anything that adds, removes, or rewords a rule in `SKILL.md` or the references. The skill is a prompt: every instruction competes with every other instruction for the model's attention, so additions must earn their place. "It would be nice if" is not enough; "sessions currently fail at X, this fixes it" is.

Doc fixes, typos, and README improvements can go straight to a PR.

## Design principles (changes must respect these)

- **One rule, one home.** A rule lives in exactly one file. Cross-references point; they don't repeat.
- **Rule + one-line rationale.** Instructions with a stated _why_ are followed more reliably — keep the why, cut the elaboration.
- **Ownership follows the target.** The learner owns the load-bearing reasoning and artifact selected for practice. The assistant may remove unrelated setup friction; do not turn scaffolding into compulsory homework.
- **Load-bearing claims get verified.** Version-specific, current, uncertain, security-sensitive, or implementation-changing facts need primary evidence. Stable fundamentals and incidental facts do not need lookup theater.

## What's especially welcome

- **Worked examples for new domains** (`references/`) — examples that begin with one outcome and the smallest prerequisite, then adapt the progression to the learner.
- **Bug reports with transcripts** — activation mistakes, target-solution leakage, unnecessary setup work, ignored journal preferences, incorrect intent switching, and weak transfer checks are easiest to fix from a real session excerpt.
- **Translations** of the README.

## Testing a behavioral change

Describe how you tested: include representative before/after session excerpts and cover
activation, target ownership, intent switching, safety behavior, and journaling when the
change touches them. Untested behavioral PRs will be asked for evidence.
