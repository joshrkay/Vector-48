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
