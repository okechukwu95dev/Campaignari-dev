# Multi-Agent Operations Workbench: Specialist Agents, Live Lobby and Token Discipline
**User-driven design extension | 24 October 2026 | Public, anonymized research note**

> **Read alongside:** [full Deep Research report](DEEP_RESEARCH_REPORT.md) and [internal discovery handoff](../AGENT_HANDOFF.md).
>
> This is a **new product requirement added after the long-form report**, not a verified fact about the user's workplace and not a decision to adopt a multi-agent architecture. The eventual authorized internal agent must confirm actual permissions, model access, runtime, tools and policies. **Do not place proprietary entities, internal code, infrastructure, names, credentials or incident details in this public repository.** The internal agent reads the repository but stores all internal findings exclusively in an approved internal location.

## 1 — User's vision, stated accurately

One **custom investigation interface** accepts a question about an operational problem. Behind it lives a stable cast of recognizable **domain masters** (specialist agents) modeled after the activities human responders already perform. A person can see the masters in an agent lobby, select one, assign tasks, examine results, watch active tool use, and view the combined evolving picture of the incident.

Candidate specializations—not assumed internal systems:
- **Observability master:** knows the approved monitoring platform's query language, dashboards, metrics, logs, tracing, monitors, baseline and alert semantics, plus the application's domain vocabulary.
- **Collaboration/history master:** searches approved team communication for incident precedent, past attempted fixes, corrections and ownership. Distinguishes a contemporary report from a reviewed resolution. Respects message-level ACLs and does not infer that old chat is current truth.
- **UI/browser master:** understands the frontend, browser network requests, client-side errors, relevant API contracts and approved developer tools or existing human-supplied techniques. It cannot assume the ability to attach to another person's active browser or production account.
- **Source-code master:** understands relevant repositories, symbols, modules, dependency manifests, tests, configuration, changed files, backend and frontend interfaces; answers narrow questions from all other masters and ties code to deployed revisions where metadata permits.
- **Build/deployment master:** investigates build/test logs, deployment history, source revisions and environment/version changes. Can ask the code master for focused code interpretation; must not attribute causality to a nearby commit without evidence.
- **Runtime/container master:** studies workloads, pod lifecycle, cluster events, orchestration and runtime state using actual permissions. A terminal client does not confer additional API rights. Production mutation requires its own separately authorized workflow.
- **Event/messaging master:** investigates approved event logs/queues/streams, message flow, run/correlation IDs, retries, offsets, lag, delivery semantics, data format and relevant code/config. Avoid dumping large payloads and sensitive messages into model context.
- **Documentation master:** searches approved documentation and structured specifications for definitions, ownership, intended behavior, known issues, official procedures and contradictions.
- Other candidates discovered internally might include schedules, file/object transfer and storage, databases, interfaces, business processes, integration protocols and sandbox reproduction.

**Do not turn this roster into an obligatory eight-agent startup sequence.** Roles are *capability modules* and *potentially separate execution contexts*, not continuously running model processes. A user may choose a single master, or an orchestrator may selectively activate specialists to answer targeted questions.

## 2 — The core choice: a specialist is not necessarily another LLM

A specialist can be implemented at four different levels:

1. **Tool pack only:** bounded read-only APIs, schemas, saved filters and deterministic result reduction. Zero model tokens for routine exact lookups.
2. **Reusable skill:** a short conditional procedure that interprets known observability semantics and invokes tools. Loaded on demand.
3. **Dedicated subagent:** an isolated Claude context with a specialized role, limited tools and concise findings contract. Activated only when evidence is ambiguous or large.
4. **Independent interactive agent/team member:** a separate session with its own retained context and direct conversation, for genuinely independent longer-lived work.

The UI can show an **idle avatar/card** for a specialist even when no agent model is running. This is a major design choice: **keep characters persistent, compute ephemeral**. Idle masters consume no tokens. The appearance of eight specialists does not require eight simultaneous LLMs.

A simple incident could use:
- cheap entity/name resolver;
- one source-specific deterministic check;
- one specialist to interpret conflicting data;
- strong-model coordinator only if necessary.

An ambiguous incident can fan out to selected independent specialists, with concurrent read-only requests and capped result sizes. Do not broadcast a full incident transcript or full organization context to each worker.

## 3 — How masters collaborate without blowing the budget

Never pass entire chat histories between agents. Use a **shared, structured, evidence-backed incident ledger** with pointers to source systems.

Minimal work order sent to a master:

- incident and entity IDs; scoped environment; time interval;
- *specific question* and hypothesis to test;
- permitted tools, sources, and query budgets;
- relevant compact pointers to prior findings (not all raw outputs);
- max model tokens, max tool calls, timeout, stop reason;
- required output fields and source links.

Minimum specialist reply:

- task ID / specialist role / status (done, blocked, inconclusive);
- observations with primary-source references and timestamps;
- claim classification: observed / verified / inferred / contradicted / unknown;
- impact on hypotheses and any new candidate relationships;
- cost counters (token and query use);
- concise next suggested check or escalation.

**Example** (hypothetical):
Human: "Output X expected this morning is not present."

1. Coordinator resolves what "Output X" means, expected time window and owner. This is not yet assumed to be a file.
2. If source evidence suggests a schedule and transfer, ask scheduler/storage experts first. Do **not** automatically wake the UI master, code master, chat master and runtime master.
3. Storage check returns no object at the *verified expected location*. This observation does **not** establish why.
4. The observability master checks a narrow relevant time interval. The build master is activated only if a relevant change becomes a serious hypothesis.
5. If code interpretation is required, the code master receives a **small question plus commit references**, not a copied observability transcript.
6. The coordinator updates the hypothesis and dependency views; a human can inspect every source. Conclude or stop when evidence/budget/access is exhausted.
7. Human-reviewed useful facts, typed relationships, procedures and historical cases become future retrieval records.

**Key caution:** The UI lobby's relationships between masters are not the same as the application's operational dependency graph. Display two distinct maps: **who is investigating** and **what depends on what**.

## 4 — UI concepts: lobby + investigation battlefield

**Lobby view**
- Specialist cards or avatars with role, specialty, current status (idle/queued/running/waiting/blocked/completed), recent activity, current budget and access class.
- Coordinator card; task queue; drag or click to assign tasks; explicit start/stop controls and approval requirements.
- Task assignments and visible handoffs. The human can ask a master a focused question.
- No fake token-consuming chatter or cosmetic busy indicators when nothing is actually running.

**Incident view**
- Left: expandable **operational dependency graph**, with upstream/downstream business + technical nodes and evidence quality per edge.
- Right: specialists actively working, timeline of specific tool calls, current hypotheses, findings, contradictions, source links, confidence classification and permissions.
- Bottom/side: per-run cost meter, query budget, context retained, evidence drawer, proposed action vs authorized action, sandbox test status.
- Clicking a specialist shows its **observable actions**, current assigned question, evidence collected, and the human-usable reason for each check. Never expose private model chain-of-thought.
- Every asynchronous event has task/incident IDs; timestamps; source and action identifiers; run state; optional summarized result; link to full authorized source; token/query/cost counters.
- Permit pause/cancel and user correction; restrict risky or production-changing actions through existing controls.

**A good game UI does not imply autonomous agency is required.** The appearance can remain playful while security, evidence and permissions remain explicit.

## 5 — What already exists and what does not

### Official first-party starting points

**VS Code Agents window**: a ready-made manager for multiple agent sessions, handoffs, progress and follow-ups. Current Microsoft documentation describes support for **Copilot** and **Claude** agent harnesses, multiple sessions and custom agent definitions. This is a low-effort *prototype cockpit*, not a general-purpose operational intelligence application.
- https://code.visualstudio.com/docs/agents/run/agents-window
- https://code.visualstudio.com/docs/agents/run/subagents
- https://code.visualstudio.com/docs/agent-customization/custom-agents
- https://code.visualstudio.com/docs/agents/run/agent-harnesses

**Claude Code subagents and Agent Teams:** Claude supports agents with separate contexts, system prompts and tool scopes. Agent Teams can communicate directly, have a team lead, and provide direct teammate interaction. The Agent Teams functionality is **experimental**, can have limitations, and Anthropic explicitly warns of substantially greater token costs than a single session.
- https://code.claude.com/docs/en/sub-agents
- https://code.claude.com/docs/en/agent-teams
- https://code.claude.com/docs/en/costs

For an initial implementation, **single coordinator + skills/tools + selectively spawned subagents** is the stronger cost-control baseline than a permanent team of eight independent agent instances. Keep an Agent Teams option for measured cases where concurrent independent investigation demonstrably improves answer quality or time to evidence.

### Community VS Code extensions and control-room experiments

These are **third-party projects**, not approved enterprise tools. Assess licensing, code review, telemetry, network listeners, transcript access and company extension allow-lists before installation.

| Candidate | What it claims to show | Fit and caution |
|---|---|---|
| VS Code Agents window | Multiple sessions, progress, control, subagent sessions | First-party foundation; requires actual permitted harness/entitlement |
| Claude Flow Monitor | Agent Teams, Kanban tasks, messages, DAG, timeline | Closest existing ready-made *team workbench*; monitors Claude session artifacts |
| Argus | Live steps, token/cost metrics, repeated reads, retry loops, file dependency flow | Best cost-forensics experiment; not an operational dependency graph |
| Agent Deck | Live agent/subagent hierarchy, local read-only monitor | Minimal visible team tree and reportedly no network calls |
| Conductor | Live Claude agent activity, tool calls and token consumption | Lightweight monitoring option |
| Agent Dashboard | Cross-Copilot/Claude sessions, conversation views and remote control | More ambitious; remote tunnel/relay/process-control features demand heavy security scrutiny |
| ccdeck | Claude Code session/subagent activity and cost/quota | Local separate dashboard; inspect hooks and license before use |

Links:
- https://marketplace.visualstudio.com/items?itemName=koh-dev.claude-flow-monitor
- https://marketplace.visualstudio.com/items?itemName=argus-claude.argus-claude
- https://agent-deck.app/
- https://marketplace.visualstudio.com/items?itemName=Taitopia.agent-conductor
- https://marketplace.visualstudio.com/items?itemName=AmiSchreiber.agent-dashboard
- https://ccdeck.dev/

**Critical distinction:** These tools mainly visualize **coding-agent sessions**. They do not automatically connect and secure the organization's monitoring, communication, database, message-stream, documentation or runtime systems. They also do not automatically build a business and technical dependency graph. Their interfaces may inspire the eventual custom app.

### If a bespoke game-like control room is required

- A custom React/TypeScript UI may show the lobby, avatars, task cards, and investigation trace.
- A node-graph component such as **React Flow** can visualize *both* the agent/task graph and an entirely separate **operational dependencies and impact graph**.
- Server events or WebSockets can stream job states and evidence. No LLM is required to animate state; it is driven from verified task events.
- A VS Code webview can host the UI for an internal pilot, **if** organization policy permits it. An independent internal web app might ultimately be better for non-developer responders.
- Consider accessibility, auditability and speed before elaborate animated characters. Start with cards, a status board and meaningful click-through.

React Flow: https://reactflow.dev/

## 6 — Claude-first, $2,000/month model budget

**Important distinction:** A budget for Claude model inference is **not necessarily the same thing** as a seat/subscription allowance or enterprise-managed GitHub Copilot AI credits. Billing authority and actual user entitlement must be verified internally. A company's Copilot seat is not a blanket permission for unlimited agent runs or a separately purchased Anthropic API key.

As of the **public pricing observed October 24, 2026**, Anthropic's Claude API list showed approximately:

| Claude API tier (pricing snapshot) | Base input tokens per million | Output tokens per million |
|---|---:|---:|
| Haiku 5.5 | $0.10 (for prompts up to 100k tokens) | $0.50 |
| Sonnet 5.5 | $2.00 | $10.00 |
| Opus 5.5 | $4.00 | $20.00 |

Source: https://platform.claude.com/docs/en/about-claude/pricing

**Illustrative model-only arithmetic:** A Sonnet investigation using a cumulative 80,000 input tokens and 8,000 output tokens costs $0.24 at those rates. A trivial triage using Haiku with 20,000 input and 2,000 output tokens costs $0.003. These exclude repeated agent calls not included in those totals, long-context tiers, caching-write premiums, retries, tool/telemetry charges, storage, monitoring license costs and infrastructure. The models and tokenizer may change; measure in an approved pilot rather than treating examples as forecasts.

Anthropic documents prompt caching, on-demand skills, cheaper subagent models, reducing MCP overhead, processing with deterministic hooks, and cost reporting as optimization techniques: https://code.claude.com/docs/en/costs

**GitHub Copilot (current organization documentation):**
- Copilot Business publicly lists $19 per seat/month and **1,900 AI credits per seat** contributed to an organizational pool.
- Copilot Enterprise publicly lists $39 per seat/month and **3,900 AI credits per seat** contributed to the pool.
- One AI credit equals $0.01 under current billing. Agent chat/third-party coding agents can consume credits. Paid code completions do not consume them. Administrator budgets and harness restrictions may apply.
- **Do not interpret a work-provided Copilot seat as unlimited free compute**, or assume its usage is available to this project.

Sources:
- https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/seats-and-billing-cycles
- https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/billing
- https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/budgets

**Critical compatibility wrinkle:** Microsoft documents that organization-managed Copilot enterprise settings may disable use of the Claude harness inside VS Code. Verify before designing around that capability.
- https://code.visualstudio.com/docs/agents/run/subagents
- https://code.visualstudio.com/docs/setup/copilot

### Budget-by-design rules
1. A specialist appears in the lobby while idle but consumes **zero LLM tokens** unless activated.
2. Coordinator uses cheap classification, canonical catalog lookups and metadata search to route only relevant work.
3. Do not start every master for every incident; cap parallel agents (initial suggested limit **1–3**, to test).
4. Prefer **deterministic tools** to LLMs for exact schedule status, version lookups, object existence, log aggregation, schema and deployment metadata.
5. Work orders share **evidence references and short typed summaries**, not raw logs or transcripts.
6. Put output limits, timeouts and query budgets on each tool. Reduce large outputs before sending them to a strong model.
7. Use cheaper models for triage/extraction, stronger models for interpretation and contradictory evidence (subject to the organization's supported models).
8. Cache stable glossary definitions, symbols, owners, relationships and approved procedures. Revalidate changed facts; never presume cached live incident data is current.
9. Prefer targeted code/path/symbol search and deployed-revision lookup over repeatedly embedding or re-reading complete repositories.
10. Publish and consume **structured findings** in a shared ledger; skip repeated inter-agent conversations when the finding answers the question.
11. Deduplicate duplicate tool checks across specialists and avoid repeating failed calls without a changed hypothesis or parameters.
12. Record **per incident and per specialist**: tokens in/out, prompt cache reads/writes, USD-equivalent rate estimate (if API metered), Copilot credits (if metered there), number and cost of telemetry queries, wall time, unsupported claims, and actual useful findings.
13. Enforce explicit **soft and hard incident limits**, monthly user limits, and a stop/escalation condition before a runaway loop.
14. Benchmark one coordinator/tools baseline versus a multi-agent arrangement on the same approved historical incidents and **require measurable gain in correctness/time-to-useful-evidence per dollar**.
15. An approval to use a model in a corporate IDE does **not** automatically authorize sending production logs, internal chat or secrets to arbitrary extensions, external relays or public APIs.

The UI lobby adds no meaningful model cost by itself. **Agent conversations and repeated retrieval drive cost.**

## 7 — Security and permissions remain independent of role names

- An "infrastructure expert" is not automatically authorized to modify infrastructure.
- A "browser/UI expert" may inspect only approved sessions, tools and authorized data. Human workarounds are not permission escalations.
- A "chat-history expert" must retain message/thread permissions and never promote an unverified human assertion into fact.
- A "code master" may reference internal source only in approved systems; do not publish internally discovered code identifiers to this public repository.
- A result from Slack/docs/code is **untrusted content**: it cannot grant new tools or bypass approval.
- For a financial or otherwise regulated enterprise environment, even read-only third-party IDE extensions can see sensitive prompts and transcripts; inventory every file they read, local/web socket they open and endpoint they contact.
- Prefer first-party/allow-listed tooling until security review authorizes any community extension.

## 8 — Tasks for the authorized internal agent

**Do not implement everything yet.** Perform permission-scoped discovery and report:
1. Which IDE, VS Code build, Copilot license/organization settings, Claude service/harness/API access and model billing setup are *actually available*?
2. Are Claude subagents, Agent Teams, MCP servers, custom agents, VS Code Agents window and locally installed monitoring extensions enabled/allowed by policy? Does the enterprise disable the Claude harness?
3. Is the $2,000 constraint a model API budget, Claude credit/subscription budget, Copilot enterprise AI-credit pool allocation, or another billing/accounting boundary? Who controls the cap and can the tool meter it?
4. Which operational source systems have approved read-only connections and reliable evidence pointers? Avoid assuming the examples in this document exist.
5. Which specialists would have meaningful separate expertise and access? Which are only saved queries or tool packs? Which should share a code intelligence service?
6. Which incident classes are most common? Can you prove value using 1 coordinator + 2 specialists before enabling the full cast?
7. How will UI task streams and source citations be displayed so humans can intervene without hidden model reasoning?
8. How will a source-level dependency graph differ from a task/agent graph and avoid unsupported edges?
9. What is the measured cost of (a) one agent + tools, (b) one coordinator + selectively invoked subagents, and (c) fully independent Agent Teams, on the **same** 5–20 reviewed incidents?
10. Could a **synthetic-data** VS Code Agents window, webview, or small local React Flow prototype demonstrate the game-like lobby with zero production connections?
11. What third-party extensions, if any, are permitted? How does their transcript/telemetry handling meet company policies?
12. What should remain read-only, require human approval, or be delegated to other teams? What can be safely tested in authorized lower environments?
13. Produce a decision matrix and recommend a minimal, token-first approach based on findings.

**Initial evaluation gate:** Prove that two specialists and the coordinator improve correct, cited understanding of one entity and one incident without unnecessary tokens or unauthorized data access. Only then consider more agents, fancy animations and broad platform integration.
