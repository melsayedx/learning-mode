# Worked Examples: the naive-vs-production teaching arc

## How to use this file

This is an **exemplar of the method**, not a script. It shows how to run one topic
through the learning-mode arc end to end — why the thing exists, the naive approach, each
production divergence with a *sourced* reason, where to offer a deeper dive, and how to
get the learner to generate the tests. Do not paste it at the learner or walk them
through it top to bottom; use it as the shape to follow while *they* build.

It also models the rules the skill insists on: the production claims below are grounded in
Stripe's official documentation (docs.stripe.com/webhooks), each divergence leads with the
failure it prevents, and the one place the docs are deliberately vague is flagged as a
thing to verify rather than guessed. Technology specifics drift — this arc was grounded
against Stripe's docs at authoring time; re-verify against current docs before teaching.

---

## Example: Stripe webhooks (backend)

The learner says webhooks are new to them and they want to handle Stripe payment events.
Run the arc. Throughout: you ask and they build — none of the fixes below are code you
write for them.

### Beat 1 — Let the naive solution fail first

Before introducing webhooks at all, hand them the problem and let them solve it the
obvious way. Ask, don't tell:

- "Stripe charges a customer's card on Stripe's servers. How does *your* server find out it succeeded?"
- They'll reach for polling ("I'll call Stripe's API every few seconds"). Don't stop them — let it stand, then make the wall felt: "At what cost, with 10,000 customers? How fast do you learn about a failure? What happens to your rate limits?"

This is the naive *solution*, and feeling it break is the whole point — the need has to
be felt before the concept is named.

### Beat 2 — The failure is the answer: why webhooks exist

Now the "why" lands, because they just hit the wall it solves: a webhook is Stripe
*pushing* you the event instead of you *pulling* for it — lower latency, no wasted
polling, no rate-limit pressure. That is why webhooks exist, and it's true of any
event-notification system, not just Stripe (the general model). Note the contrast with a
cold lecture: you didn't open with "webhooks are a push mechanism," they earned it.

### Beat 3 — Now implement it, naively

With webhooks motivated, have them sketch their first handler. Most people land on:
receive the POST, read the JSON body, do the work inline (mark the order paid, email the
customer), return 200. Don't correct it yet — it *looks* right, and naming why it's
tempting sharpens the contrast. This is the second naive step: the obvious *use* of the
new tool. Now walk them into its failures one at a time, each as a question. Some of these
walls they'll feel in a local test; others (a forged event, a duplicate, an out-of-order
delivery) you'll have to *pose*, because the happy path hides them.

### Beat 4 — "What stops me from POSTing a fake event?"

The failure: the naive handler trusts the request body, so anyone who knows the URL can
POST a fake `payment_intent.succeeded` and trigger fulfillment. Stripe's docs are blunt
about the stakes: without verification, an attacker can send fake events to fulfill
orders, grant account access, or modify records.

The fix (let them implement it): verify the `Stripe-Signature` header using Stripe's
official library and the endpoint's signing secret. The *why* behind the two details
they'll most likely get wrong:

- **Verify over the raw body, not the parsed JSON.** Stripe's docs state that verification requires the raw request body, and any middleware that re-serializes it breaks verification. Ask: "Your framework parses JSON before your handler runs — what has it done to the exact bytes Stripe signed?"
- **Use the library; don't hand-roll the HMAC.** Stripe signs with HMAC-SHA256, scheme `v1`. Hand-rolled checks tend to skip the constant-time comparison and the downgrade-scheme guard (deep-dive offer below).

Verify-it beat: have them open Stripe's signature page and confirm the exact
construct-event call for their language, rather than taking your word for the API shape.

### Beat 5 — "Your handler emails the customer and syncs an ERP inline. What happens when that's slow?"

The failure: heavy work inline makes the response slow; Stripe times out, marks the
delivery failed, and retries — now you have duplicate processing and retry storms.
Stripe's docs require the endpoint to return a 2xx *quickly, prior to any complex logic
that could cause a timeout* — explicitly, return the 200 before updating an invoice as
paid in your accounting system.

The fix (let them derive the shape): verify → persist or enqueue → return 2xx → do the
heavy work in a background worker. Stripe itself recommends processing events with an
asynchronous queue, because a spike (start-of-month subscription renewals, say) can
overwhelm a synchronous handler.

**A verification lesson worth making explicit.** Ask "how many seconds do you have to
respond?" They'll find blog posts confidently saying 5, 10, and 20 seconds — and the
numbers contradict each other. Stripe's official docs deliberately don't give a number;
they say "quickly," and the dashboard simply marks a delivery "timed out." So the honest
answer is: treat *fast* as the contract, return before any heavy work, and build nothing
that depends on a specific number. This is the skill's verification discipline in
miniature — when the primary source is silent and secondary sources disagree, you say so
rather than pick one to sound decisive.

### Beat 6 — "Stripe delivered the same event twice. Did you fulfill twice?"

The failure: Stripe uses at-least-once delivery and retries for up to three days with
exponential backoff in live mode, so the same `event.id` will arrive more than once. The
naive handler fulfills on every delivery.

The fix (let them implement): record processed `event.id`s and short-circuit on a repeat
— Stripe's docs say to log the event IDs you've processed and skip already-logged ones.

Production refinement (this is the production-practice tier, reasoned past the docs): the
idempotency record and the business work belong in the *same database transaction*. Ask:
"If you fulfill the order, then crash before writing the 'processed' row — what happens on
the retry?" They'll see the double-fulfill and derive that the dedupe insert and the
fulfillment must commit atomically. Mark this as the widely-used production pattern,
reasoned from the crash case — not a line in Stripe's docs.

### Beat 7 — "The 'updated' event arrived before the 'created'. Now what?"

The failure: the naive handler assumes ordering. Stripe's docs are explicit that delivery
order is *not* guaranteed — creating a subscription can deliver
`customer.subscription.created`, `invoice.created`, `invoice.paid`, and `charge.created`
in any order.

The fix (let them reason it out): don't depend on order. When you need current state,
re-fetch the object from Stripe's API rather than trusting whichever event arrived first
— which is exactly what Stripe recommends.

### Deep-dive offers (ask first — don't pour these out)

At a natural pause, once the arc above is solid, offer these — and only if they say yes:

- **How signature verification actually works:** the `Stripe-Signature` header's `t=` timestamp and `v1=` HMAC-SHA256 scheme, why you ignore non-`v1` schemes (downgrade attacks), and why the comparison must be constant-time (timing attacks). This is the layer beneath the one-line library call.
- **The replay-attack defense:** the timestamp is part of the signed payload, libraries default to a five-minute tolerance, and a tolerance of 0 disables the check entirely. Good for a learner who asks "what stops someone replaying a captured event?"
- **At-least-once delivery and the transaction boundary:** why a queue buys you at-least-once and not exactly-once, and why that pushes idempotency down into the database.

### Make them generate the tests

Don't hand them a list. Ask "what could go wrong with this handler?" and build it
together, steering toward the cases they miss: a duplicate `event.id`; out-of-order
events; a forged or malformed signature; middleware that mutated the raw body (so
verification fails for a *correct* payload); a slow downstream causing a timeout; an event
type you don't handle. Then point them at the real tooling so they test empirically rather
than hypothetically: `stripe listen --forward-to localhost:<port>/webhook` to forward real
events locally, and `stripe trigger payment_intent.succeeded` to fire one. Watching a real
duplicate or a real signature failure teaches more than reasoning about it.

---

## Reusing this shape for any topic

The Stripe specifics are incidental; the arc is the reusable part.

1. **Let the naive solution fail first** — have them solve the problem the obvious way (here, polling) and feel where it breaks, before the new technology is named.
2. **The failure motivates the technology** — the wall they just hit is *why* the thing exists; name the general model now that they've earned it.
3. **Implement it naively** — have them build the obvious first version with the new tool, and name why it's tempting.
4. **Each production divergence** — lead with the failure it prevents, surface it with a question, let them implement the fix, and ground the *why* in a real source (official docs or a real production practice), flagging anything you can't verify. Some walls they'll feel; others you'll have to pose.
5. **Deep-dive offers** — name the layer beneath and let them choose to enter it.
6. **Test elicitation** — they generate the cases; push them toward testing empirically with real tooling.

Whatever the topic — a database index, a consensus protocol, a rate limiter — the same
six beats apply. The sources change (university courses and canonical texts for the
general model, the project's own docs for specifics, engineering blogs and real production
code for practice); the shape does not.
