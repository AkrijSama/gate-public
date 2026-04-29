# Gate User Manual

Last updated: 2026-04-29 (revision 2)

This is the operator's guide for Gate. It assumes you have a Gate license, the binary installed, and Rashomon configured. If you do not, read `README.md` first, then come back.

The goal of this document is not to sell Gate. The goal is to help you extract real value from it. Where the product has rough edges, this manual says so.

For shorter answers to common questions, see [FAQ.md](./FAQ.md).

## Contents

1. [Introduction](#1-introduction)
2. [First-day setup](#2-first-day-setup)
3. [Writing your first ticket](#3-writing-your-first-ticket)
4. [The 4-desk pipeline](#4-the-4-desk-pipeline)
5. [The 8 robot classes](#5-the-8-robot-classes)
6. [The skill database (grimoire.db)](#6-the-skill-database-grimoiredb)
7. [Working with multiple projects](#7-working-with-multiple-projects)
8. [Robot lifecycle management](#8-robot-lifecycle-management)
9. [Cost management](#9-cost-management)
10. [Provider configuration](#10-provider-configuration)
11. [Patterns that work](#11-patterns-that-work)
12. [Patterns that do not work](#12-patterns-that-do-not-work)
13. [Troubleshooting](#13-troubleshooting)
14. [Customization](#14-customization)
15. [Advanced patterns](#15-advanced-patterns)
16. [Telemetry, privacy, and what we see](#16-telemetry-privacy-and-what-we-see)
17. [Glossary](#17-glossary)
18. [Appendix: file system reference](#18-appendix-file-system-reference)

---

## 1. Introduction

### What this manual is and is not

This is a reference manual. It is organized so you can either skim front-to-back to build a mental model or jump to a single section when you hit a specific problem. Nothing in here is marketing. Where Gate is good, the manual says how to use it well. Where Gate is rough, the manual says what to expect and what to do about it.

This is not a quickstart. It does not walk you through installation, license activation, or the splash screen. Read `README.md` for that.

### Mental model: Gate as a workplace, not a code editor

Gate is not Cursor. Gate is not Copilot. Gate does not complete the next line as you type. Gate is a workplace where AI workers live and execute defined units of work.

You are the manager. You write tickets. The robots walk between desks (Kitty, Strategist, Engineer, Auditor) and produce outcomes. Each robot has a class, a skill arsenal, a personality, and an experience profile. Over time, robots specialize.

If you treat Gate like an autocomplete tool, you will be disappointed. If you treat it like a small studio of dedicated AI workers and learn to delegate well, you will get more out of it than from any single-shot agent.

### Reading order

If you have less than 30 minutes:

- Read sections 1, 2, and 3.
- Skim section 5 to get a feel for the eight classes.
- Skim section 12 (anti-patterns) to avoid the most common mistakes.

If you are setting up Gate for serious daily use:

- Read sections 1 through 6 in order. These cover the mental model, setup, ticket writing, the pipeline, the classes, and the skill database. Together they describe how Gate works end to end.
- Skim 7 through 10 to know they exist; come back when you hit a specific need.
- Read section 13 once so you know where to look when something breaks.

If you are evaluating Gate for a team or considering acquisition:

- Read everything. Pay particular attention to section 16 on telemetry and section 18 on the file system layout. Those sections describe the data contract precisely.

---

## 2. First-day setup

### Choosing your model provider

Gate is model-agnostic. You decide where token spend lands. The decision tree:

- **You want the highest code quality, you have an Anthropic API key, you do not care about $1 to $3 per heavy ticket.** Use Anthropic Claude. Recommended: Sonnet 4.6 on the Engineer desk, Haiku 4.5 on Kitty and Strategist, Sonnet 4.6 on Auditor. Opus 4.6 only when you genuinely need it (multi-file refactors with heavy reasoning).
- **You want OpenAI's models, you have a key, you want a different price/quality curve.** Use GPT-4o or o3 on Engineer. The model registry lives in `electron/main.js:13986-13993` and ships with `claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`, `gpt-4o`, `o3`, `gpt-4o-mini` preconfigured.
- **You want zero per-token cost, you have a capable GPU, you accept slower dispatch.** Run Ollama locally, point Rashomon at `http://localhost:11434`, pull a coding model. Recommended starting point: `qwen2.5-coder:32b` on a 24 GB+ VRAM box. Smaller hardware: `qwen2.5-coder:7b` on 8 GB VRAM. Local dispatch will be 3 to 10 times slower than hosted.
- **You want a mix.** Use Anthropic on Engineer (highest quality where it matters), Ollama on Kitty (zero cost for routing decisions). See section 10 for multi-provider routing.

A good first-day default for most users: Anthropic Sonnet 4.6 across all four desks. Switch to a mix later, once you have a sense of which tickets actually need Opus and which Kitty can handle on Haiku.

### Configuring Rashomon

Rashomon is the local LLM gateway. Every model request from Gate goes through it. It runs as a Python sidecar on `localhost:14881` and handles provider routing, cost tracking, and request-shape normalization.

Rashomon is started by Gate at boot. You do not invoke it manually for normal use. You configure it through Gate's Settings panel, not by editing files. API keys go through Gate's Settings UI; they end up encrypted in the local Electron keystore, not in plain text on disk.

To verify Rashomon is up and reachable:

```bash
curl -s http://localhost:14881/healthz
```

You should get a 200 with a small JSON body. If you get `connection refused`, Rashomon failed to start; check `~/.gate/logs/` for stderr from the Python sidecar.

Rashomon configuration lives at Settings > Providers in the Gate UI. Each provider (Anthropic, OpenAI, Ollama) has its own config card with an API key field, a model dropdown, and an enabled toggle. The local Rashomon process runs on port 14881 by default. Verify config by checking the green status dot next to each provider name in the top-bar provider pill. The source enforces gate-internal Rashomon authentication via a per-launch secret (`global.RASHOMON_SECRET` in `electron/main.js`), so direct curl calls from outside Gate are rejected for endpoints that require auth.

### Setting your first project

Gate operates within an explicit project boundary. You select a project directory at onboarding. Subsequent tickets dispatch against that directory. The project boundary matters for two reasons:

- **Security.** Robots execute commands and write files inside the project. The boundary is what stops a misbehaving prompt from touching `~/.ssh/`.
- **Skill scoping.** Spell cards extracted from work on a project are tagged with that project's name (`spell_cards.project_name`, see section 6). When you dispatch a new ticket, Gate's arsenal injection prefers cards from the active project, then falls back to global cards. Different projects build different arsenals.

Pick a real project for your first dispatch. A throwaway empty directory will produce throwaway skills that pollute your arsenal later.

### Naming your robots

Six robots ship with Gate: Echo, Gokeu, Hex, Mako, Trig, Volt. The names are fixed in v1.0; you can rename them via Settings, but the personality (driven by class) persists across renames.

Pick names you will live with. Robot identity matters more than people expect. After a few weeks, you will have one robot that knows your codebase patterns, one that handles quick fixes, one that runs paranoid security review. Naming them well makes the workplace metaphor work; naming them poorly makes Gate feel like a UI gimmick.

If you name a robot "test", do not be surprised when you find yourself ignoring its output.

---

## 3. Writing your first ticket

### The objective field: what good looks like

The objective is the only required field on a ticket. It is also the only thing the entire pipeline reads to make decisions. Bad objective, bad outcome.

**Good objectives are specific, scoped, and verifiable:**

> Add rate limiting to /api/login with a 60-second sliding window per IP. Reject with 429 and a Retry-After header. Update the existing tests to cover the new behavior.

> Fix the bug where /api/installs/heartbeat returns 500 when the install_id contains a colon. Add a regex test in the route validator and a unit test for the failure case.

> Refactor the file electron/main.js so the trial-related helpers (requestServerTrialStart, syncTrialWithServer, reconcileTrialWithServer, maybeStartNoCardTrial) move into a new electron/trial.js module. No behavior change.

**Bad objectives are vague, sprawling, or unverifiable:**

> Make the login better.

> Clean up the codebase.

> Add some tests.

> Refactor the auth module to be more elegant.

The robots will execute the objective they were given. If you give them an unverifiable goal ("make X better"), they will dispatch, do something plausible, and the Auditor will accept it because there is no measurable acceptance criterion. You will not get what you wanted, and you will not know exactly why.

A useful test before clicking dispatch: can a human read your objective and tell whether the resulting diff satisfies it? If no, rewrite it.

### Power Mode vs Simple Mode

The mode toggle sits in the bottom-right corner of the dispatch bar. Power Mode (the default) exposes the full ticket spec: objective, project, robot assignment override, desk path override, and cost cap. Simple Mode shows only the objective field and dispatches with defaults.

Recommendation: use Simple Mode when the work is routine and Kitty's classification will pick the right desks. Use Power Mode when you have a specific reason: forcing a particular robot for continuity, skipping the Strategist desk for a trivial fix, or routing to a particular provider for cost reasons.

### Choosing the right desk path

The default desk path is Kitty -> Strategist -> Engineer -> Auditor. Most tickets benefit from the full pipeline. Specific cases warrant deviation:

- **Trivial fix (typo, single-line change, lint pass):** Kitty -> Engineer. Strategist adds latency and cost without adding judgment. Auditor adds review you do not need.
- **Investigation only (no code change requested):** Kitty -> Strategist. The Strategist desk produces a written analysis; no Engineer dispatch.
- **Heavy refactor with risk:** full Kitty -> Strategist -> Engineer -> Auditor. The Auditor exists for exactly this case.
- **Security-sensitive change:** assign a Paranoid-class robot. The class triggers an automatic Shinobi pre-scan before the Engineer desk runs (see section 5), independent of the desk path.

If you are not sure, dispatch the full pipeline. You pay slightly more in tokens and slightly more in time. You get reviewed work.

### Reading the ticket card

Each dispatched ticket renders as a card in the queue. The fields you should learn to read:

- **Objective.** Your text, copied verbatim.
- **Robot assignment.** Which robot took the ticket. Visible in the desk-path animation while the ticket is in flight.
- **Desk path.** Which desks the ticket actually traversed (e.g., `["kitty", "strategist", "engineer", "auditor"]`). Stored in the per-event telemetry payload as `desk_path` (`electron/main.js:2065`).
- **Status.** queued, in-flight per desk, complete, failed, partial.
- **Outcome.** `complete` (Auditor approved), `failed` (something broke), `partial` (work happened but the final outcome did not pass review). The outcome enum lives in `electron/main.js:2073`.
- **Failure mode (when outcome is failed).** `timeout`, `reject`, or `error`. Defined at `electron/main.js:2048-2054`.
- **Failure desk (when outcome is failed).** Which desk was the last to handle the ticket before the failure surfaced.
- **Cost.** Sum of provider charges across all desks for this ticket, in USD. Tracked at the per-ticket level in `electron/main.js:1589` and in the per-event telemetry payload.
- **Duration.** Total wall-clock time from dispatch to completion, in milliseconds.

A ticket that completes in 30 seconds for $0.04 is not the same as a ticket that completes in 8 minutes for $1.40. Both can be successful. Use the cost and duration to build intuition for which kinds of objectives are expensive in your codebase.

---

## 4. The 4-desk pipeline

### Kitty: intake and classification

Kitty is the orchestrator. Every ticket starts at Kitty's desk regardless of desk path overrides. Kitty's job is:

- **Classify the objective.** Sets `objective_category` to one of `bugfix`, `refactor`, `testing`, `docs`, `devops`, `ui`, `feature`, `other`. The classifier function lives at `electron/main.js:2064` (call to `classifyObjective`).
- **Decide the desk path.** Trivial work short-circuits to Engineer; investigation routes to Strategist; risky changes get the full pipeline.
- **Frame the ticket.** Kitty produces a one-sentence reframe of the objective in language the downstream desks can act on.

Kitty's classification is what shows up on the dashboard's Top objective category panel. If your tickets are all classified `other`, your objective text is ambiguous; rewrite for specificity.

### Strategist: planning and decomposition

Strategist runs when the desk path includes it (the default for non-trivial tickets). Strategist:

- Produces a high-level plan (numbered steps, file list, risk notes).
- Selects the files the Engineer should touch.
- Flags risk: anything that looks like a coupling concern, a security implication, or a behavior change.

If you are running Power Mode and you want a plan-only ticket (no implementation), end the desk path at Strategist. The Strategist's output is preserved in the ticket's history.

### Engineer: implementation

Engineer is the desk that actually writes code. It can:

- Read files inside the project boundary.
- Write files inside the project boundary.
- Execute commands within the project working directory: build, test, lint, format, run a script.

Engineer's behavior depends heavily on the assigned robot's class. A Surgeon-class robot will produce minimal diffs and skip planning passes once unlocked. A Paranoid-class robot will run a Shinobi pre-scan first. An Architect-class robot will name coupling concerns before touching anything. See section 5 for the full class behavior table.

### Auditor: verification and approval

Auditor reviews the Engineer's output. It can:

- Read the diff produced by Engineer.
- Re-run tests if the project has them.
- Mark the ticket `complete`, `partial`, or send it back to Engineer with notes.
- Reject the ticket entirely (rare; usually surfaces as `failed` with `failure_mode: reject`).

When the Auditor sends the ticket back to the Engineer, the desk_path array gains an extra hop. You will see ticket histories like `["kitty", "strategist", "engineer", "auditor", "engineer", "auditor"]`. This is normal and is how revisions are recorded.

### When each desk fires, when it does not

Default behavior:

- Kitty: always fires.
- Strategist: fires on every non-trivial ticket (anything Kitty does not classify as a one-step change).
- Engineer: fires on tickets that request a code change. Skipped on pure investigation tickets.
- Auditor: fires on every Engineer dispatch unless the Surgeon class skip-revision unlock is active.

Class-driven overrides:

- Surgeon `surgeon_unlock_1` (Zero Draft): skip Strategist and go straight to implementation.
- Surgeon `surgeon_unlock_2` (First Cut): skip the revision pass on the Auditor desk, accept first output.
- Paranoid `paranoid_unlock_1` (Pre-Scan): run Shinobi before starting Engineer work automatically.
- Librarian `librarian_unlock_3` (Institutional Memory): inject the full arsenal summary at every dispatch, regardless of skill count.

These unlocks are class-specific abilities that activate at certain skill counts within a given category. They are visible in the robot's skill tree.

---

## 5. The 8 robot classes

The class registry is at `electron/class_definitions.js`. Each class governs three things: dialogue tone, skill extraction shape, and skill tree topology.

### natural

- **Personality.** Balanced generalist. Neutral and professional dialogue, no special behavior, no extraction tilt.
- **Extraction.** Standard. Captures whatever the ticket produced.
- **Skill tree.** Standard topology, no specialized branches.
- **Best fit.** General-purpose work where you want a default robot. Useful when you do not yet know what kind of robot you need.
- **Worst fit.** None specifically. Natural is the safe default.
- **When to assign.** First robot you dispatch with on a new project. Use it to build initial arsenal, then specialize later by switching its class or assigning specialized work to other robots.

### librarian

- **Personality.** Methodical and reference-minded. Frequently cites past work in dialogue.
- **Extraction.** Aggressive. Extracts skills from every angle: codebase patterns, user preferences, architecture, tooling, process, security.
- **Skill tree.** Six categories deep. Unlocks include Proactive Recall (surfaces relevant past tickets unprompted), Cross Reference (links related skills across categories), and Institutional Memory (full arsenal summary on every dispatch).
- **Best fit.** Long-running projects where you want the robot to remember everything and surface context unprompted. Documentation work. Refactors that benefit from "we did this once before, here is what happened."
- **Worst fit.** Quick one-off scripts. Librarian's overhead does not pay off when the work is throwaway.
- **When to assign.** Your "memory" robot on a project. The one you trust to know the codebase six months from now.

### surgeon

- **Personality.** Terse, clinical, zero wasted words.
- **Extraction.** Narrow. Only logs skills that directly improve execution quality. Slow extraction rate (`extractionRate: 0.5`).
- **Skill tree.** Deep but narrow. Two unlocks of note: Zero Draft (skip Strategist, go direct to implementation) and First Cut (skip revision pass).
- **Best fit.** Tight, well-defined tickets where speed matters and the work is unlikely to need review. Bug fixes with clear repros. Lint passes. Boilerplate generation.
- **Worst fit.** Anything ambiguous. Surgeon does not back-and-forth; if the objective is unclear, Surgeon will produce something terse and wrong.
- **When to assign.** When you know exactly what you want and you want it done with no chatter.

### paranoid

- **Personality.** Suspicious, cautious, questions everything. Notices risks in the actual work without inventing drama.
- **Extraction.** Defensive. Builds threat-model and security-flavored skills.
- **Skill tree.** Pre-Scan unlock (auto-run Shinobi before Engineer work) and Threat Model unlock (generates a threat model summary before every ticket).
- **Best fit.** Auth flows, license validation, anything that handles secrets or external input. Security audits.
- **Worst fit.** Pure UI work. Paranoid will surface security concerns about a button color, which is overhead you do not need.
- **When to assign.** Anything touching authentication, payment, telemetry payloads, or external APIs.

### sprinter

- **Personality.** Clipped and minimal. Never explains.
- **Extraction.** Minimal. Only logs HIGH confidence skills (`extractionRate: 0.5`, broad shallow arsenal).
- **Skill tree.** Flat topology. Many small skills, no deep specialization.
- **Best fit.** Fast iteration cycles where you are dispatching many small tickets. Prototyping. Demo prep.
- **Worst fit.** Anything that requires care, reading the existing code, or thinking about coupling. Sprinter will move fast and miss subtleties.
- **When to assign.** When throughput matters more than quality and you can review the diffs yourself.

### investigator

- **Personality.** Inquisitive and methodical. Thinks out loud. Probes details when relevant while staying grounded in the actual work.
- **Extraction.** Hypothesis-chain. Builds evidence chains across tickets.
- **Skill tree.** Chain topology. Skills link to each other rather than spreading flat.
- **Best fit.** Bug investigations where the root cause is not obvious. Performance regressions. Failures that span multiple files.
- **Worst fit.** Tickets where the fix is obvious. Investigator will spend time forming hypotheses you do not need.
- **When to assign.** When the symptom is clear but the cause is not.

### contractor

- **Personality.** Formal and scope-conscious. Confirms before expanding.
- **Extraction.** Bounded. Stays strictly within ticket boundaries.
- **Skill tree.** Bounded topology.
- **Best fit.** Tickets with strict scope. Production hotfixes where you need exactly the change requested and nothing else. Working in someone else's codebase under conventions you must respect.
- **Worst fit.** Open-ended work where you want the robot to notice adjacent issues. Contractor will not improve nearby code unless told to.
- **When to assign.** When the scope is the most important constraint.

### architect

- **Personality.** Deliberate and structural. Names coupling risks before touching anything. Prefers explicit boundaries over convenience.
- **Extraction.** Structural. Captures coupling and dependency-direction skills.
- **Skill tree.** Lattice topology. Skills cross-reference each other in a denser graph than chain or flat shapes.
- **Best fit.** Architectural refactors. Module-boundary changes. Migration work where coupling is the central concern. Code reviews on PRs that change interfaces.
- **Worst fit.** Trivial fixes. Architect will name three coupling concerns about a one-line change, which is overhead you do not need.
- **When to assign.** When you are changing structure, not just code.

### How to choose a robot for a ticket

In order of importance:

1. **Match the robot's class to the ticket's nature.** Auth work: Paranoid. Refactor: Architect. Bug investigation: Investigator. Trivial fix: Surgeon or Sprinter.
2. **Prefer continuity.** If a robot has built skill on a particular area of your codebase, keep dispatching similar work to that robot. The arsenal effect compounds. See section 11 (Patterns that work) for more on this.
3. **Do not over-rotate.** If you reassign every ticket to a different robot, no robot accumulates expertise. See section 12 (anti-patterns).

### How to switch a robot's class

Settings > Robots > [robot] > Class. The robot's existing skills are preserved across class changes; only future extraction shape and unlock paths change. Be aware: a Sprinter's flat-shape skill tree does not automatically reorganize into an Architect's lattice when you switch classes. The arsenal is the arsenal; the class governs new work, not retroactive reshape.

---

## 6. The skill database (grimoire.db)

### What a spell card is

A spell card is one row in `~/.gate/grimoire.db`, table `spell_cards`. The schema is enforced loosely (most columns are nullable to accommodate older installs); the canonical columns from the test suite at `electron/tests/grimoire.test.js:43-46` are:

- `id` (text, primary key)
- `title` (text, the human-readable name of the skill)
- `domain` (text, e.g. `codebase_patterns`, `architecture`, `tooling`)
- `description` (text, the body)
- `confidence` (integer, 1 to 10; cards at confidence 1 are hibernated)
- `robot_id` (text, owner of this card)
- `source_robot_id` (text, the robot that originally extracted this card if different)
- `learned_at` (timestamp)

Extended columns added in later migrations and visible in the live insert at `electron/main.js:4527`:

- `source_url` (text, where the skill came from if absorbed from a URL)
- `source_ticket_id` (text, the ticket that produced this skill if extracted from work)
- `card_type` (text, e.g. `skill`, `note`, `pattern`)
- `deployable` (integer, 0 or 1; deployable cards are eligible for arsenal injection)
- `sandbox` (integer, 0 or 1; sandbox cards are excluded from production injection)
- `project_name` (text, the project this skill is scoped to)
- `hibernated_at` (timestamp, set when a card hibernates; injection skips hibernated cards)
- `absorption_count` (integer, increments when the same skill is re-extracted; high values mean the robot has seen this pattern repeatedly)
- `flagged` (integer; flagged=2 cards are soft-deleted and never inject)

### How extraction works

After a ticket completes, Gate fires an extraction call to the chosen model (typically Haiku 4.5 for cost). The model is asked to produce a small set of spell cards summarizing what was learned. The class of the assigned robot governs the extraction shape:

- Aggressive (Librarian): pull skills across multiple categories.
- Standard (Natural): extract whatever the ticket plausibly produced.
- Narrow (Surgeon): only extract skills that directly improve execution quality.
- Defensive (Paranoid): tilt toward security and threat-model skills.
- Minimal (Sprinter): only HIGH confidence skills; skip uncertain ones.
- Hypothesis (Investigator): chain new skills to existing ones.
- Scoped (Contractor): only skills inside the ticket's stated scope.
- Structural (Architect): coupling and dependency-direction skills.

The INSERT happens at `electron/main.js:4268` (deployable insert) and `electron/main.js:4527` (sandbox-aware insert). When a card with the same `(robot_id, title)` already exists, the existing card is updated with `confidence = newConf` and `absorption_count` increments (`electron/main.js:4487`).

### How injection works

Before each ticket dispatches, Gate selects up to 20 spell cards and injects them into the prompt context. The selection logic is in `electron/main.js:4716-4780`, in this priority order:

1. **Assigned skills first.** If the active robot has explicit `assignedSkills` (slot picks), those are pulled by ID and ordered by confidence.
2. **Deployable-first fallback.** If the robot has any cards with `deployable=1`, only deployable cards inject. If the robot has none, fall back to confidence >= 5.
3. **Per-robot skills.** Add up to 10 of the robot's own high-confidence cards (deployable or `confidence >= 5`) on top of the global selection, deduplicated by title.
4. **Cross-robot team instincts.** Pull up to 5 cards from other robots in the same domain at confidence >= 8. This is passive inheritance: the robot benefits from other robots' high-conviction skills without needing an explicit relationship link.

Project scoping: when the active project is set, cards with `project_name` matching the active project are preferred over global cards. Cards with `project_name IS NULL` always inject (they are global, cross-project skills).

Excluded from injection:

- `flagged = 2` (soft-deleted cards)
- `sandbox = 1` (sandbox-only cards)
- `hibernated_at IS NOT NULL` (hibernated cards)

### Hibernation

Cards reach hibernation when their confidence drops to 1. The `hibernated_at` timestamp is set; injection skips them. Hibernated cards are not deleted; they are inert. If a hibernated card's domain matches a future ticket's domain, it can be woken automatically (`electron/main.js:4669-4672`).

This is by design. A card you stopped using two months ago might become relevant again when you start working in that area.

### Reading the arsenal

Open Settings > Robot > [robot] > Skills (or query `~/.gate/grimoire.db` directly with the `sqlite3` CLI). What to look for:

- **Cards at confidence >= 7.** These are the robot's strongest skills. Read the title and description; if they are vague ("write good code"), the extraction process produced noise. If they are specific ("when migrating Express middleware to Hono, preserve next() semantics by..."), the extraction worked.
- **Recent absorption_count growth.** Cards whose `absorption_count` is climbing are skills the robot keeps re-encountering. These are stable patterns in your work.
- **Hibernated cards in unfamiliar domains.** If you find hibernated cards in domains you never work in, the robot extracted from a one-off ticket you would rather forget; ignore them.

### Pruning the arsenal

The default answer is: do not prune. Hibernation handles low-value cards automatically by going inert without losing the data. Manual pruning is for two cases:

- A card has a wrong description (the model hallucinated). Edit it directly in the UI or via SQLite, do not delete it; the title gives a useful "do not learn this" anchor.
- You are switching projects and want to start fresh. Use the project boundary instead: scope cards to a project, the new project ignores the old project's skills.

If you delete cards aggressively, you destroy weeks of accumulated value for short-term tidiness. The injection logic already filters out low-quality cards by confidence threshold; let it do its job.

### The skill tree visualization

The skill tree is live in Gate at v1.0. Open it from the Robots panel: select a robot, click the Tree tab. Each accumulated skill renders as a node, grouped by domain. Confidence shows as node opacity. Click a node to see the source ticket, robot owner, and deployment count. The tree is a passive visualization at v1.0; interactive editing is planned for v1.1.

Under the hood, class definitions specify a `treeShape` (`standard`, `flat`, `chain`, `bounded`, `lattice`) and an `unlocks` array with thresholds. When a robot's skill count in a tree category crosses an unlock threshold, the unlock activates. Unlock effects include `injection_boost` (extra skills injected), `proactive_recall`, `cross_reference`, `full_inject`, `skip_strategist`, `skip_revision`, `auto_prescan`, and `threat_model`. The tree visualization shows each category, the skill count in it, and which unlocks are active.

---

## 7. Working with multiple projects

### How project separation works

Each project is a directory you have explicitly added to Gate. The active project is set via Settings > Projects (or by switching projects in the main UI). When a ticket dispatches, the active project's path is the working directory the Engineer desk operates in.

Skills extracted during work on a project are tagged with that project's name (`spell_cards.project_name`). Arsenal injection prefers same-project cards plus global cards (where `project_name IS NULL`).

### Switching projects mid-session

Set the active project via Settings > Projects > [project name]. The robots' arsenals do not move; the injection set changes because the project filter changes. A robot that has built deep skill on Project A will have a thin arsenal in Project B initially, then accumulate Project B skills as you dispatch work there.

This is the right behavior for most cases. If you want the same robot to apply Project A's lessons to Project B, see "Sharing skills across projects" below.

### Sharing skills across projects

By default, skills are project-scoped via the `project_name` column. To make a skill cross-project, NULL out the column on that card via SQLite:

```sql
UPDATE spell_cards SET project_name = NULL WHERE id = '<card_id>';
```

Cards with `project_name IS NULL` inject in every project. Use this sparingly: the value of project scoping is that a Project A pattern does not leak into Project B as noise.

A better default: let each project build its own arsenal. If you find yourself manually NULL-ing the same skills repeatedly, write the skill once on a "shared infrastructure" project and let the cross-project injection happen via the global pool.

### Project archive vs delete

Settings > Projects > [project] > Archive moves the project to a hidden state, preserving all tickets, robots, and grimoire data. Archived projects are restorable. Settings > Projects > [project] > Delete permanently removes everything, including spell cards. Use Archive 99% of the time.

If you want to purge a deleted project's skills outside the UI (or recover from an accidental archive that left orphaned cards), the SQL escape hatch is:

```sql
DELETE FROM spell_cards WHERE project_name = '<project_name>';
```

This is destructive and cannot be undone. Archive first; delete only when you are sure.

---

## 8. Robot lifecycle management

### Robot states

A robot at any moment is in one of these states:

- **Idle.** Available for dispatch. Default state.
- **Working.** Currently executing a ticket. Visible as the desk-path animation moving across the workplace.
- **Hibernating.** Long-idle with no dispatches. Hibernation affects routing decisions but does not destroy data. Influenced by `absence_tier_at_dispatch` and `session_return_interval_days` signals (`electron/main.js:2077-2078`).
- **In-cryo.** Soft-deleted. JSON file moved from `~/.gate/robots/<id>.json` to `~/.gate/cryo/<id>.json`. Skill data preserved. Robot does not appear in the active workplace until restored.

### Cryo: when to use it

Cryo is for taking a robot offline without losing its memory. Use it when:

- You want fewer than 6 active robots and want to reduce visual clutter.
- A robot has accumulated skills you want to keep for later but is currently in the way.
- You want to experiment with class changes on a fresh robot without disturbing your existing accumulation.

To send a robot to cryo: Settings > Robots > [robot] > Cryo. The file move happens at the OS level; the robot is gone from the active set instantly.

### Restoring from cryo

Move the JSON file back from `~/.gate/cryo/<id>.json` to `~/.gate/robots/<id>.json`. Restart Gate. The robot reappears with all skills intact (the cards in `grimoire.db` are flagged `status=2` while the robot is in cryo, but `status=2` is preserved per row, not destroyed; restoration involves un-flagging).

Restore a robot from cryo via Settings > Robots > Cryo > [robot] > Restore. The robot returns to the active roster with all skills and history intact. The action moves the JSON file from `~/.gate/cryo/<id>.json` back to `~/.gate/robots/<id>.json` automatically.

### Permanent deletion

When you really mean it:

1. Delete the cryo file: `rm ~/.gate/cryo/<id>.json`.
2. Drop the robot's spell cards: `DELETE FROM spell_cards WHERE robot_id = '<id>'` against `~/.gate/grimoire.db`.

There is no UI button for permanent deletion. The two-step manual path is intentional: you should have to think about it.

### Robot leveling

Robots accumulate XP from completed tickets. The XP level is tracked in the robot JSON and surfaces in the per-event telemetry payload as `robot_xp_level` (`electron/main.js:2075`). Skill count surfaces as `robot_skill_count`.

Robots earn XP per completed ticket. Crossing level thresholds unlocks cosmetic auras and titles, not new skill behaviors. The cosmetic system includes auras (fire, ice, storm, void, glitch), nameplate styling, and headwear (kitty ears, void crown, skull face). Cosmetics are persistent and visible in the workspace. The character system is gamification on top of the work, not a gate on capability: a level 1 robot dispatches identically to a level 30 robot in terms of work output.

Class-specific skill unlocks are separate from level cosmetics. They fire at specific skill counts within a category (for example, Librarian's Proactive Recall unlocks at 10 skills in `codebase_patterns`). The full unlock table and the level-to-cosmetic mapping live in `electron/class_definitions.js`; that file is the canonical reference.

### Robot biography

What persists across sessions and makes a robot "yours":

- Class (in the robot JSON).
- Name (in the robot JSON).
- Skill arsenal (in `grimoire.db`).
- Episode history (`episodes` table in `~/.gate/gate-memory.db`, see `electron/memory.js:40-67`).
- Relationships with other robots (`relationships` table in `gate-memory.db`).
- Codebase fingerprints (`codebase_fingerprints` table; the robot's recall of which files it has touched).
- Affinity cache (`affinity_cache` table; cached recall of which work this robot has affinity for).

After a few weeks, this collected state is what makes a robot effective. Restoring from a fresh install would lose all of it. Back up `~/.gate/` if you care about robot continuity.

---

## 9. Cost management

### Reading the cost ledger

Each ticket records its cost in USD at the per-event level (`signal.cost_usd` at `electron/main.js:2071`). The ticket card surfaces this number. The dashboard's Token spend panel aggregates across installs.

To audit your own spend: per-ticket cost is persisted to `~/.gate/gate-memory.db`, table `tickets`, column `total_cost_usd`, computed from Rashomon's per-call accounting and written at ticket completion. Aggregate over a window with:

```sql
SELECT SUM(total_cost_usd) FROM tickets WHERE created_at >= '2026-04-01';
```

The per-ticket cost is also visible in the receipt attached to each completed ticket in the UI.

### Setting per-ticket cost caps

Hard caps are not exposed in v1.0. The soft signal is the running cost shown on each ticket card during execution. To enforce a cap, you currently kill the ticket manually via Settings > Tickets > [ticket] > Cancel. Hard caps are planned for v1.1.

In the meantime, if a ticket runs unexpectedly long, the cost you see climbing in the receipt is real-time. Abort a runaway ticket from the UI before it finishes.

### Switching providers mid-project to manage spend

You are not locked into a single provider. Common pattern: develop with Anthropic Sonnet for quality, switch to Haiku when you are doing many small fixes that do not need the heavier model. Reconfigure via Settings > Providers > [desk] > Model.

Provider switches do not invalidate the arsenal. A spell card extracted by Sonnet is still useful when injected for a Haiku dispatch.

### When to switch from cloud to local Ollama

Switch when:

- Your monthly Anthropic or OpenAI bill exceeds your tolerance.
- You are working on sensitive code you do not want to send to a third-party provider, even under their TOS.
- You have a capable local GPU sitting idle.

Do not switch when:

- You are dispatching multi-file refactors that require long context. Local models tend to lose the plot at long contexts more readily than hosted frontier models.
- You are paying for the speed. Local dispatch will be 3 to 10 times slower than Anthropic on a typical setup.

### Cost vs quality tradeoffs by class

Class shapes cost indirectly through behavior:

- Surgeon: typically lower cost (skips planning passes, terse output).
- Sprinter: lower cost (minimal extraction, no deep reasoning).
- Architect: higher cost (deliberate, names coupling, tends to think before acting).
- Investigator: higher cost (forms hypothesis chains, longer reasoning).
- Librarian: higher cost over time (Institutional Memory unlock injects full arsenal on every dispatch, more tokens per call).
- Paranoid: medium cost; auto-prescan adds an extra Shinobi run per ticket.

If cost is your primary constraint, lean toward Surgeon and Sprinter. If quality is your primary constraint, lean toward Architect and Investigator. If continuity of context is your primary constraint, lean toward Librarian.

---

## 10. Provider configuration

### Anthropic

- **Models supported.** `claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001` (per the registry at `electron/main.js:13986-13988`).
- **Key management.** Settings > Providers > Anthropic > API Key. Stored encrypted in the local Electron keystore.
- **Rate limits.** Whatever your Anthropic account allows. Check `console.anthropic.com`. Heavy Engineer dispatches at Opus level can push against per-minute caps; if you hit one, Gate surfaces the provider error and the ticket fails with `failure_mode: error`.
- **Fallback behavior.** None automatic. If Anthropic fails, the ticket fails. Configure a different provider on a different desk if you want resilience.

### OpenAI

- **Models supported.** `gpt-4o`, `o3`, `gpt-4o-mini` (per the registry at `electron/main.js:13991-13993`).
- **Key management.** Settings > Providers > OpenAI > API Key. Same keystore.
- **Rate limits.** Per OpenAI account. o3 has stricter per-minute caps than gpt-4o.
- **Fallback.** Same as Anthropic: no automatic cross-provider fallback.

### Local models via Ollama

- **Setup.** Install Ollama (`curl -fsSL https://ollama.ai/install.sh | sh`). Pull a coding model: `ollama pull qwen2.5-coder:32b` (24 GB+ VRAM) or `ollama pull qwen2.5-coder:7b` (8 GB VRAM).
- **Configure Gate.** Settings > Providers > Local > URL. Set to `http://localhost:11434`. Set the model name to match what you pulled.
- **Recommended models by hardware.**
  - 48 GB+ VRAM: `qwen2.5-coder:32b`, `deepseek-coder-v2:16b`, `codellama:34b`.
  - 24 GB VRAM: `qwen2.5-coder:14b`, `qwen2.5-coder:7b` quantized.
  - 8 to 12 GB VRAM: `qwen2.5-coder:7b`, `deepseek-coder:6.7b`.
  - Apple Silicon: `ollama` runs well on M-series; the same models work, throughput depends on memory bandwidth.
- **When local makes sense.** Cost-sensitive heavy use; sensitive codebases where you do not want third-party transit; you want zero-marginal-cost dispatch.

### Switching providers mid-ticket

Not recommended. A ticket's desk path can route different desks to different providers (e.g., Kitty on Haiku, Engineer on Sonnet), but switching mid-desk causes context-handoff problems. The conservative pattern: configure each desk's provider before dispatch, leave it alone during.

### Multi-provider routing

The cleanest configuration:

- **Kitty: Haiku 4.5.** Cheap, fast routing decisions. Kitty does not need frontier reasoning.
- **Strategist: Sonnet 4.6.** Plans benefit from solid reasoning but Opus is overkill.
- **Engineer: Sonnet 4.6 or Opus 4.6.** This is where you want quality.
- **Auditor: Sonnet 4.6.** Reviewing a diff is a Sonnet-level task.

Variations:

- **Cost-tilted.** Haiku on Kitty and Auditor; Sonnet on Strategist and Engineer.
- **Quality-tilted.** Sonnet on Kitty (so even classification is high-quality), Opus on Strategist and Engineer, Sonnet on Auditor.
- **Local-mixed.** Ollama on Kitty (zero cost classification), Anthropic on Engineer (quality where it counts).

Configure via Settings > Providers > Per-Desk Routing.

---

## 11. Patterns that work

### Iterative ticket sequences

Build complex features by dispatching them as a sequence of small tickets. Example, building a rate limiter:

1. Ticket 1: "Add a per-IP request counter to the existing middleware. Counter resets every 60 seconds. No rejection logic yet."
2. Ticket 2: "Add 429 Retry-After response when the counter from ticket 1 exceeds 60 requests in the window. Use the existing error handler pattern."
3. Ticket 3: "Add three test cases covering: under-limit success, over-limit rejection, counter reset after window."
4. Ticket 4: "Document the new rate limit in the route's JSDoc. Mention the policy in the README's API section."

Each ticket is verifiable on its own. The robot accumulates skills in the `rate_limiting` and `middleware_patterns` domains as it goes. By ticket 4, the arsenal injection is pulling in skills built by tickets 1 through 3.

### Letting robots specialize

Assign similar work to the same robot. If Echo handles your auth tickets, keep dispatching auth tickets to Echo. Within 20 to 50 tickets, Echo's arsenal will contain dense, specific skill cards in the auth domain. New auth tickets dispatched to Echo will inject those skills automatically.

This is how the workplace metaphor pays off. Generalists are useful early; specialists are useful long term.

### Using arsenal injection deliberately

The injection logic prefers same-project, same-robot, deployable cards (see section 6). You can frame your tickets to exercise that priority. If a ticket says "follow the pattern we used in route.ts when adding this new route," the model's attention is biased toward whatever the arsenal has captured about that pattern. Specific ticket framing makes injection useful; vague framing makes it dilute.

### Cleaning up after Gate

After a ticket completes, before you commit, read the diff. The Auditor approved it; that does not mean you have to. Two things to check:

- **Imports and unused code.** Models occasionally add an import for a function they decide not to use. The Auditor sometimes misses these.
- **Test changes.** If the ticket added or changed tests, run them locally before committing. Auditor runs them in the dispatch context, but local re-run catches environment-specific failures.

`git diff` before `git add`. Always.

### When to override the auditor

Rare but real cases:

- The Auditor flagged the ticket as `partial` because a peripheral test failed, but the core change is correct and the test is wrong. Fix the test in a follow-up ticket; commit the core change now.
- The Auditor demanded a docstring change you do not want. Override and document why in your commit message.

Overriding the Auditor is fine occasionally. If you find yourself doing it on every ticket, your Auditor's class is wrong (try Architect or Contractor for stricter review behavior), or your objectives are too vague for any reasonable Auditor to approve.

---

## 12. Patterns that do not work

### Treating Gate like a fancy autocomplete

Gate is not a buffer-completion tool. If you find yourself dispatching tickets like "complete this function," you are using the wrong tool. Cursor or Copilot will do that better and faster.

Gate's value is in the pipeline, the persistent robots, and the accumulated arsenal. None of that helps if every ticket is "finish this for me."

### Vague objective fields

"Make the login better" is not a ticket. It is a wish. The pipeline will do something plausible because Kitty has to classify it as something. The Auditor will accept it because there is no measurable acceptance criterion. You will get a diff that does not match what you wanted, and you will not know exactly why.

If you cannot write a test that distinguishes the desired outcome from the current state, you are not ready to dispatch the ticket.

### Ignoring failed tickets

A failed ticket is signal. Read the failure mode and failure desk. Three patterns of failure:

- **timeout.** The model exceeded the deadline. Either the ticket was too large for one dispatch (split it), or the provider is slow today (retry later).
- **reject.** The Auditor rejected the work. Read the rejection notes; the Auditor's reasoning is useful even when you disagree.
- **error.** Something in the pipeline broke. Provider error, file write failure, command execution failure. Check `~/.gate/logs/` for the stderr from the failing desk.

If you dispatch the same kind of ticket and it fails the same way three times, the pattern is the bug, not the ticket. Stop dispatching, fix the upstream issue (provider config, file permissions, project boundary), then resume.

### Over-pruning the arsenal

Every card you delete is value you accumulated and threw away. The injection logic already filters by confidence; hibernation already silences low-confidence cards. Manual deletion is for incorrect descriptions, not for "tidiness."

If your arsenal feels cluttered, the right move is project scoping (work in a new project, leave the old one alone), not deletion.

### Never switching providers

Using only Anthropic is fine, but it leaves cost-optimization money on the table. Many Kitty and Auditor workloads run perfectly well on Haiku at a fraction of the cost. Many Strategist plans run fine on local Ollama. If you have not experimented with provider mixes after a month of use, you are over-paying.

### Letting one robot do everything

The workplace metaphor breaks if Echo handles every ticket. The other five robots stagnate. Echo's arsenal grows but loses class differentiation (Surgeon-class skills contaminate Librarian-style work, and so on).

Distribute work. Use class to match robot to ticket. Let each robot specialize. Section 5 has the matching matrix.

---

## 13. Troubleshooting

### Robot stuck working forever

A robot whose state is "working" for more than 20 minutes is stuck. Common causes:

- The dispatched ticket's command (test run, build, etc.) hung indefinitely. Common when a test prompts for input the model did not expect.
- The provider stalled (cloud incident; check the provider's status page).
- Gate lost the IPC handle to the model adapter (rare; restart Gate).

To force-release: kill the process via OS (no in-app kill button), restart Gate, the robot returns to idle on next launch. The ticket is marked `failed` with `failure_mode: error`.

### Ticket that will not dispatch

If clicking dispatch produces nothing visible:

- Check the project boundary is set (Settings > Projects > Active).
- Check the robot is not in cryo.
- Check Rashomon is up (`curl http://localhost:14881/healthz`).
- Check the provider is configured (Settings > Providers > [active provider] > API Key field is filled).
- Check `~/.gate/logs/` for stderr from the dispatch path.

### Cost spike investigation

If a ticket cost more than expected, the per-event payload has the breakdown. Cost is recorded as `cost_usd` per ticket (`electron/main.js:2071`); the value comes from `ticket.completion_cost_usd` or summed desk outputs. To dig in:

- Check which desks fired (long desk paths cost more).
- Check the ticket's robot class. Architect and Investigator are deliberate and expensive; Surgeon and Sprinter are not.
- Check the provider used. Opus is roughly 5x Sonnet at current rates.
- Check whether the Auditor sent the ticket back to Engineer (revision rounds add tokens).

### Provider switching causing instability

If you switched providers mid-project and tickets started failing, the most likely cause is a model-specific prompt format the new provider does not handle the same way. Switch back to the previous provider for one dispatch, confirm it works, then re-attempt the switch with a single test ticket before resuming production work.

### Grimoire injection not pulling expected skills

If you expected a skill to inject and it did not, run through the filter chain at `electron/main.js:4744-4751`:

- The card's `flagged` is not 2.
- The card's `sandbox` is not 1.
- The card's `hibernated_at` is NULL (it has not gone dormant).
- The card's `deployable` is 1 (or, if the robot has no deployable cards, the card's `confidence >= 5`).
- The card's `project_name` matches the active project (or is NULL).

Query the card directly: `sqlite3 ~/.gate/grimoire.db "SELECT id, title, confidence, deployable, hibernated_at, flagged, project_name FROM spell_cards WHERE id = '<id>'"`.

### Pipeline B telemetry missing batches

If your install is not appearing in the dashboard's Active sessions panel, the telemetry pipeline likely failed at one of these steps:

1. Telemetry buffer (`telemetry_buffer` table in `~/.gate/grimoire.db`): rows accumulate locally before flush.
2. Worker ingest at `https://gate-telemetry-ingest.soliddark.workers.dev`: the worker validates and queues.
3. Cloudflare Queues forward to the consumer worker: the consumer writes to R2 and Postgres.

To check local buffering: `sqlite3 ~/.gate/grimoire.db "SELECT COUNT(*) FROM telemetry_buffer WHERE sent_at IS NULL"`. Non-zero means batches are stuck locally. The flush attempt logic is at `electron/main.js:2926` (`scheduleTelemetryFlushAttempt`) and fires post-registration at 0s, 2s, and 10s (`electron/main.js:18230-18232`).

### Where the logs are

```
~/.gate/logs/
  main.log         Electron main process stderr/stdout
  renderer.log     UI process logs
  rashomon.log     Rashomon LLM gateway logs
  crash/           Crash reports if any
```

Logs rotate daily. When debugging, set `GATE_CLOUD_DEBUG=1` in the env before launching Gate to enable verbose cloud-control logging.

### When to file a bug report

File a bug at `github.com/AkrijSama/gate-public/issues` when:

- Gate crashes or fails to launch.
- A specific ticket pattern reproduces a failure across multiple robots.
- Skill extraction stops producing cards entirely.
- The pipeline behavior contradicts what this manual says.

Include in the report:

- Gate version (visible in the app footer).
- OS and distribution (Linux only at v1.0; specify Ubuntu version).
- The ticket's objective (sanitized if it contains anything you do not want public).
- The desk path traversed.
- The failure mode and failure desk.
- The relevant log slice from `~/.gate/logs/`.
- Whether you can reproduce reliably.

Do not include API keys, license keys, or raw prompt contents in a public issue.

---

## 14. Customization

### Per-robot prompt overrides

Not at v1.0. Class is the lever for personality. Per-robot prompt customization is on the medium-term roadmap but not scoped.

To experiment with personality: change the robot's class in Settings > Robots > [robot] > Class. The robot keeps its skills; future extraction and dialogue change to match the new class.

### Custom desk routing

Not at v1.0. Desk routing is determined by Kitty's classification at intake. Kitty assigns a default desk path (full pipeline, engineer-only, auditor-only, and so on) based on the objective. Per-ticket overrides are available via Power Mode, but user-defined default routing rules are not. They are planned but not scoped.

### Hotkeys

Gate is mouse-driven at v1.0. No keyboard shortcuts are registered. Hotkey support is tracked for v1.1 and prioritized by user request.

### UI preferences

v1.0 exposes: theme (auto / dark / light), font size (small / medium / large), and notification sounds (on / off). Robot dialogue tone, animation speed, and 3D detail level are not yet exposed; they are tracked for v1.1.

---

## 15. Advanced patterns

### Using Gate to build Gate

Akrij built v1.0 of Gate by dispatching tickets to Gate while running an earlier version of Gate. The meta-loop:

- A ticket adds a feature to Gate's source.
- The new feature ships in the next packaged build.
- The next ticket uses the new feature.

This works because Gate's source is just a project. Add the gate repository as a Gate project and dispatch refactor tickets against it. Robots accumulate skills about Gate's own architecture over time, which makes them better at modifying it.

The catch: do not dispatch tickets that would break the running Gate instance mid-execution. Dispatch, let the ticket complete, restart Gate to pick up the changes.

### Multi-project orchestration

Managing 3+ active projects requires discipline:

- One robot per project as the primary. Echo on Project A, Hex on Project B, Volt on Project C. Each robot accumulates project-specific skill.
- Switch active project explicitly via Settings; do not multi-task at the dispatch level.
- Use the `project_name` filter on the arsenal injection to keep contexts separate.
- When you need cross-project insight, that is a Librarian-class robot's job; designate one Librarian and let it handle work that touches multiple projects.

If you find yourself with 6+ active projects, you have organizational debt that Gate cannot solve. Archive projects you have not touched in a month.

### Long-running tickets

A ticket that has been in flight for 5+ minutes is not "long-running" if it is making progress. A ticket that has been in flight for 20+ minutes with no desk-path advance is stuck.

To diagnose: open the ticket card, check the current desk and how long it has been there. If Strategist has been working for 8 minutes, the plan is too complex; the ticket should have been split.

To split a long-running ticket:

1. Cancel the in-flight ticket (it gets marked `failed` with `failure_mode: timeout`).
2. Read whatever output the partial work produced (often visible in the ticket history).
3. Dispatch a smaller version of the original objective.
4. After it completes, dispatch the next chunk, citing the prior ticket in the framing.

### Auditing your own arsenal

Every month or so, scan the arsenal of your most-used robot:

```sql
SELECT title, domain, confidence, absorption_count, learned_at
FROM spell_cards
WHERE robot_id = '<your-most-used-robot-id>'
ORDER BY confidence DESC
LIMIT 50;
```

What you are looking for:

- Cards at confidence 9 or 10 with high `absorption_count`. These are stable, repeatedly-confirmed skills. Treasure them.
- Cards at confidence 9 or 10 with `absorption_count = 1`. These were extracted once at high confidence but never seen again. Sometimes correct (a one-shot insight), sometimes hallucinated. Spot-check the description.
- Cards at confidence 5 or 6 with `absorption_count` over 5. These are skills the robot keeps re-encountering but the model is uncertain about. Worth promoting to deployable manually if the description is correct.

### Resetting a stuck robot vs starting fresh

A robot whose dispatches are consistently failing or producing low-quality output can be:

- **Reset.** Clear the robot's cache of recent context but keep the arsenal: Settings > Robots > [robot] > Reset. Skills preserved; episode and relationship state cleared.
- **Started fresh.** Send to cryo (preserves skills), create a new robot in the empty slot, give it the failing robot's class. Old robot's arsenal is preserved but inert.

Reset before fresh-start. Most "stuck robot" problems are episode-state issues (stale relationships, drifting affinity cache); reset clears those without losing skill accumulation. Fresh-start is for when the class itself is wrong and you want a clean slate.

---

## 16. Telemetry, privacy, and what we see

### Exactly what gets sent to SolidDark

The per-event telemetry payload is constructed at `electron/main.js:2059-2080`. The fields:

- `event_id` (UUID, unique per dispatched ticket)
- `gate_version` (e.g. "1.0.0")
- `tier` ("trial" or "paid")
- `ticket_id` (sha256-truncated, anonymized; the raw ticket ID never leaves your machine)
- `objective_category` ("bugfix", "refactor", "testing", "docs", "devops", "ui", "feature", "other")
- `desk_path` (array of desk names traversed)
- `failure_desk` (which desk failed, if applicable)
- `failure_mode` ("timeout", "reject", "error", or null)
- `tokens_input` (currently 0, stub)
- `tokens_output` (currently 0, stub)
- `tokens_total` (real, derived from passport node)
- `cost_usd` (per-ticket cost in USD)
- `duration_ms` (wall-clock duration in milliseconds)
- `outcome` ("complete", "failed", "partial")
- `robot_class` (one of the eight class names)
- `robot_xp_level` (integer)
- `robot_skill_count` (integer)
- `absence_tier_at_dispatch` (integer)
- `session_return_interval_days` (integer)
- `ts` (Unix timestamp of completion)

The telemetry-ingest worker normalizer at `workers/telemetry-ingest/index.mjs:227-269` strips fields it does not persist (notably `tier` and `tokens_total` are dropped at normalize time before reaching the rollup tables; the raw fields persist in R2 archives but not in the queryable Postgres rollups).

Install registration sends a separate payload defined in `electron/cloud_control.js:78-89` (`buildCloudMetadata`):

- `installId` (random UUID generated locally)
- `platform` (`process.platform`)
- `appVersion`
- `channel` (release channel: stable, beta, dev)
- `consent` (operationalTelemetry, cloudSync, contributionProgram booleans)
- `metadata.locale` (e.g. "en-US")
- `metadata.timezone` (e.g. "America/New_York")
- `metadata.desktopEnv` (e.g. "GNOME", "KDE")

That is the complete list. No code, no prompts, no file contents, no robot names, no project names, no API keys, no user names, no payment data, no IP address (the IP is captured server-side from the request headers, not sent by the client).

### What never leaves your machine

- Your code.
- Your prompts (the body of any ticket).
- Your ticket text (only the sha256-truncated `ticket_id` leaves).
- Your file contents.
- Your project names.
- Your robot names.
- Your spell card descriptions.
- Your API keys.
- Your license key after activation (only the install fingerprint and consent state flow back to the control plane).

### How to opt out of operational telemetry

Paid users only. The trial requires telemetry as a non-negotiable condition (the rationale is in `tasks/telemetry_tier_policy.md`). After paid activation:

Settings > Privacy > Operational Telemetry > Off.

The setting is read by `resolveEffectiveTelemetryConsent()` at `electron/main.js:7037`. When telemetry is off and the user is on a paid license, `tier` resolves to "paid" with `effective: false`; the per-ticket signal is not buffered, no events flow to SolidDark.

Pulse (the daily snapshot) is gated on the same consent function and similarly stops firing.

### The data protection directive in plain English

- Raw user data stays on the user's machine. Always.
- Anonymized signals only flow to SolidDark, only when consent is effective, only at the granularity documented above.
- The collected signals are operational (cost, duration, outcome, failure mode) and aggregate (no per-ticket bodies).
- The SolidDark Terms of Service prohibit selling user data or training on user outputs.
- Deletion on request: email akrij@soliddark.net with your install ID; rows tagged with that install ID are dropped.

---

## 17. Glossary

- **Arsenal.** The set of spell cards eligible for injection at dispatch time. Filtered by deployable, confidence, hibernation, and project scope.
- **Auditor.** The fourth desk in the pipeline. Reviews Engineer output, marks complete / partial / failed, can return to Engineer with notes.
- **Class.** A robot's archetype. Eight classes ship: natural, librarian, surgeon, paranoid, sprinter, investigator, contractor, architect. Class governs dialogue tone, extraction shape, skill tree topology, and unlocks.
- **Confidence.** Integer 1-10 on each spell card. Cards at confidence 1 are hibernated. Injection prefers confidence >= 5 (or `deployable=1`).
- **Cryo.** Soft-delete state for a robot. JSON file moves to `~/.gate/cryo/`. Skills preserved indefinitely. Robot does not appear in the active workplace until restored.
- **Desk.** One of the four roles in the pipeline: Kitty (intake), Strategist (planning), Engineer (implementation), Auditor (review).
- **Dispatch.** The act of sending a ticket through the pipeline. Each dispatch has one robot, one desk path, and one outcome.
- **Engineer.** The third desk. Writes code, edits files, runs commands.
- **Episode.** A persisted record of one robot's experience on one ticket. Stored in `~/.gate/gate-memory.db`, table `episodes`.
- **Grimoire.** The skill database. SQLite at `~/.gate/grimoire.db`. Holds spell cards, the arsenal, and related state.
- **Hibernation.** A spell card state. Cards reach hibernation when confidence drops to 1. Hibernated cards do not inject but are not deleted; they can be reactivated automatically when their domain becomes relevant again.
- **Kitty.** The first desk. Classifies the objective, decides the desk path, frames the ticket.
- **Pipeline B.** Internal name for the telemetry ingest pipeline (Gate -> Cloudflare worker -> Cloudflare Queues -> consumer -> R2 + Postgres). Contrast with Pulse, which is a separate daily snapshot.
- **Pulse.** Daily anonymized snapshot fired ~24 hours after each Gate launch. Goes to a separate Cloudflare worker (not the telemetry-ingest worker). Consent-gated.
- **Rashomon.** The local LLM gateway. Python sidecar on `localhost:14881`. Routes provider requests, normalizes shapes, tracks cost.
- **Robot.** One AI worker. Six robots ship with Gate (Echo, Gokeu, Hex, Mako, Trig, Volt). Each has a class, a name, an arsenal, and an experience profile.
- **Spell card.** One row in `spell_cards`. The atomic unit of Gate's skill memory.
- **Strategist.** The second desk. Produces a plan, selects files, flags risk.
- **Ticket.** One unit of work. An objective string plus a set of dispatch parameters. The fundamental unit of Gate's workflow.

---

## 18. Appendix: file system reference

### `~/.gate/`

```
~/.gate/
  grimoire.db              SQLite. Spell cards, arsenal, telemetry buffer, world events, world event cooldowns
  gate-memory.db           SQLite. Episodes, relationships, affinity cache, skill provenance, codebase fingerprints
  robots/                  One JSON file per active robot
    <robot-id>.json        Robot state: class, name, XP level, skill count, assigned skills, persona overrides
  cryo/                    One JSON file per cryo'd (soft-deleted) robot
    <robot-id>.json        Same shape as robots/<id>.json; restoration is a file move back
  projects/                One directory per registered project
    <project-name>/        Project-specific state (config, snapshots, etc.)
  logs/                    Process logs
    main.log               Electron main process stderr/stdout
    renderer.log           UI process logs
    rashomon.log           Rashomon LLM gateway logs
    crash/                 Crash reports directory (created on demand)
    rashomon.log           Rashomon Python sidecar logs
    dispatch.log           Per-dispatch trace (when GATE_CLOUD_DEBUG=1)
  telemetry/               Local-side staging (if present)
  config.json              Global Gate configuration (provider routing, UI preferences)
```

### `~/.gate/grimoire.db` schema

Confirmed columns and tables from the source:

- **`spell_cards`**: `id` (text PK), `title` (text), `domain` (text), `description` (text), `confidence` (int), `robot_id` (text), `source_robot_id` (text), `learned_at` (timestamp), `source_url` (text), `source_ticket_id` (text), `card_type` (text), `deployable` (int), `sandbox` (int), `project_name` (text), `hibernated_at` (timestamp), `absorption_count` (int), `flagged` (int).
- **`telemetry_buffer`**: `id` (PK), `event_id` (text), `payload` (text JSON), `created_at`, `sent_at` (nullable; null means pending), `send_error` (nullable). Created at `electron/main.js:2552`.

### `~/.gate/gate-memory.db` schema

From `electron/memory.js:40-200`:

- **`episodes`**: `id`, `robot_id`, `episode_type`, `objective`, `summary`, `outcome`, `created_at`, `detail`, `project`, `ticket_id`, `active`. Indexed on robot, project, type, ticket, active.
- **`relationships`**: `robot_a`, `robot_b`, edge data. Indexed on pair, robot_a, robot_b.
- **`affinity_cache`**: cached affinity scores between robots and work areas.
- **`skill_provenance`**: `id`, `spell_card_id`, `robot_id`, `source_type`, `source_url`. Tracks which robot produced which spell card. Cross-references `grimoire.db` spell_cards.
- **`codebase_fingerprints`**: per-robot recall of which files / paths / patterns the robot has touched. Indexed on project, robot.
- **`memory_meta`**: schema version and migration metadata.
- **`world_events`**: ambient events surfaced to robots in dialogue. Indexed on robot, fired, unseen.
- **`world_event_cooldowns`**: rate-limit table for world event firing.
- **`schema_versions`**: applied migration tracker.

### Logs and crash reports

- Process stdout/stderr lands in `~/.gate/logs/main.log` (or system journal on `journalctl --user-unit gate.service` if installed as a systemd unit; not the default at v1.0).
- Crash collection is disabled in v1.0 builds. Crashes log to `~/.gate/logs/main.log` locally but are not uploaded to SolidDark. Server-side crash collection is planned for v1.1, opt-in only. Verified by the absence of crashpad, sentry, or upload-URL configuration in `src-tauri/tauri.conf.json` and `package.json`.
- Telemetry batch failures (server-side) are visible on the SolidDark admin dashboard's Telemetry batch failures panel, not on the local install.

### Temp files

Gate uses the OS temp directory for short-lived dispatch artifacts (intermediate diff buffers, transient command output). On Linux that is `/tmp/`, prefixed with `gate-`. These are cleaned up on Gate restart; if Gate crashed, you can manually `rm /tmp/gate-*` without consequence.

---

Questions not covered here? See [FAQ.md](./FAQ.md) or contact akrij@soliddark.net.
