# Agent Design Patterns

A running library of reusable patterns for building agent products, sourced from
external videos/courses/talks as we study how others are building. Treat entries
as leads to try, not verified fact, unless a "Verified" note says otherwise —
the goal right now is breadth of ideas to draw from, not rigor.

Each pattern lists: what it is, why it matters for a product (not just a demo),
and the source it came from.

## How to extend this file

When we watch/read a new source: add patterns under the relevant category below
(create a new category if none fits), and add the source to the Sources list at
the bottom. Keep entries terse — one to three sentences. Link back to the source
for detail instead of duplicating it here.

---

## Memory & self-improvement

- **Self-updating rule files.** Agent appends a numbered rule to its own
  claude.md/agents.md every time a user corrects it (e.g. "never use dark mode
  — because X"). Claimed effect: repeat-error rate on that class of mistake
  drops each session. Directly applicable to any product where users correct
  the agent repeatedly over time — turns correction into permanent
  personalization instead of a one-off fix.
  *Source: AI Agents Full Course 2026 (28:59–32:53)*

- **Config hierarchy.** Global user-level md → project-level md → skills →
  inline prompt. Lets common behavior live once instead of being repeated per
  project/session.
  *Source: AI Agents Full Course 2026 (33:36–34:44)*

> **Tension worth tracking**: the pattern above (agent appends a rule every
> time it's corrected) will grow a memory file indefinitely. Anthropic's own
> Claude Code team did the opposite on their system prompt — see "Trim
> instructions as models improve" below — cutting it 80% because long lists
> of examples/never-rules were *constraining* a smarter model, not helping
> it. Likely resolution: self-updating rules are good for genuine standing
> preferences; they still need periodic pruning (see "Explain the reason,
> not just the rule" and Boris's `/simplify`-style cleanup under Operator
> habits) rather than growing forever unchecked.
> *Source: How I Plan, Build, and Run Loops with Claude Code (34:26–36:00)*

## Task specification (highest leverage for reliability)

- **Definition of done.** Most user disappointment with agents traces back to
  a missing explicit stop/success condition in the prompt, not model
  capability. Any product surface that takes free-text goals should force this
  out of the user somehow.
  *Source: AI Agents Full Course 2026 (7:06–7:29, 1:21:24–1:21:55)*

- **Prompt contracts.** Before starting non-trivial work on a vague ask, the
  agent drafts a mini contract — goal / constraints / format / explicit
  failure conditions — and gets approval before proceeding. Generalizes
  Claude Code's "plan mode" into a product-agnostic pattern. Implementation:
  a skill that self-analyzes the request and produces the four-section
  contract.
  *Source: AI Agents Full Course 2026 (1:22:27–1:27:16)*

- **Reverse prompting.** Before drafting the contract, have the agent ask
  ~5 dynamically generated clarifying questions to surface assumptions the
  user didn't think to state (aesthetic references, success metric, format
  constraints). Chain: reverse-prompt → prompt-contract → execute. Demoed to
  materially outperform a vague one-shot prompt.
  *Source: AI Agents Full Course 2026 (1:27:39–1:31:28)*

- **Planning is iterative, not a one-shot handoff.** Treat "spec" as something
  that evolves through rounds of explore → investigate → surface unknowns →
  simplify, rather than writing it once and then implementing. Ask the agent
  to keep implementation notes as it builds, and feed surprises back into a
  respec — there's no clean line between planning and implementation.
  *Source: How I Plan, Build, and Run Loops with Claude Code (8:37–9:10, 16:19–17:48)*

- **Have the agent explain the system back to you as part of planning.**
  Before building on top of an unfamiliar piece of tech (e.g. a transcription
  API), ask the agent to explain how it works and where it breaks (edge
  cases, failure modes) — this both builds real confidence and surfaces
  unknown unknowns before they turn into a wasted build.
  *Source: How I Plan, Build, and Run Loops with Claude Code (10:19–11:45)*

- **Prototype cheap before investing expensive.** Build the cheapest version
  that proves out the concept (e.g. an HTML mock) before committing to the
  expensive version (e.g. a full React rebuild). Ask: "what's the smallest
  step that proves out the spec?"
  *Source: How I Plan, Build, and Run Loops with Claude Code (17:56–18:27)*

- **Reference-driven exploration via generated artifacts.** For subjective,
  hard-to-spec asks (design, tone, "make it beautiful"), have the agent
  generate an interactive HTML artifact with several variations to react to,
  optionally seeded with a reference (an existing site/brand) it can fetch.
  Especially useful when the requester "won't know it until they see it."
  *Source: How I Plan, Build, and Run Loops with Claude Code (9:35–10:15, 15:50–16:00)*

## Multi-agent quality control

- **Sub-agent verification loops.** Implementer → fresh-context reviewer →
  fresh-context resolver. An agent grading its own output has sunk-cost bias
  ("I wrote this, so it must be right"); a reviewer given only the output
  (not the reasoning trail) catches what self-review misses. Likely the most
  directly productizable pattern in this set — cheap to build (just a second
  agent call with a clean context window), and it's a general reliability
  multiplier, not task-specific.
  *Source: AI Agents Full Course 2026 (1:13:29–1:20:16)*

- **Stochastic multi-agent consensus.** Spawn N agents on the same task with
  varied framing, run in parallel, aggregate outputs into consensus /
  divergent / outlier buckets. Best fit: ambiguous strategic decisions, not
  routine execution — cost was ~$3–4 for a 10-agent run in the demo.
  *Source: AI Agents Full Course 2026 (58:49–1:07:23)*

- **Agent chat rooms.** Assign distinct personas (systems thinker, pragmatist,
  edge-case finder, user advocate, contrarian) and have agents debate
  round-robin on a shared doc. Claimed to produce sharper, less generic
  conclusions than independent parallel consensus when the problem benefits
  from adversarial framing.
  *Source: AI Agents Full Course 2026 (1:08:00–1:13:03)*

- **Deterministic-check vs. squishy-rubric verification.** If the task has
  something precisely checkable (a Figma file via MCP, a test suite), use a
  goal/exit-condition loop that verifies directly against it. If success is
  subjective (does this screenshot look right?), use a workflow: a rubric
  plus a separate verification agent grading against it — don't try to force
  a precise check onto a fuzzy target.
  *Source: How I Plan, Build, and Run Loops with Claude Code (3:37–4:11)*

- **Separating implementer from verifier also buys more total compute.**
  Beyond reducing self-referential bias, splitting work across separate
  agents (coordinator, implementer, verifier) means each one is less likely
  to stop early — cited as "self-referential bias": a model grading its own
  output tends to be lenient on it, and a single agent juggling several
  things at once tends to under-invest in each.
  *Source: How I Plan, Build, and Run Loops with Claude Code (30:03–30:45)*

## Context & cost engineering

- **Iceberg technique.** Keep only active-task state (memory, current task,
  open files) "above the waterline" in context; fetch everything else
  on-demand via read/grep/glob instead of front-loading it. The load-bearing
  operational pattern for keeping a long-running agent product both cheap and
  high-quality, since claimed quality degrades as context fills up.
  *Source: AI Agents Full Course 2026 (1:44:00–1:45:12, 1:54:56–1:58:59)*

- **Skills via progressive disclosure.** Only a skill's YAML header
  (name/description) sits in context by default; the full body loads only
  when the agent actually invokes it. Relevant if we build a skill/tool
  library for our own agents — lets the library grow without linearly
  growing baseline context cost.
  *Source: AI Agents Full Course 2026 (2:00:33–2:00:58)*

- **60-30-10 model routing.** Route ~60% of token volume to cheap models
  (e.g. Haiku/Flash) for simple tasks, ~30% to mid-tier, ~10% to a frontier
  model for routing decisions and the hardest reasoning. Worked example in
  the source: same volume split across tiers vs. all-frontier cut cost ~60%
  with claimed minimal quality impact. This is the concrete lever for unit
  economics in an agent product — worth prototyping early since it shapes
  pricing viability.
  *Source: AI Agents Full Course 2026 (2:04:53–2:10:22)*

- **Batch API for non-urgent volume.** Bulk requests processed on a delayed
  (e.g. ~1 day) turnaround get a price discount, since the provider schedules
  the work into low-demand inference windows. Applicable to any pipeline step
  that doesn't need real-time latency (enrichment, classification, etc.).
  *Source: AI Agents Full Course 2026 (2:10:51–2:11:55)*

- **Trim instructions as models improve.** Anthropic cut Claude Code's own
  system prompt by 80% — smarter models need fewer examples and fewer
  explicit constraints; long lists of "never do X" examples were found to
  over-fit/constrain behavior rather than help it. Practical implication
  called out directly: CLAUDE.md files and skills are "probably too long
  right now" for most teams and should be trimmed, not grown by default.
  *Source: How I Plan, Build, and Run Loops with Claude Code (34:22–35:37)*

- **Explain the reason, not just the rule.** "Never do X" is a blunter, more
  constraining instruction than explaining *why* X is usually wrong — the
  reasoning generalizes to cases the literal rule didn't anticipate, and a
  smarter model can apply judgment instead of pattern-matching a fixed list.
  Give principles + the few real hard constraints (e.g. an actual character
  limit), not a wall of examples.
  *Source: How I Plan, Build, and Run Loops with Claude Code (35:37–37:01)*

## Orchestration architecture

- **Router/orchestrator pattern.** One model plans, decomposes, delegates
  sub-tasks to whichever model/tool is best suited, and validates
  integration; specialist agents execute via MCP. This is the shape most
  "agent products" converge on. Caveat: routing across multiple model
  providers usually costs more per-token than staying on one plan, since you
  lose the subsidized-usage benefit of a flat-rate subscription.
  *Source: AI Agents Full Course 2026 (39:14–46:41)*

- **Parallel execution for repetitive multi-step actions.** Giving each of N
  agents its own execution environment (e.g. browser instance) for the same
  repeated action (form fill, scrape) scales throughput roughly linearly.
  Relevant for any product doing per-target volume work (outreach, scraping,
  data entry).
  *Source: AI Agents Full Course 2026 (1:32:59–1:40:56)*
  *Caveat noted in source: overlaps with anti-bot-detection evasion territory
  (browser fingerprinting) — needs its own legal/ethics review before
  productizing, not just a technical green light.*

## Skills as reusable infrastructure

- **A skill can be "how to build the workspace," not just "how to do the
  task."** Beyond packaging a procedure, a skill can be instructions for
  scaffolding a reusable workspace (scripts, folder structure, config) —
  the more of that scaffolding exists, the less the agent has to rebuild
  from scratch on each run.
  *Source: How I Plan, Build, and Run Loops with Claude Code (25:35–26:06)*

- **Chain skills into pipelines.** Compose small, single-purpose skills
  (transcribe → generate thumbnails → cut clips) rather than one skill that
  tries to do everything — the presenter explicitly flagged his own
  everything-skill as an anti-pattern to fix.
  *Source: How I Plan, Build, and Run Loops with Claude Code (23:56–24:24, 28:20–28:24)*

- **Workflows vs. skills, concretely.** A workflow is what you reach for when
  a skill alone isn't enough: it can spin up parallel sub-agents (keeping
  each one's context clean) and attach a verification rubric per sub-agent
  — e.g. generating N video shorts in parallel, each judged against the same
  rubric by an independent agent. A workflow can itself be saved back into a
  skill for reuse.
  *Source: How I Plan, Build, and Run Loops with Claude Code (27:34–30:03)*

## Collaboration surfaces (human↔agent, agent↔agent)

- **Chat platforms (e.g. Slack) as the "multiplayer" agent surface.**
  Terminal/IDE agent sessions are single-player; a chat platform is where
  humans naturally already collaborate, so it becomes the default surface for
  async, multi-agent, or multi-human-plus-agent work — each channel/thread
  can carry its own agent memory, similar to how a coding agent's threads
  work, but visible to a team instead of one person.
  *Source: How I Plan, Build, and Run Loops with Claude Code (20:44–21:20, 22:19–22:26)*

- **"Babysit the PR" pattern.** Tell the agent to watch a PR, fix failing
  tests, and tag a human reviewer when it's ready — all inside the same chat
  channel, so the reviewer gets pulled in at exactly the right moment instead
  of the requester having to manually hand off.
  *Source: How I Plan, Build, and Run Loops with Claude Code (19:46–19:54)*

- **Reserve one active foreground session; push the rest to background/chat.**
  The team's working pattern: one focused, iterative session for the thing
  you're actively driving, with everything else (initial exploration, specs,
  background jobs) delegated to parallel chat-based agent threads rather than
  juggling many foreground sessions at once.
  *Source: How I Plan, Build, and Run Loops with Claude Code (18:45–19:35)*

## Operator habits (how the humans using these agents actually work)

- **Single-focus discipline beats heavy multitasking.** Even with many
  background agent threads running, keep one primary task as your actual
  focus — the real time-cost isn't parallelism, it's writing a lazy prompt
  to save 30 seconds and paying for it later in wasted agent time.
  *Source: How I Plan, Build, and Run Loops with Claude Code (31:45–32:23)*

- **Periodic repo/workspace cleanup is for the human, not just the agent.**
  Agent-generated output accumulates cruft ("the whole repo turns to slop")
  especially when outputs aren't read carefully; schedule a cleanup/simplify
  pass rather than assuming the agent self-organizes. (We already have a
  `/simplify`-style skill available in this environment — worth using
  deliberately, not just reactively.)
  *Source: How I Plan, Build, and Run Loops with Claude Code (33:11–33:56)*

- **"Becoming more technical" with agents means learning unknown unknowns,
  not syntax.** The valuable skill isn't knowing a language's syntax; it's
  knowing the trade-offs and constraints of the systems you're building on
  (which backend, which library, why) so you can direct the agent well and
  catch when it's wrong. Push the agent to teach you this deliberately — it
  takes real effort, similar to Karpathy's point that education should feel
  like work, not entertainment.
  *Source: How I Plan, Build, and Run Loops with Claude Code (37:30–39:01)*

---

## Sources

1. **"AI Agents Full Course 2026: Master Agentic AI"** (YouTube, 2h13m,
   https://www.youtube.com/watch?v=EsTrWCV0Ph4) — broad survey of agent
   architecture, multi-agent orchestration, prompt engineering, and context/
   cost management across Claude Code, Codex, and Antigravity. Single
   creator's opinionated synthesis, not peer-reviewed — treat specific
   numbers (cost figures, "percentage point" quality claims) as illustrative,
   not benchmarked.

2. **"Claude Code Tutorial - Build Apps 10x Faster with AI"** (YouTube, first
   hour of a 9-hour paid course, https://www.youtube.com/watch?v=IuyVVtr1uhY)
   — hands-on Claude Code fundamentals: prompting, CLAUDE.md, plan mode,
   context window management (`/clear` vs `/compact`), and a pragmatic take
   on MCP server sprawl. Complements source 1's higher-level patterns with
   concrete day-to-day workflow habits.

3. **"How I Plan, Build, and Run Loops with Claude Code in 40 Minutes"**
   (YouTube interview with Thariq Shihipar, Claude Code team at Anthropic,
   41m, https://youtu.be/aVO6E181cNU) — higher trust than sources 1–2: this
   is an Anthropic Claude Code team member describing how the team actually
   works internally, not a third-party creator's course. Covers `/loop`,
   `/goal`, and workflows as the three primitives for long-running agent
   work; iterative (not one-shot) planning; skills-as-workspace-scaffolding;
   Claude Tag/Slack as the multiplayer collaboration surface; and why
   Anthropic trimmed Claude Code's system prompt 80% as models got smarter —
   which directly complicates the "self-updating rule file" pattern from
   source 1 (see the tension note under Memory & self-improvement).
