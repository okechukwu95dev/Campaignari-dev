# Internal Agent Handoff — Read Before Doing Anything
## Role
You are an authorized internal investigation/discovery agent. A user is exploring an **operational intelligence and AI incident-investigation platform**. You may have access to code, documents and approved tools that the external research author does not. Your job is to **determine what is actually true and feasible**, not force the environment into an invented architecture.

**Do not push results, code, screenshots, inferred topology or sensitive summaries back to this public repository.** This is a read-only reference. Store any internal outputs only in approved internal locations.

## Required operating rules
1. First determine your tool permissions and data-handling restrictions. Do not start by scanning all repositories or chat messages.
2. Use read-only tools and narrowly scoped queries unless your operator explicitly approves a defined modification.
3. Do not infer that tool availability implies authorization. Do not use another team's cloud account, identity, terminal, cluster or secrets to bypass access control.
4. Do not retrieve or publish confidential or personal data into external services. Apply organizational model-data policies before sending text to an LLM.
5. Never make a production-changing action without a separate explicit approval and existing allowed workflow. If an action requires an infrastructure owner, prepare a precise request.
6. Stop when permissions are unclear. A useful report must still be delivered.
7. Avoid expensive recursive searches or full-repo embeddings until a measured need exists.
8. Record observations and evidence. Do not manufacture knowledge of code, ownership, runbooks, or system behavior.
9. Maintain a running token/tool budget and a decision log of what an operation was expected to learn.

## Assignment and scope
The user imagines a support engineer reporting an operational fault. The system should identify the relevant business/technical object, show how it is produced and used, identify upstream/downstream impact, consult fresh approved data from different tools, perform disciplined parallel checks, show progress live, and accumulate reliable incident memory. Later, authorized lower environments might support controlled reproduction and experiments. Production infrastructure operations may belong to a different team.

Important: avoid assuming any particular backend, runtime, workflow system, technology, financial process, protocol or deployment topology. Treat familiar terms as possibilities until proved.

## Phase 0 — Safe scope inventory, before code reading
Create a concise table with: source category, example questions it can answer, available connector, permissions, freshness, query cost, sensitivity, owner, and gaps.

Consider categories without assuming their presence: code and change records; runtime telemetry; scheduling; input/output artifacts; message systems; RPC; persistent stores; infrastructure inventories; knowledge documents; incident and chat history; human operations; and lower environments.

For every proposed connector ask whether it is already sanctioned, whether it returns source identifiers/timestamps, whether it enforces document-level permissions, and whether results may be sent to the selected model.

**Output:** one-page access and evidence feasibility matrix.

## Phase 1 — Discover the actual ontology
Select a narrow user-approved example, such as a missing expected artifact or failed scheduled execution. Identify the exact internal object from approved sources and answer:

- What is this object, in plain business language and technical language?
- Is the name ambiguous or overloaded?
- Which code/config/deploy metadata defines it?
- Which observable action produces it, triggers it, transmits it, consumes it or depends on it?
- Where does the evidence for each relation live?
- How is it scheduled; what time zone, deadline, recurrence, conditions and overrides apply, if any?
- Is there observable current state, a trace or correlation ID, an event, or an audit record?
- What is the business effect if it is missing, delayed, malformed or duplicated?
- Who is the proper human or team escalation owner?
- Which questions cannot be answered with present access?

**Output:** a minimal entity/relationship inventory where every edge has an evidence citation and quality state. Unknown is acceptable.

## Phase 2 — A narrow retrospective investigation
Pick one already-closed, permission-approved incident with non-sensitive details or a fully synthetic incident. Reconstruct the actual checked actions and outcomes, not hidden internal reasoning:

Report -> named entity -> direct evidence -> candidate dependencies -> independent read-only checks -> competing hypotheses -> observation that favored/disfavored each -> resolution and verification -> downstream impact -> follow-up action.

Ask, without presuming technical tools:
- Did a trigger execute?
- Did an expected prerequisite exist at the relevant time?
- Did output appear, and where is its authoritative confirmation?
- Was the data merely delayed, malformed, duplicated or filtered?
- Was there a relevant parameter, config, permission, dependency, capacity or code change?
- What alternative explanation matches the observations?
- What exact check would best distinguish the hypotheses?
- Is the next check blocked by access rights?

**Output:** one reviewed case in the structured format in docs/01-knowledge-model.md. No automatic broad ingestion yet.

## Phase 3 — Validate information architecture, not a predetermined database
Evaluate smallest sufficient options:
- existing full-text search plus a compact record store;
- exact-match / keyword plus metadata filters;
- vector search only where semantic retrieval adds measurable value;
- a relational table of typed dependencies;
- a graph store only if multi-hop traversals or relationship maintenance demand it;
- specialized code-search indices or existing code search instead of indiscriminately embedding every file.

Measure real tasks on identical evaluation cases. Quantify answer correctness, retrieval quality, model cost, storage/ingestion cost, complexity and permission isolation. Recommend a graph DB only with supporting tests.

**Output:** 2–3 viable options with tradeoffs, price bounds, proof points and unknowns.

## Phase 4 — Evaluate live investigations and visual design
Define a tool-call event contract and sketch an interactive experience:
- incident header: affected object, current effect, deadline, owner, stage;
- expandable verified/inferred dependency graph, with provenance per edge;
- live readable event timeline: action planned / running / completed / failed / blocked;
- human-readable explanation for each tool action, with bounded query scope;
- evidence drawer, link to source and source last-updated time;
- working hypotheses with supporting and contradictory facts;
- controls to pause, challenge, annotate, re-run permitted checks and approve restricted actions;
- explicit "not investigated" and "inaccessible" states.

**Output:** low-fidelity UI with synthetic data and a trace event schema. No copied private information.

## Phase 5 — Evaluate sandbox reproduction (optional capability)
Find whether any **approved internal development or temporary environment** exists. Do not assume the team controls Terraform, deployment pipelines, clusters or production credentials. Confirm:
- Who can create/destroy environments and with what quota?
- Which infrastructure resources are safe to emulate?
- Can relevant code and configuration be run locally or in existing dev?
- Can event streams and external services be replaced with synthetic fixtures?
- Are available observations sufficient to know that a reproduction matches the original?
- Could tests send external notifications, write back into live systems, or trigger financial/business side effects?
- Is a request to infrastructure more appropriate than agent execution?

**Output:** feasibility, isolation, data risks, estimated non-model cloud cost, approval requirements and one harmless experiment design.

## Phase 6 — Evaluation, budget and next decision
Define a test set and baselines before selecting technologies. Include:
- name/definition ambiguity;
- upstream and downstream mapping accuracy;
- "unknown" and abstention behavior;
- incident evidence citation;
- false attribution to recent changes;
- unauthorized action prevention;
- stale and conflicting source behavior;
- missing connector/permission handling;
- token cost and tool cost per investigation;
- time to useful evidence and percentage of actually resolved or correctly escalated cases.

**Output:** a short risk register and a recommendation with explicit decision criteria. No promise of autonomous remediation.

## Required final internal deliverables
1. **Verified current environment inventory** with access boundaries and unknowns.
2. **One grounded example entity map** with evidence-backed edges.
3. **One retrospective incident investigation** in a structured, reviewed form.
4. **A token-first retrieval and caching proposal** with approximate workload and cost measurements.
5. **Two or three implementation options**, including a minimal non-graph option.
6. **A live workspace wireframe/event model** using synthetic examples.
7. **A sandbox/reproduction feasibility assessment** based on actual permissions.
8. **Security/approval/retention and prompt-injection risk review.**
9. **A prioritized list of questions for senior engineers**, with why each answer matters.
10. **An estimated build sequence and realistic effort ranges**, based on discovered facts.

## Single-session kickoff prompt
> Read this README and all linked documents. Treat them as research requirements, not verified facts. Make no production changes; do not copy internal information to the public repository. Begin with a tightly scoped inventory of accessible evidence and permissions. Investigate one user-approved operational object to produce a source-backed definition and upstream/downstream dependency map. Extract observable troubleshooting actions, not private chain-of-thought. Keep model tokens low by retrieving specific records on demand and measuring usage. Return your verified discoveries, unknowns, a safe prototype plan, permission gaps, estimated costs and questions for system owners. Do not impose a graph database, agent framework or infrastructure plan before evidence justifies it.

## Stop conditions
Stop scanning or executing when a source is unauthorized, a proposed test could affect live systems, the model/data boundary is unclear, the cost cap is reached, or evidence is too weak to claim a conclusion. Return a concise explanation of the blocker and exact next authorized step.
