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

- **Concrete Claude Code memory file hierarchy.** `~/.claude/CLAUDE.md`
  (personal, global, applies across all workspaces) → `.claude/CLAUDE.md`
  (per-project) → an enterprise-managed system tier for orgs, with global
  superseding local on conflict. Team pattern: a lead owns global rules
  (naming conventions, "don't touch X"), individual engineers own their
  local file. Fills in real mechanics behind the "config hierarchy" entry
  above. Separately, `CLAUDE.local.md` / `settings.local.json` are gitignored
  siblings of `CLAUDE.md` / `settings.json`, for machine-specific
  instructions and secrets that shouldn't be shared via the repo.
  *Source: Claude Code Full Course 4 Hours (56:30–57:33, 63:15–66:00)*

- **A `rules/` folder can replace a monolithic CLAUDE.md.** Split
  instructions into topic-scoped files (code-style, testing, security,
  frontend) instead of one growing file — enables giving a teammate edit
  access to one rule file without the whole memory file, and keeps each file
  short enough to stay high-signal. Demoed by asking Claude itself to split
  an existing CLAUDE.md into rules files.
  *Source: Claude Code Full Course 4 Hours (59:12–62:51)*

- **`memory.md` is a separate, auto-managed file from CLAUDE.md.** When you
  tell Claude conversationally to "remember" something, it's written to an
  auto-injected `memory.md`, distinct from CLAUDE.md — lower-value for
  steering behavior, more useful as ambient personalization/fact storage
  (demoed: telling one session a personal fact, confirming a brand-new
  session recalls it unprompted). Worth tracking as a separate layer from
  the "self-updating rule file" pattern above, not a duplicate of it.
  *Source: Claude Code Full Course 4 Hours (75:24–77:32)*

- **Primacy/recency applies to memory-file structuring.** Put
  non-negotiable constraints at the very top of a CLAUDE.md/system prompt —
  content in the middle of a long file/prompt gets less attention than the
  start or end. A concrete, general-purpose heuristic for structuring any
  long memory file, not Claude-Code-specific.
  *Source: Claude Code Full Course 4 Hours (70:33–71:27)*

- **Prune memory files like technical debt, on a cadence.** Complements the
  tension noted above (self-updating rules vs. Anthropic's 80% system-prompt
  trim): this source independently confirms both directions — keep letting
  Claude add corrective rules after it repeats a mistake 2-3 times, *and*
  periodically review and cut accumulated cruft. Practical ceiling cited:
  ~200-500 lines before a CLAUDE.md starts hurting more than helping. Also:
  never paste raw API docs/full style guides in — have the agent distill
  only the relevant slice.
  *Source: Claude Code Full Course 4 Hours (71:27–74:17)*

> **Nuance, not quite a tension**: a different chapter of this same source
> treats a project-specific CLAUDE.md as low-effort and disposable — content
> AI-generated from scraped reference material, explicitly called "not super
> important," functionally interchangeable with a one-off pasted message.
> Read together with the pruning/hierarchy guidance above, the likely
> resolution: a narrow, single-purpose project file can be throwaway
> boilerplate; the general-purpose, cross-session memory file is where the
> "curate it carefully" advice actually applies.
> *Source: Claude Code Full Course 4 Hours (30:12–30:41)*

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
  **✅ Prototyped:** `.claude/skills/intake-contract/` — applied to client
  intake specifically. Update this entry (and that skill) together if either
  changes based on real use.

- **Reverse prompting.** Before drafting the contract, have the agent ask
  ~5 dynamically generated clarifying questions to surface assumptions the
  user didn't think to state (aesthetic references, success metric, format
  constraints). Chain: reverse-prompt → prompt-contract → execute. Demoed to
  materially outperform a vague one-shot prompt.
  *Source: AI Agents Full Course 2026 (1:27:39–1:31:28)*
  **✅ Prototyped:** built into `.claude/skills/intake-contract/` as Step 2.

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

- **AI's edge is iteration speed, not one-shot precision.** A human doing a
  single careful pass is often *more* precise per-attempt than a model — the
  value comes from cycling a task→verify loop many times fast (e.g.
  80%→90%→95%→99% quality in minutes) rather than getting it right in one
  shot. Reframes "definition of done" and verification patterns as the
  mechanism that actually captures this speed advantage — without a verify
  step, most of the value is left on the table.
  *Source: Claude Code Full Course 4 Hours (41:13–41:53)*

- **Plan mode as "shift correctness left."** Spend time catching a wrong
  approach while it only costs a rewrite of a plan document, not a rewrite
  of the built thing — "a minute of planning saves you 10 minutes of
  building." Concretely demoed as: unstructured brain-dump → Claude responds
  with multiple-choice clarifying questions on the ambiguous parts →
  disambiguated plan. This is effectively reverse-prompting implemented
  natively in Claude Code's plan mode, independently confirming that pattern.
  *Source: Claude Code Full Course 4 Hours (101:12–108:00)*

- **Treat the agent as a literal specialist, not a motivatable generalist.**
  Vague aspirational instructions ("be smart," "make no mistakes") don't
  work the way they might on a person — specificity beats exhortation.
  Direct complement to "explain the reason, not just the rule" above: give
  principles and real constraints, but don't substitute vibes for either.
  *Source: Claude Code Full Course 4 Hours (72:58–73:37)*

## Multi-agent quality control

- **Sub-agent verification loops.** Implementer → fresh-context reviewer →
  fresh-context resolver. An agent grading its own output has sunk-cost bias
  ("I wrote this, so it must be right"); a reviewer given only the output
  (not the reasoning trail) catches what self-review misses. Likely the most
  directly productizable pattern in this set — cheap to build (just a second
  agent call with a clean context window), and it's a general reliability
  multiplier, not task-specific.
  *Source: AI Agents Full Course 2026 (1:13:29–1:20:16)*
  **✅ Prototyped:** `.claude/skills/verify-before-ship/` — pre-delivery
  quality gate for client work, including the resolver step on blocking
  findings. Update this entry (and that skill) together if either changes
  based on real use.

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

- **Why fresh-context review works: it's not just "more eyes," it's the
  absence of anchoring bias.** The strongest causal explanation yet for the
  sub-agent-verification-loop pattern above: an agent that just wrote code
  carries a long reasoning trail that self-justifies its own choices, so it
  can't evaluate its own work objectively. A blank-context reviewer has no
  such trail to defend, and catches things (e.g. "why was it done this way?")
  the author-agent structurally cannot see about itself. Explicit human
  analogy: a developer who's been heads-down for hours needs an outside
  reviewer for the same reason.
  *Source: Claude Code Full Course 4 Hours (82:05–84:29)*

- **What "squishy vs. deterministic" verification actually hinges on: the
  artifact type, not just how strict you want to be.** Sharpens the
  deterministic-vs-rubric entry above — visual/design work can be verified
  cheaply via screenshot comparison; backend/logic work needs an actual
  behavioral check (tests), since you can't eyeball whether claimed behavior
  really happens. The axis is "what kind of artifact is this," which
  determines what verification tooling is even possible, before you get to
  choosing rigor.
  *Source: Claude Code Full Course 4 Hours (85:23–85:36)*

- **Verification pipelines are a cost/rigor dial, not a default.** Explicit
  caveat on the implementer→reviewer→resolver pattern: valuable mainly at
  enterprise scale (security-sensitive, edge-case-heavy work) or when a
  human needs a legible review artifact; skip it for internal tooling or
  low-stakes one-off work. Complements "spin-up overhead can exceed the
  savings for trivial tasks" — the same caution as the ~6-7-tool
  sub-agent threshold, applied to verification specifically, not just tool
  delegation.
  *Source: Claude Code Full Course 4 Hours (201:53–202:38, 206:19–206:36)*

- **Fan-out reliability degrades multiplicatively — do the math before
  scaling parallel agents.** If each parallel agent has a 95% success rate,
  10 running concurrently succeed *together* only ~59% of the time; 50
  concurrent drops to ~7%. Concrete argument for keeping each sub-agent's
  task definition narrow (e.g. "just classify," not "classify + merge +
  apply") rather than loading more responsibility onto each one as you
  scale fan-out width.
  *Source: Claude Code Full Course 4 Hours (197:17–198:24)*

- **Adversarial debate recipe for auditing existing work (concrete
  template).** Scan → synthesize → adversarial pair debates → targeted
  fixers: N "scanner" agents partition and read a large artifact in
  parallel, a smaller number of agents compile findings, then an explicitly
  opposed pair (one arguing a finding is real, one arguing false-positive)
  debates until a lead judges consensus, then one fixer agent per confirmed
  issue (with instructions to negotiate if two fixers' edits collide).
  Explicitly likened to GANs (generator vs. discriminator) as the underlying
  justification. More structured and reusable than a generic "have two
  agents disagree" — worth treating as a template, not just a technique.
  *Source: Claude Code Full Course 4 Hours (229:59–234:03)*

- **Parallel variant generation + selective iteration, as a generative (not
  just verification) multi-agent pattern.** Spin up N agents to independently
  generate fundamentally different outputs from one brief, pick a favorite,
  spin up N more to iterate variations on the winner, repeat. A general
  search/exploration strategy — applicable beyond design to any generative
  task where "search the space of possible outputs" beats iterating on a
  single lineage.
  *Source: Claude Code Full Course 4 Hours (215:29–228:07)*

- **Iterative refinement via fresh/blind instances applies to building
  skills, too — not just reviewing code.** Draft a skill → run it on a fresh,
  context-free Claude Code instance to see how it performs blind → give
  corrective feedback → repeat. Cited progression from ~70% to ~98-99%
  reliability. Same underlying mechanism as the reviewer-subagent pattern
  (fresh eyes catch what the author can't) applied to testing instructions
  themselves, not just checking output.
  *Source: Claude Code Full Course 4 Hours (89:51–90:36)*

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

- **There's a fixed token tax before you send a single message.** Concrete
  numbers from one Claude Code session: system tools alone cost roughly
  17,000 tokens on a 200K window, non-negotiable overhead regardless of
  task. With MCP tools, system prompt, and memory files also loaded, total
  pre-message baseline reached ~45,000 tokens (~22.5% of the window) before
  any real work started — in the highest-attention region of the context
  window, i.e. the worst place to waste tokens. Worth benchmarking our own
  agent harness's baseline tax the same way.
  *Source: Claude Code Full Course 4 Hours (136:39–138:57, 179:00–180:00)*

- **Skills vs. MCP tools, with hard numbers.** A skill's frontmatter costs
  roughly 58-63 tokens; a single MCP tool definition can cost more than all
  of a builder's skills combined (example cited: ~1,600 tokens for one MCP
  search tool). Confirms and quantifies the "skills via progressive
  disclosure" entry above — the cost gap isn't marginal, it's roughly two
  orders of magnitude per unit.
  *Source: Claude Code Full Course 4 Hours (179:42–180:00)*

- **MCP tool-search kicks in automatically past a real threshold.** Once MCP
  tool descriptions exceed ~10% of the context window, Claude Code stops
  preloading every tool definition and instead does an on-demand
  search/lookup for just the relevant tool before loading it. A specific,
  actionable mechanism — not just "MCP can be expensive," but the actual
  point at which the system compensates for it automatically.
  *Source: Claude Code Full Course 4 Hours (151:07–151:31)*

- **`/context` and `/cost` as standing audit habits.** `/context` gives a
  literal category breakdown (system prompt, system tools, MCP tools,
  memory, skills, messages, free space) with token counts — concrete enough
  to spot a bloated MCP server (one install eating ~20% of the window was
  cited as a real, quality-degrading case) rather than guessing. `/cost` is
  called out as Anthropic's own top proactive-check recommendation.
  *Source: Claude Code Full Course 4 Hours (134:39–136:02, 148:13–151:05)*

- **Concrete auto-compact mechanics, plus a verification habit.**
  Auto-compaction triggers when remaining context drops to a reserved
  buffer (~33,000 tokens cited), and now runs continuously in the
  background rather than only firing once at a threshold; `/compact` accepts
  a custom prompt telling it what to prioritize keeping. Verification habit
  worth adopting: immediately after a compact, ask the model to state what
  it retained, as a sanity check that nothing important got silently
  dropped.
  *Source: Claude Code Full Course 4 Hours (145:13–149:52)*

- **Pre-compress noisy input with a cheap model before it reaches the main
  agent.** For voice-dictated prompts specifically: pipe the raw (filler-
  word-laden) transcript through a cheaper model first to distill it to a
  tight, high-density request, *then* send that to the primary coding agent.
  A pre-processing relay pattern distinct from task-complexity-based model
  routing (like 60-30-10 above) — this one's about input quality, not task
  difficulty.
  *Source: Claude Code Full Course 4 Hours (143:54–144:22)*

- **Sub-agents as a ~50x cost-isolation lever for open-ended research.**
  Offload exploratory research (which could burn 50-100K tokens investigating)
  into a sub-agent so only a short summary (cited: ~2K tokens) returns to
  the parent — lets you route the actual searching to a cheaper model too,
  since research doesn't need frontier-level reasoning. A concrete number to
  anchor the general "context isolation reduces cost" claim.
  *Source: Claude Code Full Course 4 Hours (80:02–82:05)*

- **Agent teams cost roughly 7x more tokens than a single session — with a
  real number attached.** Each teammate in a multi-agent "team" (as opposed
  to a single-shot sub-agent) keeps its own full context window, and cost
  scales with team size and coordination overhead. Concrete example: a
  10-agent security-audit team burned ~1.3M tokens (~$80) in under 15
  minutes for just the scan+debate phase, before any fixes were applied.
  Explicit decision rule: reserve agent teams for high-value, time-boxed
  research/audit work where the time saved is worth more than the (large,
  non-refundable) token spend — not routine generation tasks.
  *Source: Claude Code Full Course 4 Hours (152:41–152:57, 212:12–212:24, 228:26–229:12, 234:39–235:19)*

- **Front-load exploration into the planning pass, not the execution pass.**
  API calls/web fetches are one of the biggest token sinks; doing exploration
  once during plan mode (rather than repeatedly rediscovering the same
  things mid-build) is a direct token-saving move, per the source's citation
  of Anthropic's own guidance.
  *Source: Claude Code Full Course 4 Hours (153:19–153:37)*

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

- **Prototype in MCP, then port to a skill once proven.** A two-step
  methodology, new territory beyond what we had for skills/MCP: use an MCP
  server to quickly validate a new integration is feasible (low setup cost,
  minimal config), and once it's proven useful, have the agent rebuild the
  same capability as a skill calling the underlying API/scripts directly
  (using existing skills as a formatting reference). Treats MCP as
  disposable scaffolding rather than the production path. Concrete before/
  after example: an MCP-based email-labeling flow was slow; the ported
  skill processed 100 emails in 36 seconds using direct API calls.
  *Source: Claude Code Full Course 4 Hours (180:00–187:54)*

- **A three-stage maturity ladder for turning agent work into a product:
  skill → sub-agent → hosted endpoint.** A skill packages *what* to do; a
  sub-agent packages *who* does it in an isolated, parallelizable context; a
  hosted HTTP endpoint (wrapping the same skill/sub-agent) makes the
  capability externally/business-accessible — usable from a browser, curl,
  or wired as a webhook target into other automation platforms. Demoed as a
  same-day step, not a big lift, once the underlying skill already works.
  Directly relevant to how we'd package anything we build here for a client
  or product surface.
  *Source: Claude Code Full Course 4 Hours (247:56–250:11)*

- **Agent Teams: a distinct orchestration primitive from sub-agents, not
  just "more parallelism."** Sub-agents are single-shot delegation — a
  sub-agent owns its own context but reports back to the caller and can't
  talk to siblings. An agent team is a peer structure: a team lead spawns
  teammates that each get a *full independent* agent instance (own memory
  file, own MCP servers, own skills — not just what the lead hands them),
  can message each other directly rather than only reporting up, and
  coordinate via a shared task list. This is the vendor-native, productized
  version of the "agent chat room" pattern above — worth treating as a
  concrete implementation of that idea rather than a separate concept. One
  practitioner's explicit pushback worth logging too: he calls teams "more
  of the same... just another way to parallelize," skeptical of
  categorical-paradigm-shift framing — treat vendor claims about teams-vs-
  sub-agents boundaries as marketing, fuzzy in practice.
  *Source: Claude Code Full Course 4 Hours (206:44–212:24)*

- **Official guidance for using agent teams economically (relayed, not
  independently verified).** Use a cheaper model for teammates; keep teams
  small since cost scales with team size; keep spawn prompts focused, since
  everything in a spawn prompt adds to that teammate's starting context; let
  teammates auto-load their own memory/MCP/skills rather than re-explaining;
  explicitly tear down teams when done since idle agents still burn compute.
  Gated behind an experimental opt-in flag specifically to prevent
  inexperienced users from accidentally burning large sums.
  *Source: Claude Code Full Course 4 Hours (212:26–214:39)*

- **Skills vs. sub-agents, the actual decision rule.** A skill hands
  instructions to the *same* parent agent — no context isolation, parent
  keeps full context. A sub-agent is a separate instance with separate
  context. Use a skill when you want the parent to retain context while
  following a procedure; use a sub-agent when you want context/cost
  isolation. Cleaner and more concrete than a general "skills vs. workflows"
  framing — this is the specific mechanical difference.
  *Source: Claude Code Full Course 4 Hours (85:37–86:29)*

- **Git worktrees: the concrete mechanism (and rationale) for conflict-free
  parallel agent work.** Each worktree is a separate working directory
  checked out to its own branch, sharing git history but not files, so
  multiple agent instances can each work in a physically separate folder
  without touching the main folder or each other's in-flight changes, then
  merge back once done. The rationale matters more than the mechanic: real
  codebases rarely have clean file/functionality boundaries, so parallel
  agents sharing one folder *will* eventually collide on a shared file —
  worktrees eliminate that possibility by construction, at the cost of an
  explicit merge step afterward (itself a good task to hand to an agent).
  Framed as the older, manual predecessor to native agent teams — "basically
  what agent teams are today" before that feature existed — and still worth
  composing *with* teams/sub-agents as an extra isolation layer.
  *Source: Claude Code Full Course 4 Hours (236:10–242:39, 245:23–245:39)*

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

- **A blunt unifying mental model: MCP tools are skills someone else built.**
  Skills are self-authored, flexible procedural instructions; MCP is the
  same underlying idea, outsourced to a developer/vendor. Useful one-liner
  for explaining the two side by side rather than treating them as
  unrelated concepts.
  *Source: Claude Code Full Course 4 Hours (171:04–171:40)*

- **Customize the default research capability rather than trusting generic
  web search.** Claude Code ships a default research sub-agent; the
  recommendation is to point it at trusted sources / preferred APIs rather
  than leaving it to generic search — makes research repeatable and
  higher-signal, relevant if we build our own research-heavy agent flows.
  *Source: Claude Code Full Course 4 Hours (203:56–204:04)*

- **A specific tool endorsement worth trying: Chrome DevTools MCP** (built
  by the Claude Code team) for direct browser control/inspection — claimed
  ~100x faster than generic agent-driven click-and-screenshot browser
  interaction, and composable into skills for repeatable browser-automation
  flows. Unverified performance claim, but a concrete lead if we need
  browser automation.
  *Source: Claude Code Full Course 4 Hours (172:30–173:51)*

- **MCP source trust is a real concern, not paranoia.** Third-party MCP
  server directories (mentioned: mcpservers.org, modelcontextprotocol.io/
  servers, mcpmarket) are explicitly flagged as unvetted listings — treat
  installing an MCP server with the same scrutiny as installing any other
  third-party code with tool-calling access, which is the same posture we
  already took when installing the `watch` skill itself.
  *Source: Claude Code Full Course 4 Hours (172:03–172:27)*

- **Plugins are a thin, optional packaging layer — treat as leads, not a
  category to build around yet.** One practitioner's stance: plugins
  (installed via a marketplace) are rarely necessary — "vanilla Claude Code
  does really well without extensions" — and he predicts plugins will
  likely get absorbed into skills over time as the more durable primitive.
  Explicitly speculative, but worth revisiting before investing in a
  plugin-based distribution strategy for anything we build.
  *Source: Claude Code Full Course 4 Hours (188:15–191:09)*

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

- **Security/legal hygiene before shipping anything agent-built publicly.**
  Three concrete habits from a practitioner who ships client work: never use
  a short/guessable URL for a public deployment (bots scan DNS ranges and
  probe fast — cited 30-40 unauthorized hits on a freshly-launched demo, use
  a non-trivial custom domain instead); get a paid third-party security
  audit before charging money or collecting credentials through anything
  agent-built, rather than shipping payment/auth flows unaudited; personal
  policy of keeping self-built apps internal/client-only rather than selling
  them publicly on the open internet, citing a real security-incident/rebrand
  story as the cautionary tale. Directly relevant before we ship anything of
  our own to real users.
  *Source: Claude Code Full Course 4 Hours (131:19–133:20)*

- **Delegate the deploy/GitHub steps back to the agent instead of doing them
  by hand.** When a plan says "create a repo, push the code, set up the
  webhook," hand that instruction to the agent rather than executing it
  yourself — it usually can, and if it can't, it tells you exactly which
  part failed. Concrete hybrid pattern for the parts it genuinely can't do
  (e.g. clicking through a third-party UI to enter payment webhook secrets):
  agent narrates each step, human executes just that click, back to the
  agent for the rest.
  *Source: Claude Code Full Course 4 Hours (125:21–127:07)*

- **Embed mandatory human-approval checkpoints directly in a skill's
  instructions for consequential actions**, rather than relying solely on
  the platform's permission-mode system. Concrete example: a
  "shop online and purchase" skill with an explicit instruction to "get
  purchase approval, do not skip this step" baked into the skill body
  itself — a belt-and-suspenders safeguard for anything with real financial
  or irreversible consequences.
  *Source: Claude Code Full Course 4 Hours (86:31–89:19)*

- **Outcome-based guarantees as a selling mechanic.** One course's own sales
  structure: a money-back guarantee tied to a concrete client outcome
  (landing the buyer's first paying customer within a set window), not just
  "buy the course." Worth remembering as a productization pattern for
  packaging *our* work — buyers trust outcome-tied guarantees more than
  feature lists.
  *Source: Claude Code Full Course 4 Hours (249:52–250:29)*

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

- **A skill can be the official successor to ad hoc slash commands / a
  homegrown orchestration framework.** One practitioner explicitly retired
  his own custom instruction-framework once skills shipped natively — not a
  technique to copy, just a signal that skills are the durable, standardized
  primitive to invest in rather than inventing a parallel system.
  *Source: Claude Code Full Course 4 Hours (85:37–86:10)*

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

- **Permission mode is a deliberate productivity/safety dial, not a
  one-size-fits-all default.** One practitioner defaults to fully
  auto-accepting edits for "knowledge work and intellectually valuable
  tasks," arguing that constant approve-prompts kill flow once an agent is
  deep in a task — but pairs this with real caution: a cited real incident
  of an auto-accepting agent running a destructive command from a
  misinterpreted request, and a `/permissions` mechanism for setting
  granular per-tool allow/ask/deny rules instead of one global mode. The
  more common everyday cost of loose permissions isn't destruction, it's
  workspace/context bloat (unused files, abandoned approaches accumulating)
  — worth a periodic "audit and clean up unused files" ask as a mitigation
  habit, not just guarding against worst-case damage.
  *Source: Claude Code Full Course 4 Hours (17:53–18:44, 96:03–99:59)*

- **Terminal over GUI as the default surface for serious/parallel work.**
  Explicit recommendation to get comfortable in the terminal specifically
  because it unlocks things a GUI doesn't as easily — running multiple
  sessions side by side, and terminal-only features (e.g. a live token-usage
  status line). Relevant input for our own "collaboration surfaces" thinking
  — terminal-first isn't just a preference, it's a capability floor.
  *Source: Claude Code Full Course 4 Hours (12:02–12:16, 148:20–149:40)*

- **Cap concurrent parallel agent sessions with a concrete rule of thumb.**
  One practitioner caps himself at 3-4 concurrent instances, using this
  heuristic: if an agent sits idle waiting on you more than ~10-20% of the
  time, you have too many tabs open. A more concrete version of the
  "single-focus discipline" entry above — gives an actual threshold rather
  than just "stay focused."
  *Source: Claude Code Full Course 4 Hours (43:05–50:12)*

- **Only install agent-tooling extensions/MCP servers from verified
  publishers.** Named risk: malicious lookalike extensions exist in the
  wild. A basic hygiene habit worth stating explicitly, especially since
  we're actively installing third-party skills/MCP servers ourselves in
  this environment.
  *Source: Claude Code Full Course 4 Hours (16:09–16:19)*

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

5. **"CLAUDE CODE FULL COURSE 4 HOURS: Build & Sell (2026)"** (YouTube,
   ~4h11m, by Nick Saraev, https://www.youtube.com/watch?v=QoQBzR1NIqI) —
   the densest source in this library so far, and the most directly
   on-topic (an entire course specifically on Claude Code's advanced
   surface, from a practitioner who says he runs a $4M/year business on it
   daily). Went deep on territory we had thin or no coverage of: the
   `.claude` directory's actual configuration surface (rules/ folder,
   settings.local.json, memory.md as distinct from CLAUDE.md), concrete
   token-cost numbers (fixed system-tool tax, skills-vs-MCP cost gap, the
   10%-of-context MCP tool-search threshold), Agent Teams as a genuinely
   new orchestration primitive distinct from sub-agents, a concrete
   adversarial-debate verification template, git worktrees' conflict-
   avoidance rationale, and real security/legal hygiene for shipping
   agent-built products. Also produced one real tension worth tracking: this
   source both reinforces "keep memory files lean/curated" *and* separately
   treats a narrow project CLAUDE.md as disposable/low-effort — resolved as
   a scope distinction (see the nuance note under Memory & self-improvement)
   rather than a contradiction. Single practitioner's opinionated take with
   real client-work examples, not benchmarked — treat specific numbers
   (costs, percentages, thresholds) as illustrative anecdotes, not verified
   facts. Processed via 6 parallel agents after the `/watch` skill's normal
   pipeline failed on this video (YouTube's video-CDN 403 recurred; captions
   were recovered and parsed manually).
