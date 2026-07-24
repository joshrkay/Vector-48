---
name: intake-contract
description: Turn a rough, ambiguous client ask (a call transcript, an email, a voice memo, a one-line brief) into a structured, clarified project contract before any build work starts. Use at the start of a new client engagement or whenever a deliverable's scope is fuzzy.
argument-hint: "<raw client ask, or path to notes/transcript/email>"
allowed-tools: Read, Write, AskUserQuestion, Glob, Grep
user-invocable: true
---

# /intake-contract

Most rework and scope creep on client work traces back to one thing: starting
the build before anyone forced the ambiguous parts of the ask into the open.
This skill runs two steps before any deliverable work begins — reverse
prompting, then a written contract — so the team (and the client, if the
contract is shared back) agrees on scope before time gets spent.

## Step 1 — Read the raw input

Accept whatever the user hands you: pasted text, a file path (call notes,
email thread, voice-memo transcript), or a one-line ask typed directly. If
it's a file path, `Read` it. Don't ask the user to reformat it first — this
skill's whole point is starting from messy input.

## Step 2 — Reverse prompting: ask what's actually unclear

Before drafting anything, identify the assumptions this specific request
is quietly making, and turn them into 3-6 concrete questions. This is a
judgment call about *this* request, not a generic checklist — bad example:
"what's your timeline?" on every project regardless of whether timeline was
ever ambiguous. Good example, tailored to what's actually fuzzy in the
input: "You said 'automate our lead follow-up' — does that mean drafting
replies for a human to send, or sending them autonomously?"

Common categories worth checking (only ask the ones actually unresolved by
the input):
- **Success/failure boundary.** What does "done" look like, concretely? What
  would count as this project failing even if it technically works (e.g. "it
  works but no one at the client trusts it enough to turn it on")?
- **Autonomy level.** Does the client want a human-in-the-loop step
  anywhere, or full autonomy? (Directly maps to the "gate side effects
  behind classification" and "human-approval checkpoints in skills" patterns
  in `docs/agent-patterns.md` — if the client wants a human checkpoint on
  anything irreversible, that belongs in the contract, not discovered later.)
- **Existing systems/data.** What does this need to integrate with, and
  what's the current source of truth?
- **Volume/scale.** Rough number of records/requests/users this needs to
  handle — changes the right architecture (see "bounded task → deterministic
  logic, open-ended task → full agent" in the pattern library).
- **Ownership after delivery.** Who maintains this once it ships — us, the
  client's team, nobody?

Use `AskUserQuestion` to ask these (batch them into one call, up to 4
questions per call, multiple calls if you have more than 4 genuinely
unresolved items). If the user says "just make reasonable assumptions" for
some or all of them, that's fine — record the assumption explicitly in the
contract's Constraints section rather than silently deciding for them.

## Step 3 — Draft the contract

Once the ambiguous points are resolved (answered or explicitly assumed),
write a contract with exactly these four sections:

```markdown
# [Project/deliverable name] — Intake Contract

**Date:** [today]
**Client:** [client name, if known]

## Goal
One or two sentences. What this delivers and why it matters to the client —
not a feature list, the actual outcome.

## Constraints
Everything that bounds the solution: budget/timeline if known, systems it
must integrate with, autonomy level agreed in Step 2, data/privacy
constraints, explicit assumptions made where the client deferred to us.

## Format
What the deliverable actually is — a hosted endpoint, a skill the client's
team runs locally, a Slack bot, a one-time script, a dashboard. Be concrete;
"an automation" is not a format.

## Explicit failure conditions
Name the ways this could look done but actually fail — the things a
literal reading of "it works" would miss. E.g. "runs but produces emails
that don't sound like the brand," "processes leads but misses ones that
come in outside business hours," "technically automated but nobody on the
client's team trusts it enough to stop double-checking it by hand."
```

## Step 4 — Save and present

Write the contract to `intake/[client-or-project-slug]-contract.md` in the
current repo (create the `intake/` directory if it doesn't exist). Show the
user the full contract and ask them to confirm before treating this as
final — if anything in Step 2's answers was a guess, flag it one more time
here rather than burying it in the document.

## Why this exists

This operationalizes the "prompt contracts" and "reverse prompting" patterns
in `docs/agent-patterns.md` (Task specification category), applied
specifically to client intake rather than a generic coding task. If you
change how this skill works, update that pattern-library entry too so the
two don't drift apart.
