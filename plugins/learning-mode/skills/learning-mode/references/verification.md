# Verification Method

A tutor that reinforces a misconception is worse than no tutor, because the learner
leaves *confident* and wrong. This file is the procedure for grounding factual claims
about a specific technology. It is the single source of truth for both readers:

- The **main agent**, reading it inline before stating a fact in the conversation.
- The **doc-verifier subagent**, which loads this file as its complete methodology.

It is written to be self-sufficient. A subagent that has only this file and the claim
to check has everything it needs — it does not re-derive the method, it follows this
one. The choice of *inline vs. subagent* is about keeping the main context clean and is
decided elsewhere; the method below is identical either way.

---

## 1. Triage first: what to verify, and what not to

Do not verify everything. Verifying stable, foundational facts is theater — it burns
latency and tokens and trains the user to distrust your judgment. Verifying nothing
lets a stale or hallucinated claim through. The line between them is a single question:

> **If I am wrong about this, does the learner build a wrong mental model or wrong code?**

If yes, verify. If no, proceed on reliable knowledge (and offer a canonical source only
if the learner wants to go deeper).

**Proceed without a lookup** for slow-changing fundamentals you hold reliably: what a
hash table is, how TCP's handshake works conceptually, the Big-O of binary search, what
ACID means. These have not changed and do not need a live source to state.

**Always verify**, regardless of how confident you feel:

- Version-specific API surface — method names, signatures, default values, deprecations, return types.
- Current behavior or limits of a service — timeouts, retry windows, rate limits, quotas, guarantees.
- Anything that could have changed after your knowledge cutoff.
- Anything you are not certain of from reliable knowledge. Uncertainty is itself a trigger; do not paper over it.
- Anything where a wrong answer sends the learner down a wrong implementation path.

Examples: *"What's the Big-O of binary search?"* → reliable knowledge, no lookup.
*"What's the default pool size in asyncpg 0.30?"* → verify, version-pinned.
*"Does Stripe guarantee webhook delivery order?"* → verify against Stripe's docs.

## 2. Source hierarchy by claim type

Pick the tier from the *kind* of claim. Within every tier, primary sources settle a
claim and secondary sources only point you toward one (see section 3).

- **Conceptual / theoretical claim** ("what is X", "why does X exist", "what do most systems do here") → top-tier academic material: graduate and upper-undergraduate course materials from leading universities, plus the canonical texts of the field. Treat the named courses and books below as *examples of the bar*, not a closed list — use whichever top-tier source actually covers the topic. By domain: databases (CMU 15-445/645, Berkeley CS186), distributed systems (MIT 6.824), operating systems (OSTEP, MIT 6.S081), algorithms and data structures (MIT 6.006, Stanford CS161, CLRS, Skiena), networking (Stanford CS144), and the equivalents for compilers, architecture, and the rest. The defining property is the one a strong course has: it teaches the general model and the *reason* for it, not one product's API. Use these to establish the **general model first**.
- **Library / framework API specific** ("what does method M do", "what's the default", "is this deprecated") → **Context7**, version-pinned. Prefer the official `ctx7` CLI when it's installed: a CLI is invokable ad hoc through the shell with no per-session MCP wiring, which matters most for a subagent running in a forked context where MCP tools may not be configured. Resolve the name to an ID (`ctx7 library <name>`), then fetch docs (`ctx7 docs <library-id> <query>`). If the CLI isn't present, use the Context7 MCP — same source, same version-pinning. Context7 beats a generic web fetch for library facts because it serves version-specific documentation derived from source. Fall back, in order, to the library's official documentation site, then its source and tests on the repo host.
- **Service / protocol behavior** ("does Stripe retry", "what status code", "what does the spec require") → the official vendor documentation plus the governing spec or RFC.
- **Production-practice claim** ("how is this *actually* done in production", "what's the real-world approach", "what do mature teams do here") → real, named industry sources — never a hypothesis. Use engineering blogs from a team that builds or operates the system (Stripe, Cloudflare, Netflix, Shopify, Uber, and the like), the production source of a well-known open-source project that does this, public postmortems and incident write-ups, conference talks (QCon, Strange Loop, re:Invent), and official reference implementations or sample apps. These are *primary for practice* — a real team's real decision — even though they aren't vendor API docs. Check recency: an approach from a 2016 post may have been superseded. If you cannot find a real source for how something is done in production, say so and label the suggestion reasoned, not sourced (see section 6) — do not present a guess as the industry standard.

The academic tier and the docs/Context7 tier answer different questions and you will
often use both for one topic — the academic source for the principle, the docs for this
technology's specific take on it.

> Context7 command names (`ctx7 library`, `ctx7 docs`) and MCP tool names
> (`resolve-library-id`, `get-library-docs`) are stated from general knowledge and
> current as of this writing. Confirm them against `ctx7 --help` or your installed MCP
> before relying on them — CLI subcommands and MCP tool names shift between releases.
> This is the same version-pinning the rest of this file demands; apply it to your own
> tools too.

## 3. Primary settles, secondary points (the anti-hallucination rule)

This is the rule that does the real work, because the characteristic LLM failure is not
inventing a source — it is *searching, then pattern-matching a snippet to what you
already assumed*, and calling that verification.

- **Primary sources** settle a claim: official documentation, the spec or RFC, the library's own source code, and Context7 (source-derived).
- **Secondary sources** never settle a claim: blogs, Stack Overflow, tutorials, forum answers, AI-generated summaries. They are useful only to *find* the primary source.
- **Never confirm a fact from a search-result snippet.** Open the primary page and read it. Let the source correct you — go in expecting it might, not hunting for the line that agrees with you.
- A secondary source contradicting a primary source does not create a conflict. The primary wins.

**Stronger than reading docs: get to ground truth.** Where you can obtain it, direct
evidence beats documentation, because docs lag the code, omit edge cases, and
occasionally state things that were never true:

- **Run it — empirical verification.** For a claim about *behavior* (what a call returns, whether an event fires, what an error actually looks like), the most reliable check is to execute a small probe and observe. In a learning session this is also the better teaching move: not "the docs say X" but "write five lines, run them, and watch." One caveat that matters — observing a behavior tells you what the system does *in your version, right now*, not whether it is *guaranteed*; an undocumented behavior you lean on can vanish in the next release. So pair the two: run it to see what happens, read the spec to learn whether that behavior is contractual or incidental.
- **Read the source and its tests.** When prose docs are ambiguous, the library's own source is ground truth, and its test suite is the most precise statement of intended behavior — tests are executable specifications the maintainers committed to.
- **Prefer machine-readable contracts over prose.** A type signature, an OpenAPI or protobuf definition, a JSON Schema — these are what the code actually enforces, so they outrank a prose sentence that may have drifted from the implementation.

## 4. Version-pin everything

"Framework X does Y" is meaningless without a version. Before verifying a library or
service claim, establish which version the user is actually on, and verify against
**that**, not against the latest. When the installed version and the latest docs differ,
the installed version is the truth for this learner. If you don't know their version,
ask or check their lockfile before asserting version-dependent behavior. When behavior
appears to have changed, the changelog, release notes, and migration guide pinpoint
*which* version changed it — consult them before trusting an older source.

## 5. Conflict resolution

- Primary beats secondary, always.
- The user's installed version beats the latest documentation.
- If two primary sources genuinely conflict, or the primary source is silent on the point, **say so explicitly and mark the claim unconfirmed.** Do not pick one to look decisive. "The docs don't specify this; here's what I'd test to find out" is a correct and honest answer.

## 6. Confidence labels: what you hand the learner

Every non-trivial factual claim carries one of these, so the learner always knows what
is checkable truth versus a teaching device. This is the operational form of SKILL.md's
"separate what's verified from what you're inventing."

- **Verified [source]** — read in a primary source, cited with a link.
- **Inferred** — not stated directly but derived by reasoning from verified facts. Flag it as inference and show the reasoning so the learner can check the logic.
- **Unconfirmed** — could not verify against a primary source. Flag it, and state what it would take to confirm (which doc, which experiment).
- **Claude's creativity** — an analogy, mental model, or example you designed. Flag it as a teaching device, not authoritative.

Triangulate where the stakes warrant it: a claim confirmed by two independent primary
sources — say, the spec and the source code, or the docs and an empirical probe — is
stronger than one resting on a single page. A lone source can be stale or wrong;
agreement across independent ones is the cheapest confidence you can buy.

## 7. The teaching link: general model → specific divergence → naive vs. production

This file is not only a fact-checker; it is the grounding for the naive-vs-production
arc in SKILL.md, so that contrast rests on verified fact rather than vibes.

- The **academic tier** supplies the "why it exists" and "what most systems do" beats — the general principle.
- The **docs / Context7 tier** supplies "what *this* technology does" — the specific implementation.
- **Name the divergence explicitly.** The pattern to imitate is the way a strong systems course teaches: *"most databases do A here; this one does B, because C."* That single move — general rule, then the specific deviation, then the reason — is what makes a concept transfer instead of being memorized.

## 8. Subagent return contract (read by the doc-verifier in Piece 3)

When verification runs in a subagent, the subagent receives the specific claim plus this
file. It returns a compact, citable result per claim — not a dump of raw documentation,
because the main agent has to present it inside a teaching turn and cite it cleanly.

Return this shape for each claim checked:

```
CLAIM:        <the claim as actually checked, restated precisely>
VERDICT:      verified | unconfirmed | contradicted
SOURCE:       <primary source URL> — <section/anchor> — <version, if applicable>
DETAIL:       <the supporting fact, paraphrased in one or two sentences, not quoted at length>
CONFIDENCE:   verified | inferred | unconfirmed
```

If the verdict is `unconfirmed`, the subagent additionally returns what it searched and
what specific source or experiment would settle it, so the main agent can decide whether
to dig further or flag it to the learner.

---

## The loop (quick reference)

1. **Triage** — would being wrong derail the learner? If not, proceed; cite a canonical source only on request.
2. **Pick the tier** by claim type (section 2): concept → academic; library API → Context7; service/protocol → official docs + spec.
3. **Version-pin** if it's a library or service; verify against the user's installed version.
4. **Go primary** — open and read the primary source; let it correct you. Never settle from a snippet.
5. **Resolve conflicts** by section 5; when sources are silent or conflicting, mark unconfirmed.
6. **State the claim with a label and citation** (section 6). For long groundings, write the explanation to a Markdown file rather than inline (per SKILL.md).
