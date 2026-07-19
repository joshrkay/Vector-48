# Vector-48

We're building agent products for people. Part of how we do that: continuously
studying how others build agents (videos, courses, talks, docs) and turning
what we learn into patterns we can actually use — not just a pile of notes.

## Pattern library

`docs/agent-patterns.md` is the living reference: reusable agent-design
patterns (memory, task specification, multi-agent QC, context/cost
engineering, orchestration), each tagged with why it matters for a product
and where it came from. Read it when working on anything agent-related in
this repo.

## The self-improvement loop (apply this, don't just describe it)

This is the same self-updating-memory pattern documented in
`docs/agent-patterns.md` — applied to this file and that one, on ourselves.

- **When we watch/read a new source**: add new patterns to
  `docs/agent-patterns.md` under the right category (new category if none
  fits), add the source to its Sources list. Capture claims as leads to try,
  not verified fact, unless we've actually confirmed one.
- **When the user corrects something** (a wrong assumption, a bad default, a
  preference): don't just fix it in the moment — add a short rule here so the
  correction persists across sessions instead of repeating.
- **Periodically** (when asked, or when it's a natural point in the work):
  look at `docs/agent-patterns.md` and ask which pattern is worth actually
  prototyping next in whatever we're building, rather than letting the list
  grow indefinitely without being used. Learning that never gets applied
  isn't the goal.

## Learned rules

*(Corrections and standing preferences accumulate here, newest last.)*
