---
name: verify-before-ship
description: Independently verify a finished deliverable (code, workflow config, document, design) before it goes to a client, using a fresh-context reviewer sub-agent that catches what the builder can't see in its own work. Use before delivering or shipping any client-facing output.
argument-hint: "<path to the deliverable, or a description of what to review>"
allowed-tools: Read, Task, Write, Glob, Grep, Bash
user-invocable: true
---

# /verify-before-ship

An agent (or a person) that just built something is structurally bad at
grading it — the reasoning trail that produced it also self-justifies it.
This skill fixes that by handing the finished output, and *only* the
output, to a reviewer with no memory of how it was built, then optionally
loops a fix back through a third, equally fresh agent. See "Why fresh-
context review works" in `docs/agent-patterns.md` (Multi-agent quality
control) for the underlying reasoning.

## Step 0 — Decide if this is worth it

This is a cost/rigor dial, not a default for everything. Skip straight to
shipping for low-stakes internal tooling or something already covered by
automated tests passing. Run this for anything client-facing, anything
touching money/credentials/irreversible actions, or anything where a
mistake would cost more than the few minutes this takes.

## Step 1 — Identify what's actually being reviewed

Get the concrete artifact(s): file paths, a workflow export, a document, a
deployed URL. If an intake contract exists for this project (see
`/intake-contract`, likely at `intake/[project]-contract.md`), read it —
the reviewer should check against the client's actual stated goal/failure
conditions, not a generic quality bar.

## Step 2 — Determine the right kind of check

Per the "deterministic vs. squishy-rubric" and "what verification hinges on
is artifact type" patterns in the library:

- **Checkable behavior** (code with defined expected output, a workflow that
  should produce a specific record shape, an API contract): have the
  reviewer actually exercise it — run it, check the output, don't just read
  the source and eyeball correctness.
- **Subjective/visual output** (design, copy, UX, "does this feel right for
  the brand"): the reviewer needs the same reference material a human judge
  would use — the intake contract's Format/Goal sections, any brand/style
  reference the client gave. Have the reviewer produce a rubric-based
  judgment, not a pass/fail on vibes.

## Step 3 — Spin up the fresh-context reviewer

Use the Task tool (or this environment's Agent tool) to launch a **new**
sub-agent for this — do not review it in the same conversation/context that
built it. Give the reviewer:
- The deliverable itself (files, exported config, whatever Step 1
  identified)
- The intake contract if one exists, or a plain statement of what this was
  supposed to accomplish
- Explicit instructions to look for both `blocking` issues (would embarrass
  us in front of the client, or violates a stated constraint/failure
  condition) and `non-blocking` issues (real but not worth delaying
  delivery for)

Do **not** give the reviewer the implementation conversation, rationale, or
any framing like "I built this, is it good?" — the value of this step
depends on the reviewer having no reason to defend the work. Prompt it
neutrally: "Here is a deliverable and its requirements. Review it."

## Step 4 — Act on findings

- **No blocking issues:** ship it, note the non-blocking issues as
  known-acceptable or a fast-follow.
- **Blocking issues found:** don't fix them in the same context that
  originally built the thing either, if you can avoid it — hand the
  findings to a third, fresh agent (a "resolver") whose only job is fixing
  the listed issues, then re-run this skill once more before shipping. This
  mirrors the implementer → reviewer → resolver pipeline in the pattern
  library, not just implementer → reviewer.

## Step 5 — Record the result

Write a short findings report (what was reviewed, against what standard,
what was found, what was fixed) to
`intake/[project]-verification-[date].md` if this is client work worth a
paper trail — useful both as a delivery artifact you can show the client if
asked, and as a record for us if something surfaces later.

## Why this exists

This operationalizes "sub-agent verification loops," "deterministic vs.
squishy-rubric verification," and "why fresh-context review works" from
`docs/agent-patterns.md` (Multi-agent quality control category). If you
change how this skill works based on real use, fold the learning back into
that pattern-library entry — that's the whole point of the self-improvement
loop in `CLAUDE.md`.
