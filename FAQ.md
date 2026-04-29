# Gate FAQ

Last updated: 2026-04-29 (revision 2)

This is the canonical FAQ for Gate. The version-controlled file lives in this public repository so changes are transparent and dated.

If a question is missing, open an issue on this repo or email akrij@soliddark.net.

## Contents

1. [The product](#the-product)
2. [Pricing and trial](#pricing-and-trial)
3. [Privacy, security, and data](#privacy-security-and-data)
4. [Installation and platforms](#installation-and-platforms)
5. [How Gate actually works](#how-gate-actually-works)
6. [Robots and skills](#robots-and-skills)
7. [Model providers](#model-providers)
8. [Comparison to other tools](#comparison-to-other-tools)
9. [The founder and the company](#the-founder-and-the-company)
10. [Roadmap and timeline](#roadmap-and-timeline)
11. [Support and community](#support-and-community)

---

## The product

### What is Gate?

Gate is a desktop application that dispatches AI workers ("robots") through a four-desk pipeline (Kitty, Strategist, Engineer, Auditor) to execute developer work autonomously. You write a ticket describing the work. The robots walk between desks, plan, execute, and review. You read the result. Built on Electron and Tauri, model-agnostic, your API keys, your machine.

### Who is Gate for?

Developers who already use AI coding tools and want a layer above the IDE that handles routing, planning, and review without manual context-switching. Heaviest current users run multi-step refactors, bug investigations, and documentation passes. Not built for non-technical users; you still need to write the ticket and read the diff.

### What problem does Gate solve?

AI coding tools today operate inside an editor. They write code well, but you do the orchestration: open the right file, frame the prompt, evaluate the output, switch tools, repeat. Gate moves the orchestration into the application. One ticket goes in, multiple robots collaborate across distinct desks, you get a finished outcome with cost, duration, and outcome attribution per ticket.

### Is Gate an IDE?

No. Gate runs alongside your IDE. It reads your project files, writes changes back to disk, and lets your IDE pick them up. If you want autocomplete inside a buffer, use Copilot or Cursor. If you want a robot to take a defined task and return when it is done, use Gate.

### Is Gate an autocomplete tool like Copilot?

No. Copilot completes the next few lines as you type. Gate dispatches a robot to complete an entire ticket while you do something else. Different problem, different surface.

### What does a Gate ticket look like?

A short objective string ("Add rate limiting to /api/login with a 60-second window") plus the project root. Kitty (the orchestrator) classifies the objective, picks the desk path, and dispatches. The Engineer desk does the work, the Auditor desk reviews, and you see the final outcome with cost, duration, and the desk path traveled.

### Can I see Gate working before installing?

Not today. There is no hosted demo. Run the 3-day trial on your own machine; no credit card required.

---

## Pricing and trial

### How much does Gate cost?

$50 per month. One tier. Billed via Lemon Squeezy. No annual discount yet.

### What does the trial actually cost?

Zero dollars. The trial is 3 days, hardware-bound (one trial per machine), no credit card collected. Telemetry is mandatory during the trial; that is the only obligation. After 3 days, dispatch is gated until you activate a paid license.

### What model costs am I paying on top of the $50?

Whatever your model provider charges. Gate does not mark up tokens. If you bring an Anthropic key, you pay Anthropic's per-token rate directly. If you run local Ollama, your model cost is zero. The Token spend panel in the admin dashboard tracks per-event cost so you can see exactly what each ticket cost.

### Is there a free tier?

No. The 3-day trial is the only free path.

### Can I cancel anytime?

Yes. One click in the Lemon Squeezy customer portal. No email required, no retention games. Your subscription stops at the end of the current billing period; the license remains active until then.

### Do you offer refunds?

Lemon Squeezy's default policy applies. Email akrij@soliddark.net within 7 days of the most recent charge for a refund, no questions asked.

### What payment methods are accepted?

Whatever Lemon Squeezy accepts on the soliddark store: credit and debit cards globally, with regional methods (PayPal, Apple Pay, Google Pay) where Lemon Squeezy enables them. Cryptocurrency is not supported.

### Are there team or enterprise plans?

Not at v1.0. Single-seat licenses only today. Team plans are planned but not scoped. If you need multi-seat licensing now, email akrij@soliddark.net and we'll work something out.

### Will the price ever increase?

The plan is to move to $79 per month for individual users and $159 per month for teams after 90 days of retention data, per the SolidDark pricing roadmap. Anyone subscribed before that change keeps the $50 rate for the duration of their continuous subscription.

### Do you offer discounts for students or open-source maintainers?

Not at v1.0. Open to it post-launch once base pricing stabilizes. Email akrij@soliddark.net if you're a student or maintain a meaningful open-source project and want to discuss.

---

## Privacy, security, and data

### Is my code sent to the model provider?

Yes, by definition. To act on your code, the robot has to read it, and the model provider has to receive a request that contains the relevant context. This is how every AI coding tool works. SolidDark does not see this traffic; it goes from your machine directly to your chosen provider using your API key.

### Is my code sent to SolidDark servers?

No. Raw code, prompts, file contents, project names, and ticket bodies stay on your machine. SolidDark does not have a copy.

### What data does Gate send back to SolidDark?

Anonymized operational telemetry only: install ID (a random UUID generated locally), platform, app version, and per-ticket signals like cost in USD, duration in milliseconds, outcome (complete, failed, partial), failure mode (timeout, reject, error), failure desk, robot class, and the desk path traversed (e.g., `["kitty", "strategist", "engineer", "auditor"]`). No code, no prompts, no file contents, no API keys, no user names, no payment data. The full per-event payload shape is enumerated in the SolidDark data protection directive and the source code of the telemetry ingest worker.

### Can I disable telemetry entirely?

After you activate a paid license: yes, in Settings, telemetry is opt-out. During the 3-day trial: no, telemetry is mandatory. The trial requirement is disclosed before activation and exists so SolidDark can offset trial compute costs and evaluate product health on real usage.

### Where is my data stored?

Code, project files, robot state, skill cards, and the local skill database (`~/.gate/grimoire.db`) live on your machine. Anonymized telemetry rows are stored in Supabase (Postgres) hosted on AWS us-east-2. Raw event archives are stored in Cloudflare R2 with no automatic expiry.

### Is the data encrypted?

In transit: every connection between Gate and SolidDark is HTTPS over TLS. License keys stored locally in Gate's keystore are encrypted with a per-machine key. At rest in SolidDark's Supabase: Supabase's standard at-rest encryption applies. R2 archives use Cloudflare's standard encryption. Your model API keys never leave your machine; they sit in your local Gate keystore and travel only to your chosen provider.

### What happens to my data if I cancel?

Your local install is unaffected by cancellation; Gate stops dispatching tickets but the data on your disk stays. The anonymized telemetry rows tied to your install ID remain in SolidDark's database for product analysis. To request deletion of those rows, email akrij@soliddark.net with your install ID.

### What happens to my data if SolidDark shuts down?

License verification runs against SolidDark's control plane, so a permanent shutdown would eventually block dispatch. SolidDark commits to publishing a license-bypass build under PolyForm Strict 1.0.0 if the company is wound down so existing installs continue working without phoning home. Local data on your machine is unaffected.

### Do you sell or share user data?

No. The Terms of Service prohibit SolidDark from selling user data and prohibit any third party from scraping or training on outputs. The full clause is at soliddark.net/tos.

### Can I see exactly what telemetry Gate sends?

Yes. The ticket-completion signal is built in `electron/main.js` at the ticket-completion handler. Every field is named explicitly. The same source file is in this public repo's parent organization once you have access; for now, the per-event field list is in the SolidDark data protection directive and is enumerated in the question above.

### Are you GDPR compliant?

Not formally certified. The practice is aligned: data minimization, no PII collected, all telemetry opt-out for paid users, raw user data stays on the user's machine, deletion on request via email. We have not undergone a formal third-party GDPR audit. If your procurement team requires formal certification, Gate is not the right tool yet.

### Are you SOC 2 certified?

No. SOC 2 typically requires a multi-quarter audit and is not on the v1.0 roadmap. SolidDark is a single-founder operation; the cost of a SOC 2 Type II audit exceeds 90 days of revenue at current prices. If your procurement requires SOC 2, Gate is not a fit yet.

---

## Installation and platforms

### What platforms does Gate run on?

Linux x86_64 only at v1.0. Tested on Ubuntu 22.04 and 24.04. Other glibc-compatible distributions are likely to work but unverified.

### When will Mac be available?

May 2026, after the SolidDark Apple Developer enrollment completes and Mac code-signing is set up. No earlier.

### When will Windows be available?

After Mac. No firm date. The codebase already runs on Windows in development; the gap is signed release packaging and Windows-specific install testing.

### What are the system requirements?

Linux x86_64. Tested on Ubuntu 22.04 and 24.04. CPU: any modern x86_64 (Gate is not CPU-bound for cloud providers). RAM: 4 GB minimum for the Electron app, more if you're running local models. Disk: see install size below, plus 50 to 200 MB for grimoire.db growth over months of use. GPU: irrelevant for cloud providers (Anthropic, OpenAI). For local models via Ollama or llama.cpp, your VRAM requirements depend entirely on which model you load. A 7B model fits on 8 GB VRAM; a 70B model needs 48 GB+ or quantization.

### How big is the install?

The v1.0.0 AppImage is approximately 201 MB (201,009,656 bytes exactly), measured from the GitHub release asset. The `.deb` is approximately 139 MB (139,261,626 bytes). After install and skill extraction, expect another few hundred MB in `~/.gate/` over months of use.

### What does the install actually do to my system?

The AppImage is a single self-contained binary. Running it does not touch system directories. State lives under `~/.gate/`. The installer creates no system services, no auto-start entries, no PATH modifications. Removing the AppImage and the `~/.gate/` directory removes Gate completely.

### Where does Gate store its files?

- AppImage binary: wherever you put it, typically `~/Downloads/` or `~/Applications/`
- App state, robot data, skill DB: `~/.gate/grimoire.db` and adjacent files in `~/.gate/`
- Logs and crash reports: `~/.gate/logs/` (when present)
- License key: encrypted in the local Electron keystore

### Can I install Gate without admin/root access?

Yes. The AppImage runs as a regular user. No sudo required at any step.

### How do I uninstall Gate?

Delete the AppImage. Delete `~/.gate/`. That is the full uninstall. To also revoke the hardware fingerprint binding on the trial or paid license, email akrij@soliddark.net.

### How do I update Gate?

The Tauri auto-updater checks the manifest at `https://github.com/AkrijSama/gate-public/releases/latest/download/latest.json` and prompts when a new version is available. You can also redownload the AppImage directly from the latest release page.

### Is the binary signed?

The Linux AppImage is signed with the SolidDark Tauri signing key. The signature file (`Gate_X.Y.Z_amd64.AppImage.sig`) is published alongside each release on the GitHub release page. Verification uses the Tauri public key bundled with the updater.

### How do I verify the SHA256?

Each release page on this repo lists asset SHA256 sums. Download the AppImage, run `sha256sum Gate_1.0.0_amd64.AppImage`, compare against the published value. If they do not match, do not run the binary.

### Is Gate open source?

The source is published under PolyForm Strict 1.0.0 (see `LICENSE` in this repo). PolyForm Strict permits non-commercial reading, study, and modification; it prohibits redistribution and commercial use without a separate license. This is "source-available," not "open source" in the OSI sense.

---

## How Gate actually works

### What is the 4-desk pipeline?

A ticket flows through up to four desks in sequence: Kitty receives the objective and classifies it, Strategist plans the approach, Engineer executes the change, Auditor reviews the result. A robot walks between desks; the desk does not own the work, the robot does. Some tickets short-circuit (Kitty -> Engineer for trivial fixes); some retry (Auditor sends back to Engineer with notes). The desk path traversed for every ticket is logged.

### What does each desk do?

- **Kitty**: ticket intake, objective classification (bugfix, refactor, feature, docs, etc.), routing decision.
- **Strategist**: high-level plan, file selection, risk assessment.
- **Engineer**: actual code changes, file writes, command execution.
- **Auditor**: review of changes, reject or accept, optional return to Engineer.

### How does Gate dispatch work?

You write the ticket, click dispatch. Gate selects an idle robot, builds the dispatch payload (objective, robot context, current desk's prompt template), and sends it through Rashomon to your chosen model provider. The response is parsed, applied to disk, and the robot moves to the next desk. Each step is logged with cost, duration, and outcome.

### What is Rashomon?

A local LLM gateway that runs as a Python sidecar on `localhost:14881`. Every model request from Gate goes through Rashomon. It handles provider routing (Anthropic, OpenAI, Ollama, etc.), security (no plain-text key forwarding), cost tracking (per-call dollar accounting), and anomaly detection. You can swap providers without restarting Gate.

### What is Grimoire?

A skill extraction PKM (personal knowledge management) database stored at `~/.gate/grimoire.db` (SQLite). It absorbs content (YouTube transcripts, GitHub repos, documentation) and extracts discrete reusable skills. The skills are injected into Gate dispatches as "spell cards" so robots execute work informed by skills you have collected. Grimoire ships with Gate and is included in the $50/month price.

### What is a spell card?

One row in the Grimoire database representing a discrete skill (a coding pattern, a debugging heuristic, an API quirk, a test-writing approach). Each card has a title, description, and the pattern itself. Cards are extracted from absorbed content automatically and reviewed by you before being added to the active arsenal.

### What is the arsenal?

The set of spell cards currently active for dispatch. When Gate sends a ticket, the relevant cards from the arsenal are concatenated into the prompt context so the robot has access to your collected skills. You curate the arsenal manually; not every extracted card is automatically included.

### How do robots learn over time?

Each robot accumulates an experience profile (XP level, skill count, absence tier, session return interval). When a robot completes a ticket, its profile updates. Higher XP robots receive priority routing on certain desk paths. The exact learning model is in the source; experience is per-robot, not pooled across all installs.

### Does Gate run any code on my machine?

Yes. The Engineer desk can execute commands inside your project root: build commands, test runs, lint, format, etc. Gate does not run arbitrary code from the model unless that command is explicitly invoked as part of the dispatched task (a ticket like "run the tests after applying the patch" will run the tests). Sandboxing scope is the project working directory.

### Can Gate access files outside its working directory?

Yes, by design. Gate's robots execute developer tickets which often require touching files across a project tree. The robot only acts on the project directory you've explicitly added to Gate. There is no IPC-layer path-escape enforcement today; security relies on the project boundary you set in Gate's UI. Run Gate in a container or VM if you need stricter isolation.

### Can Gate access the internet?

Yes, for two purposes only: model provider API calls (whatever your provider needs) and SolidDark control-plane calls (license verification, telemetry ingest, update check). It does not browse the open web for content during ticket execution. If you want web research as part of a ticket, you bring that in via a model provider that supports it (e.g., Anthropic Claude with web search enabled).

---

## Robots and skills

### How many robots come with Gate?

Six: Echo, Gokeu, Hex, Mako, Trig, Volt. Each is persistent across sessions, has its own experience profile, and can be assigned to any desk.

### Can I add or remove robots?

Six robots ship with Gate (Echo, Gokeu, Hex, Mako, Trig, Volt). Maximum 6 robots active at once. Add and remove via Settings > Robots. Removed robots enter cryo (file move from `~/.gate/robots/<id>.json` to `~/.gate/cryo/<id>.json`); skill data is preserved indefinitely. To restore, move the cryo file back. To permanently destroy, delete the cryo file and the robot's spell card rows in `~/.gate/grimoire.db` table `spell_cards`.

### What are the 8 robot classes?

Eight classes are shipped in v1.0, defined in `electron/class_definitions.js`. Each class governs dialogue tone, skill extraction shape, and skill-tree topology:

- **natural**: balanced generalist, standard extraction, neutral and professional tone, no special behavior.
- **librarian**: knowledge archivist, aggressive extraction across six categories, methodical tone that frequently cites past work, surfaces relevant prior tickets unprompted at higher skill counts.
- **surgeon**: precision executor, narrow deep extraction, terse and clinical tone with zero wasted words, can skip strategist and revision passes at higher skill counts.
- **paranoid**: security sentinel, defensive extraction, suspicious and cautious tone, runs Shinobi pre-scan before Engineer work automatically once unlocked.
- **sprinter**: speed optimizer, minimal extraction (only HIGH confidence skills, broad shallow arsenal), clipped tone that never explains.
- **investigator**: root cause analyst, hypothesis-chain extraction, inquisitive tone that thinks out loud, builds evidence chains across tickets.
- **contractor**: scope enforcer, bounded extraction strictly within ticket boundaries, formal scope-conscious tone, confirms before expanding.
- **architect**: structure-first, structural extraction with a lattice skill tree, deliberate tone that names coupling risks before touching anything, prefers explicit boundaries over convenience.

Class governs robot behavior holistically; tone, extraction shape, and unlock paths all change per class.

### Can I customize a robot's personality?

Not at v1.0. Each robot's class determines its personality, dialogue tone, and skill extraction shape. Per-robot prompt overrides are planned but not scoped. To experiment with personality, change the robot's class in Settings > Robots > [robot] > Class.

### What happens if I delete a robot?

Soft delete by default. The robot's JSON file moves from `~/.gate/robots/<id>.json` to `~/.gate/cryo/<id>.json`. The robot's spell cards in `grimoire.db` are flagged (`status=2`) but never destroyed. To restore, move the cryo file back. To permanently destroy, delete the cryo file and the robot's spell card rows in `~/.gate/grimoire.db` table `spell_cards`.

### What is robot hibernation?

When a robot has been inactive for an extended period, it enters a hibernated state. Hibernated robots can be reactivated; their experience profile is preserved. Hibernation affects routing decisions but does not destroy data.

### What is the absence system?

Each robot tracks days since last dispatch (`absence_tier_at_dispatch`) and the interval since last user session (`session_return_interval_days`). These signals influence how Gate frames robot interactions and contribute to the per-event telemetry payload. The absence system is documentation of how rusty a robot is, not a penalty.

---

## Model providers

### What models can I use with Gate?

Anything Rashomon supports. At v1.0 the supported list is Anthropic Claude (Opus, Sonnet, Haiku), OpenAI (GPT-4o, o3, GPT-4o-mini), and any Ollama-served local model. Provider plugins for additional services (Mistral, Cohere, Together, Groq, etc.) are added on demand to Rashomon.

### Do I need an API key?

Yes, unless you run local Ollama only. The trial does not include hosted model credits; you bring your own keys. This keeps the $50 trial cost-free for SolidDark and gives you transparent per-token billing from your provider.

### Where do I get an API key?

- Anthropic: console.anthropic.com -> API Keys
- OpenAI: platform.openai.com -> API Keys
- Ollama: no key needed; just run Ollama locally and point Gate at `http://localhost:11434`

### Can I use multiple providers at once?

Yes. Different desks can use different providers. A common configuration: Anthropic Claude on Engineer (highest code quality), OpenAI on Strategist or Auditor (cheaper), Ollama on Kitty (zero cost for routing decisions).

### Can I use local models (Ollama, llama.cpp)?

Yes. Ollama is a first-class supported provider. Pull a coding-capable model (`ollama pull qwen2.5-coder:32b` is a reasonable starting point), and point Rashomon at it during onboarding. Local models are slower than hosted providers but cost zero per token.

### Is there a recommended model?

For Engineer desk: Claude Opus 4.7 or Claude Sonnet 4.6. Best code quality per dollar at current pricing. For Auditor: Sonnet 4.6. For Strategist and Kitty: Haiku 4.5 is sufficient and the cheapest of the Anthropic family. Local Ollama with a 30B-class coder model works but is materially slower on long-context tickets.

### How much will running Gate cost in API fees?

Highly variable. A trivial typo-fix ticket might cost less than a cent. A multi-file refactor with full test runs might cost a dollar or more. The Token spend panel in the admin dashboard tracks per-event spend so you can build your own intuition. Expect a working developer's monthly API spend on Gate to be in the same range as their existing Cursor or Copilot heavy usage, often less because Gate batches work into discrete tickets rather than continuous streaming.

### What happens if my API key runs out of credits mid-ticket?

The dispatch fails with a clear provider error (rate limit or insufficient credit). The ticket is marked failed, telemetry records the failure mode, and the desk path is preserved up to the failure point. You add credits and re-dispatch. Gate does not retry silently against an exhausted key.

---

## Comparison to other tools

### How is Gate different from Cursor?

Cursor is an IDE fork with AI editing built into the buffer. You are still the orchestrator; Cursor responds to your typing and your prompts. Gate is a layer above the IDE: you write a ticket, the robots execute, you read the result. Different surface, different workflow. They are not mutually exclusive.

### How is Gate different from Aider?

Aider is a terminal application that pairs you with a single AI coder on a given Git repo. Gate dispatches a multi-robot pipeline (Strategist plans, Engineer codes, Auditor reviews) on a ticket basis with cost, duration, and outcome attribution per ticket. Aider is closer to a CLI partner; Gate is closer to a delegation surface.

### How is Gate different from GitHub Copilot?

Copilot completes the next few lines as you type, primarily inside an editor. Gate dispatches whole tasks. You can use both at the same time without conflict.

### How is Gate different from Claude Code?

Claude Code is a single-agent CLI from Anthropic that operates against a project root, calling tools as the model decides. Gate runs a multi-agent pipeline with explicit desks (Strategist, Engineer, Auditor), persistent robot identities with experience profiles, a skill database (Grimoire) that injects collected knowledge into every dispatch, and per-ticket cost and outcome tracking. Gate is also model-agnostic; Claude Code is Anthropic-only.

### How is Gate different from Devin?

Devin is a hosted autonomous agent operated by Cognition. Gate runs entirely on your machine with your API keys. Your code never leaves your local environment except for the model provider call you initiate. Pricing model differs: Devin charges by ACU; Gate is a flat $50 per month plus your model fees.

### Why would I use Gate instead of just calling Claude directly?

You can call Claude directly. For one-off questions, the API is enough. Gate is the compound surface: structured dispatch, multi-step pipeline, per-ticket cost and outcome attribution, persistent robot state, skill arsenal injection, and a local UI that lets you queue and review tickets without writing the orchestration yourself. If you do not need any of that, the raw API is cheaper.

---

## The founder and the company

### Who built Gate?

Akrij, founder of SolidDark, building solo as the "Digital Architect." Every line of Gate's code was delegated to AI agents (primarily Claude Opus and OpenAI Codex) through structured prompts. Akrij does not write code by hand; he directs.

### Is SolidDark a real company?

Yes, registered as a US legal entity. Gate is one product in a planned suite (Rashomon, Grimoire, Shinobi, Beacon). SolidDark is not venture-funded.

### Are there other people working on Gate?

Not at the time of v1.0. SolidDark is a single-founder operation. AI agents handle code generation; design, product direction, and operations are Akrij.

### Is SolidDark venture-funded?

No. The stated goal is to reach $5K MRR before any investor conversation, per the SolidDark roadmap.

### Why should I trust a solo founder?

Reasonable question. Mitigations:
- Source is published under PolyForm Strict in this repo so the code is inspectable.
- License verification is documented; if SolidDark winds down, the public commitment is to publish a license-bypass build so existing installs keep working.
- Telemetry payloads are anonymized and disclosed; no PII is collected.
- Cancellation is one click in the Lemon Squeezy customer portal.

The trust trade-off is real. You are buying a $50/month tool from a one-person operation. Decide based on whether the product solves a real problem for you, not on company size.

### How can I reach the founder?

Email akrij@soliddark.net. GitHub issues on this repo are also read directly.

---

## Roadmap and timeline

### What's on the roadmap?

Stated short-term: macOS build (May 2026), Windows build (after Mac). Stated medium-term: per-robot prompt customization, team licensing, expanded skill extraction sources beyond ticket completion. No firm dates beyond what's in this list. The repo's recent commits and open issues are the most current source of what's actively being worked on.

### When is the next release?

Releases ship as needed. The current shipped version is v1.0.0 (2026-04-27). Patch releases for the Linux build happen on the SolidDark internal cadence; subscribe to this repo's releases for notifications.

### How do you decide what to build next?

Activation rate, conversion funnel, and direct user feedback drive prioritization. The admin dashboard tracks the leading indicators (download to install, install to first ticket, first ticket to paid conversion). Features that fix friction in those steps come before features that add surface area.

### Can I request a feature?

Yes. Open an issue on this repo with the `enhancement` label or email akrij@soliddark.net. Expect a direct response, not a marketing reply.

### Can I report a bug?

Yes. Open an issue on this repo with the `bug` label. Include OS version, Gate version (visible in the app footer), the ticket objective, and any logs from `~/.gate/logs/`. Telemetry rows are also reviewed when investigating reported failures.

### Where do I see what's currently shipping?

Releases on this repo. Each release has a CHANGELOG entry. Major roadmap moves are announced on the SolidDark X account.

---

## Support and community

### How do I get help if something breaks?

In order of preference:

1. Check this FAQ for the failure mode.
2. Open an issue on this repo with logs from `~/.gate/logs/`.
3. Email akrij@soliddark.net for paid-license issues that need direct response.

### Is there a Discord or community?

Not yet. The public surfaces today are this repo (github.com/AkrijSama/gate-public) and the SolidDark X account (@SolidDarkX). A Discord may launch post-Mac-build if there's user demand.

### How fast do you respond to support requests?

Best-effort, solo founder. Typical response within 24 hours. No formal SLA at v1.0. Email akrij@soliddark.net or open an issue at github.com/AkrijSama/gate-public/issues. Bugs that block paying customers are prioritized.

### What hours is support available?

Best-effort during EST waking hours (Akrij is in Orlando, FL, US). No 24/7 support at v1.0.

### Is there documentation beyond this FAQ?

The CHANGELOG in this repo, the README in this repo, and the source-code comments in the parent organization once you have access. Beyond that: this FAQ is the canonical reference until full docs ship. If you find a question this FAQ does not answer, opening an issue here adds it.

---

For purchase, soliddark.net/gate. For source under PolyForm Strict, this repository. For direct contact, akrij@soliddark.net.
