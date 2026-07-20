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

- **Memory is not automatic — it must be explicitly wired in.** n8n's "AI
  Agent" node has zero memory by default; asking it about an earlier message
  in the same conversation fails until a memory component is attached
  (default window: 5 messages). Worth treating as a checklist item for any
  agent product: don't assume conversational continuity exists just because
  the framework has an "agent" primitive — verify it, and size the window
  deliberately.
  *Source: N8N Full Course 6 Hours (142:34–144:20)*

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

- **Schema-first structured output.** Prompt the model with the exact JSON
  keys/shape you want back, rather than asking for free text and parsing
  after the fact. Used consistently across a course's real, sold client
  workflows — treat this as a default, not an optimization.
  *Source: N8N Full Course 6 Hours (85:36–85:47, 131:10–131:53, 178:00–178:11)*

- **Classify-then-generate in one call.** Have a single LLM call return both
  a gate field (e.g. `relevant: true/false`) and the conditionally-generated
  content in one JSON response, instead of a separate classification call
  followed by a separate generation call. Cheaper and simpler than two
  round-trips when the two steps are always paired anyway.
  *Source: N8N Full Course 6 Hours (177:02–178:11)*

- **Bounded task → deterministic control flow + single LLM call. Open-ended
  task → full agent (memory + tools).** Observed as consistent practice
  rather than a stated rule: the course's two *sold* client workflows
  (single-purpose transformations) both use one-shot LLM calls wired through
  plain if/filter/merge logic; the full agent node (memory, autonomous tool
  selection) is reserved for genuinely open-ended, multi-turn tasks. Useful
  as a default heuristic for not over-agentifying simple steps.
  *Source: N8N Full Course 6 Hours (73:03–74:34 vs. 140:41–146:40)*

- **Gate side effects behind classification, not behind every item.** Filter
  *after* generation using an AI-returned boolean field, so anything with a
  real-world side effect (sending an email, drafting a reply) only fires for
  items the model actually flagged as relevant — keeps irreversible actions
  behind a checkpoint instead of firing on every record in a batch.
  *Source: N8N Full Course 6 Hours (183:52–184:28)*

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

- **Quick token-budget heuristic.** Rough rule of thumb: `characters / 4.7 /
  0.75 ≈ tokens` (≈4.7 chars/word, ≈0.75 words/token in English). Not
  precise enough to bill against, but cheap enough to sanity-check a
  prompt/context chunk's cost before sending it, without calling a tokenizer.
  *Source: N8N Full Course 6 Hours (200:00–203:01)*

- **Strip boilerplate before it reaches the model.** Remove HTML tags,
  markdown formatting, and other non-signal content from scraped/fetched
  text before it hits an LLM call — cheaper input tokens and cleaner
  downstream matching/search. A specific instance of "don't pay to process
  waste."
  *Source: N8N Full Course 6 Hours (221:01–222:29)*

- **Pin/cache expensive outputs during iterative development.** Freeze an
  LLM (or any upstream) call's sample output while building/testing
  downstream steps, instead of re-triggering and re-paying for the real call
  on every test run. Test on a small slice (1–3 records) before scaling to
  the full batch, for the same reason — avoid burning tokens on iteration
  that doesn't need live data.
  *Source: N8N Full Course 6 Hours (33:52–34:24, 100:53–100:59, 179:56–180:22)*

- **Rate-limit-aware sequential looping.** When hitting per-item API/rate
  limits on bulk operations, loop one item at a time with a short wait
  between iterations rather than firing all requests concurrently. A
  recurring real-world failure mode, not an edge case — worth building in
  by default for any bulk-processing pipeline, agent-driven or not.
  *Source: N8N Full Course 6 Hours (161:53–163:09)*

- **Self-hosting flips the cost model from per-execution to fixed-cost.** A
  small fixed-cost server can absorb volume that would cost far more on a
  metered cloud plan — directly relevant to margin planning for any agent
  product with unpredictable or high-volume usage. Separately: data
  residency/compliance requirements (healthcare, legal, financial) can force
  self-hosting regardless of cost, making it a default rather than a
  preference in regulated verticals.
  *Source: N8N Full Course 6 Hours (305:07–305:34, 305:43–306:16)*

- **Meter by execution, not by internal step.** Concrete numbers from one
  platform comparison: a plan that bills per full workflow-run scaled ~40x
  further per dollar (self-hosted) than a comparable plan billing per
  internal operation/node, because a single run can contain many internal
  steps. Worth remembering when pricing our own agent product's usage —
  metering granularity changes the economics more than the sticker price
  does.
  *Source: N8N Full Course 6 Hours (347:56–353:52)*

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

- **Multi-agent threshold heuristic: past ~6-7 tools on one agent, spawn a
  sub-agent instead of stacking more tools.** A flat single-agent-with-many-
  tools design is claimed to degrade in tool-selection accuracy past that
  point; the fix is to have the agent call *another* agent that owns its own
  smaller tool set and makes its own tool-selection decision — described as
  "almost like a big search tree." One of the more concrete, actionable
  numbers in any source so far for when to introduce hierarchy.
  *Source: N8N Full Course 6 Hours (146:24–146:40)*

- **Let the agent decide runtime tool-call values, not the workflow author.**
  When a tool needs a dynamic value (e.g. a date range for a calendar
  query), bind it to an "ask the model" expression rather than hardcoding it
  — the agent fills in the value at call time based on context. The general
  version: don't pre-compute what the agent is actually positioned to reason
  about itself.
  *Source: N8N Full Course 6 Hours (145:31–146:03)*

- **Applied shape: unstructured content → structured record via one LLM
  call.** Fetch raw content (e.g. scrape a webpage's text), then a single
  LLM call with a JSON-schema prompt turns it into a structured record
  (summary, key attributes, contact info, etc.). A reusable "ingest →
  structure" shape worth having as a building block before reaching for
  dedicated scraping/RAG tooling.
  *Source: N8N Full Course 6 Hours (129:01–133:19)*

- **Guard side-effecting nodes against accidental fan-out re-execution.**
  When a downstream node has a real-world side effect (send email, write
  record) and sits after a fan-out over multiple items, explicitly guard it
  to run once per intended trigger, not once per upstream item — an easy
  footgun once a pipeline has more than one branch.
  *Source: N8N Full Course 6 Hours (165:59–166:14)*

## Tool selection for agent-building infrastructure

Patterns for evaluating/choosing the tools we build agent products *with* —
distinct from patterns for how the agents themselves behave.

- **Check whether a tool's business model is structurally opposed to using
  it efficiently.** A no-code platform that meters per-operation has a built-
  in incentive not to offer an easy code escape hatch, since code lets users
  collapse many billable steps into one. Before adopting a metered tool,
  check whether efficient usage is something the vendor is incentivized to
  make hard.
  *Source: N8N Full Course 6 Hours (330:36–330:57)*

- **Check for native branch/loop/merge/error-handling primitives, not
  simulated ones.** Orchestration tools vary widely in whether control flow
  is a first-class primitive (native switch, native loop, native error
  handling) vs. something you have to hand-simulate by chaining filters.
  This matters a lot more for agent orchestration specifically than for
  simple linear automations, since agent flows branch and retry constantly.
  *Source: N8N Full Course 6 Hours (331:42–336:37)*

- **A fast pin/cache-and-rerun debug loop is worth weighting heavily in tool
  choice.** The ability to freeze sample data on a step and rerun only
  downstream steps against it (vs. re-hitting live APIs on every test) was
  called out as an order-of-magnitude difference in iteration speed between
  two compared platforms — arguably more consequential than raw feature
  count when picking a build tool.
  *Source: N8N Full Course 6 Hours (335:26–336:37)*

- **Tooling purpose-built around the agent primitive beats general-purpose
  automation tooling, once you're actually building agents.** A platform
  with a native agent construct (memory, tool-calling, agent-as-callable-
  tool) beat a general iPaaS competitor for agent-building specifically, even
  though the competitor had more integrations overall — capability that
  matches the actual primitive you're building with matters more than raw
  breadth.
  *Source: N8N Full Course 6 Hours (340:03–342:36)*

- **Text-representable, pasteable definitions + native in-canvas
  documentation = maintainability.** Workflows/specs that export as
  plain-text (JSON, markdown) and can be pasted/imported directly, with
  documentation embedded next to the logic rather than hidden behind a
  click, are what makes something handoff-able to a team later. Relevant to
  how we structure our own agent configs/specs, not just to picking a
  vendor.
  *Source: N8N Full Course 6 Hours (342:44–346:54)*

- **Simple tools win the early/non-technical segment of a market; power
  tools pull ahead as the median builder's skill rises.** Drawn as an
  explicit analogy to the Zapier→Make→n8n progression. Worth revisiting
  periodically as a lens on how our own product's target user — and the
  tooling they reach for — might shift over time.
  *Source: N8N Full Course 6 Hours (353:55–354:32)*

## Productizing & selling agent automations

Business/product-delivery patterns, as distinct from technical-build
patterns — from a course explicitly about building *and selling* automations
to clients.

- **Template-first delivery.** A large public template library can be a
  legitimate fast path to shipping client/product work, not just a learning
  aid — start from a close-enough template and customize, rather than
  building every workflow from scratch.
  *Source: N8N Full Course 6 Hours (05:01–05:54)*

- **Humanize automated timing.** Insert a deliberate delay (e.g. ~2 minutes)
  before sending an AI-drafted response, specifically so automated outreach
  reads as human-paced rather than obviously instant/robotic. A small,
  cheap lever for perceived quality.
  *Source: N8N Full Course 6 Hours (52:19–52:51)*

- **Perceived personalization is the sellable value, not automation
  volume.** The pitch for a cold-outreach workflow wasn't "sends more
  emails" — it was that paraphrasing a prospect's own stated interests back
  to them made outreach *read* as individually researched. Worth remembering
  when framing what an agent product's value prop actually is to a buyer.
  *Source: N8N Full Course 6 Hours (100:41–100:57)*

- **Strip platform attribution branding for client/business use.** Small but
  repeated polish note: default "sent via [tool]" branding on outbound
  communications should come off for anything client-facing — a cheap
  professionalism signal that's easy to forget.
  *Source: N8N Full Course 6 Hours (24:28–24:46)*

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

- **Use the platform's own AI assistant as a first debugging pass.** Before
  manual troubleshooting, check whether a built-in AI helper resolves the
  issue — claimed to clear 30–40% of one builder's own questions. Cheap
  triage step worth defaulting to.
  *Source: N8N Full Course 6 Hours (11:15–11:28)*

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

4. **"N8N FULL COURSE 6 HOURS (Build & Sell AI Automations + Agents)"**
   (YouTube, ~6h, https://www.youtube.com/watch?v=2GZ2SNXWK-c) — a
   practitioner course on n8n (visual workflow-automation tool) covering
   both building AI agent pipelines and selling them as client work. Most of
   the runtime is n8n-specific UI/JS mechanics (skipped from this library);
   the substantive chapters (agent nodes/memory/tools, JS-function
   utilities, self-hosting vs. cloud, and an n8n-vs-Make comparison)
   contributed real content in three new areas: concrete agent-orchestration
   heuristics (the tool-count threshold is the most specific number in this
   library so far), a new "tool selection for agent-building infra" category
   (evaluating the tools we'd build *with*, not just agent behavior itself),
   and a new "productizing & selling" category. Single practitioner's
   opinionated take with real client-work examples, not benchmarked — treat
   specific numbers (pricing, thresholds, ratios) as illustrative rather
   than load-bearing.
