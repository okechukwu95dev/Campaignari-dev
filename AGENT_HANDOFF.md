# Internal AI Agent — Latest Deep Research Handoff

**Source of this assignment:** [Full Deep Research report (October 24, 2026)](docs/DEEP_RESEARCH_REPORT.md). This handoff is extracted from that report, not based on preliminary notes. For background, alternatives, public vendor research, limitations and cost assumptions, read the full report first.

**Public reference, private investigation:** This repository is public and read-only for your investigation. Never commit, push, open an issue, or attach any internal application names, code snippets, customer information, credentials, logs, runtime topology, incident details or work artifacts here. Keep discoveries exclusively in an explicitly authorized internal workspace and comply with organizational policies on external models and data.

**Start with discovery, not implementation.** Do not access production data, execute commands, provision infrastructure, or automate internal processes merely because this document discusses those capabilities. Confirm the user's scope and your authorized access first; keep discovery read-only. Obtain explicit permission before touching confidential sources or making changes.

**Most important requirement:** Minimize model tokens and expensive tool calls throughout. Measure useful evidence and correct diagnosis per token; avoid broad ingestion, giant model contexts, unnecessary model-to-model conversations, or adopting graph/vector infrastructure without demonstrated value. Every relationship and conclusion must have auditable provenance and a fresh verification state. A human must be able to see the developing dependency graph and investigation events live.

---

# Operational Investigation Platform — Discovery Assignment

## Mission

Investigate this organization's real engineering and production-support
environment and determine how an evidence-grounded AI operational investigation
system could be built here.

Do not assume that any example technology, entity, protocol, architecture,
database, cloud provider, scheduler, queue, storage system, observability
platform, infrastructure tool, or deployment model mentioned in this document
exists internally.

Discover the actual environment from available evidence.

The eventual goal is an internal system capable of helping a human understand
and investigate operational problems across scattered sources while keeping
model/token usage low.

The target is not a chatbot.

The target is an operational investigation engine with:

- entity understanding
- upstream/downstream dependency understanding
- live operational evidence retrieval
- code/change correlation
- hypothesis-driven investigation
- evidence-backed conclusions
- reusable operational memory
- reusable troubleshooting skills
- historical evaluation
- human-visible investigation state
- strict permission boundaries
- optional controlled sandbox reproduction where feasible

Do not make production changes as part of this discovery.

Do not introduce new infrastructure merely because this document mentions it.

Do not commit to a graph database, vector database, agent framework, MCP,
Terraform, Kubernetes, a specific LLM, or any other implementation technology
until the real requirements justify it.

## Core problem

Operational troubleshooting knowledge is distributed.

A human responder may need to move among several systems to understand a single
problem.

Depending on the actual environment, evidence might live in some combination of:

- monitoring/observability
- alerts
- application logs
- infrastructure logs
- traces
- metrics
- scheduler/orchestrator systems
- object/file storage
- file-transfer systems
- queues/topics/event systems
- databases
- source repositories
- deployment history
- configuration
- feature flags
- Kubernetes/container systems
- infrastructure systems
- documentation
- chat systems
- incident history
- tickets
- dashboards
- local developer tooling
- external services
- business-operated processes

These are examples only.

Discover which sources really exist.

## Desired user experience

Consider a generic report such as:

    "The Output X did not arrive."

"Output X" is only an illustrative placeholder.

The eventual system should not merely search for the word.

It should determine, from internal evidence:

    What is Output X?

    Is it a file?
    A report?
    A message?
    A business process?
    A job?
    A stage?
    An internal nickname?
    Something else?

Once identified, the user should eventually be able to inspect:

    definition
    aliases
    owner
    expected behavior
    expected timing

    upstream dependencies
    producer
    triggers
    required inputs

    transport/delivery path where applicable

    downstream consumers
    downstream business or technical impact

    relevant implementation
    repository/code references
    relevant configuration

    deployments and recent changes

    observability references
    schedules
    runbooks

    related historical incidents
    known failure patterns

Every relationship should have provenance.

The system must distinguish:

    verified facts
    runtime observations
    human-declared facts
    inferred relationships
    unknowns
    stale information
    contradicted information

Never silently turn an inference into a fact.

## Investigation model

Study whether the environment can support an investigation loop like:

    1. Identify the incident scope.

    2. Resolve the affected entity or entities.

    3. Establish:
       - environment
       - relevant time range
       - ownership
       - expected behavior

    4. Retrieve a small amount of orientation context.

    5. Generate a small set of competing, testable hypotheses.

    6. For each hypothesis determine:
       - evidence supporting it
       - evidence contradicting it
       - evidence still missing
       - the cheapest/safest next check that could distinguish it
         from competing hypotheses

    7. Run targeted read-only checks using authorized tools.

    8. Reduce large tool outputs before they enter expensive model context.

    9. Record evidence as structured findings.

    10. Update, reject, or deepen hypotheses.

    11. Stop when:
        - evidence is sufficient,
        - further investigation has low expected value,
        - a configured resource/query/token budget is reached,
        - permission prevents further investigation,
        - or human assistance is required.

    12. Produce an evidence-backed conclusion or explicitly report
        that the cause remains inconclusive.

    13. Save only reviewed/reusable knowledge.

Do not design the system around unconstrained free-form model reasoning.

Prefer a visible investigation trace containing:

    hypothesis
    query/check
    tool
    source
    result
    evidence
    conclusion change
    next action

## Evidence model

Evaluate a structured evidence record resembling:

    evidence_id
    incident_id

    claim

    status:
        verified
        supported
        contradicted
        unknown

    source_type
    source_reference
    source_timestamp
    retrieval_timestamp

    environment
    entity_ids

    query/check performed

    supporting_evidence
    contradicting_evidence

    hypothesis_ids

    confidence or evidence quality

Do not treat generated prose as the primary system of record.

The human explanation may be generated from structured records.

## Entity and relationship discovery

Inspect the real environment and identify candidate entity classes.

Do not assume the following list is complete or correct.

Possible examples include:

    application
    service
    business process
    job
    schedule
    artifact/file
    storage location
    queue/topic
    event
    database
    table
    API
    repository
    code module
    dependency/library
    deployment
    environment
    infrastructure resource
    feature
    team
    alert
    incident
    runbook
    external dependency

Determine the actual internal vocabulary.

Identify alias problems.

Examples:

    service-a
    service_a
    old-service-name
    nickname used by operations

may or may not refer to the same thing.

Determine whether a canonical identifier can be built.

Study candidate relationships such as:

    produces
    consumes
    triggers
    requires
    depends_on
    calls
    publishes
    subscribes_to
    reads_from
    writes_to
    transfers_to
    scheduled_by
    implemented_by
    deployed_from
    configured_by
    owned_by
    monitored_by
    observed_on
    changed_by
    affected_by

Again, these are examples.

Discover the relationships that actually matter internally.

For every persisted relationship, consider storing:

    source
    evidence
    verification state
    environment
    observed time
    last verified time
    validity interval

## Knowledge architecture

Do not assume that all internal information belongs in one RAG/vector index.

Classify each source by the correct retrieval mechanism.

Examples:

    current telemetry
        -> targeted live query

    deployment/change records
        -> exact lookup

    scheduler state
        -> scheduler API/query

    current storage state
        -> storage query

    dependency relationships
        -> graph/catalog lookup

    code
        -> code/search/index tools

    ownership
        -> authoritative ownership system where available

    documentation/runbooks
        -> lexical + semantic search

    historical conversations
        -> scoped search

    prior incidents
        -> incident retrieval

Determine which data must be copied/indexed and which should remain at source.

Prefer source references over unnecessary duplication.

## Freshness

Determine how each important source changes.

For every source, identify whether freshness could be maintained using:

    webhooks
    events
    change feeds
    repository commits
    deployment events
    scheduled polling
    runtime observations
    periodic validation

The eventual system should recognize that stored knowledge can become stale.

Investigate a memory lifecycle:

    discovered
    candidate
    verified
    active
    stale
    contradicted
    superseded
    forgotten

Determine which memories can be automatically revalidated.

## Annotation and labeling

Do not begin a broad manual annotation program.

First determine which metadata and relationships can be extracted
deterministically from existing systems.

Identify which relationships can be learned from:

    source code
    configuration
    manifests
    scheduler definitions
    runtime telemetry
    deployment metadata
    existing catalogs
    documentation

LLM extraction from prose should create candidate facts, not automatically
trusted facts.

Human review should focus on high-value relationships and evaluation cases.

## Historical evaluation dataset

Find a small representative set of past operational incidents.

Prefer diversity over volume initially.

For each candidate incident determine whether we can reconstruct:

    original symptom
    evidence available at incident start
    affected entities
    relevant timeline
    actual root cause
    resolution
    misleading evidence
    downstream effects
    responder investigation path

Watch for future-data leakage.

Do not allow an evaluation agent to see:

    later postmortems
    fix commits that were not available yet
    later messages revealing root cause
    later documentation

unless they were actually available at the point being simulated.

Design an initial backtesting strategy.

Propose evaluation metrics including:

    root-cause correctness
    evidence correctness
    evidence completeness
    hypothesis quality
    time to useful context
    tool-call efficiency
    tokens per investigation
    stale-knowledge errors
    human correction frequency
    permission violations
    sandbox reproduction accuracy

## Procedures and skills

Study how humans currently troubleshoot recurring incident classes.

Look for repeated sequences such as:

    check X
    then Y
    if result A, inspect Z
    otherwise inspect Q

Do not blindly convert every historical sequence into a rule.

Determine which parts are:

    stable procedure
    useful heuristic
    one-time workaround
    historical accident
    obsolete behavior

Propose a "skill" format capable of storing:

    purpose
    trigger conditions
    applicable entities/environments
    required tools
    prerequisite evidence
    suggested checks
    branching conditions
    stopping conditions
    escalation conditions
    known limitations
    owner
    version
    last verification date

Skills should guide the agent while still allowing investigation outside the
skill when evidence justifies it.

## Token and cost discipline

Token efficiency is a primary architectural requirement.

Measure it from the first prototype.

Do not load:

    entire repositories
    entire log streams
    entire chat histories
    entire documentation collections
    every available tool definition

into model context.

Investigate:

    progressive retrieval
    targeted queries
    lexical search
    semantic retrieval only where useful
    reranking
    tool discovery on demand
    deterministic filtering
    result reduction
    caching
    reusable structured facts
    incremental indexing
    model routing
    explicit tool budgets
    explicit token budgets
    investigation stopping rules

For a large result set, return a small structured summary plus a source
reference rather than the complete payload when possible.

Determine which tasks require a strong reasoning model and which can use:

    deterministic code
    SQL
    graph queries
    search
    small/cheap models

## Code and deployment correlation

Determine whether the real environment permits the following chain to be built:

    runtime entity
        ->
    deployed version
        ->
    deployment event
        ->
    commit/revision
        ->
    changed files/modules
        ->
    relevant implementation

Inspect how deployments are currently recorded.

Determine whether commit/revision metadata reaches runtime telemetry.

Identify existing dependency manifests and lock files.

Study existing code-search/navigation capabilities before proposing a new index.

Do not assume that embedding the entire repository is sufficient.

## Human-visible investigation workspace

Design concepts for an interface where a human can watch an investigation.

The interface should eventually expose:

    affected entity

    dependency map

    upstream entities

    downstream entities / likely blast radius

    current hypotheses

    validated hypotheses

    rejected hypotheses

    inconclusive hypotheses

    live investigation events

    tool/query used

    evidence returned

    links to authoritative source

    recent code/config/deployment changes

    prior incidents

    proposed checks

    permission boundaries

    sandbox experiments

    proposed actions

The human must be able to inspect why the system believes a relationship or
claim.

Do not expose hidden model monologue as the product.

Expose the auditable investigation state and evidence trail.

Consider visual states for graph relationships:

    VERIFIED
    OBSERVED
    INFERRED
    HUMAN-CONFIRMED
    STALE
    DISPUTED

## Permissions and production safety

Inventory the actual permissions available to the intended users.

Separate:

    what can be observed
    what can be queried
    what can be executed
    what requires infrastructure/security/other-team approval

Do not assume that access through Kubernetes tooling permits deployment or
mutation.

The eventual agent must operate within the same authorization boundaries as
its user/service identity.

A model recommendation is not authorization.

Production writes, if ever supported, must pass through deterministic policy
and workflow controls outside the model.

Consider an autonomy ladder:

    OBSERVE
        retrieve evidence

    RECOMMEND
        recommend next action

    PREPARE
        prepare exact action/request

    EXECUTE WITH APPROVAL
        authorized human approves deterministic workflow

    NARROW POLICY-BASED EXECUTION
        only for explicitly approved low-risk failure modes

The discovery phase must remain read-only.

## Infrastructure-team boundary

Determine which operations only an infrastructure or platform team can
perform.

For those operations, design an escalation artifact containing:

    requested action
    exact target
    why it is requested
    supporting evidence
    expected result
    known risk
    verification procedure
    rollback considerations

The investigation should remain useful even when the agent cannot itself
perform the next operation.

## Sandbox and incident reproduction

Discover what lower/development/sandbox environments actually exist.

Do not assume Terraform, Kubernetes, containers, or any specific technology is
available.

Determine whether selected failures could be reconstructed using some
combination of:

    unit/process-level reproduction
    synthetic inputs
    sanitized historical inputs
    containers
    service mocks
    local infrastructure
    emulated external services
    replayed messages/events
    lower environments
    temporary namespaces/clusters
    ephemeral infrastructure
    fault injection

If infrastructure-as-code exists and users are permitted to create isolated
resources, evaluate whether it can support temporary reproduction
environments.

The agent should record environment differences and reproduction confidence.

Possible outcomes must include:

    REPRODUCED
    PARTIALLY REPRODUCED
    NOT REPRODUCED
    CANNOT REPRODUCE WITH AVAILABLE ACCESS

Never claim sandbox behavior proves production causality without supporting
evidence.

Do not automatically copy production secrets or sensitive production data into
a lower environment.

## Architecture alternatives

Compare at minimum:

    A. Incident-first architecture

    B. Knowledge/catalog-first architecture

    C. Graph-first architecture

    D. Skill/runbook-first architecture

    E. Hybrid incident-first + thin context graph + skills + evidence ledger

Do not choose based on fashion.

Choose based on observed internal requirements.

Specifically determine whether a dedicated graph database is actually needed
or whether a relational entity/relationship representation is sufficient for
the first implementation.

Likewise determine whether a dedicated vector database is required or whether
existing search/index systems already solve the problem.

## Initial implementation candidate

Evaluate, but do not blindly adopt, a minimal architecture shaped like:

    incident/question
        ->
    entity resolver
        ->
    context builder
        ->
    hypothesis planner
        ->
    tool router
        ->
    targeted queries
        ->
    result reducers
        ->
    evidence ledger
        ->
    hypothesis updater
        ->
    human-visible investigation
        ->
    reviewed memory

with independent components for:

    operational context graph
    skills
    historical evaluation
    policy/permissions
    sandbox execution

## Discovery deliverable

Produce a detailed report.

The report must include:

### Environment inventory

What systems actually exist.

### Information-source inventory

What each system knows.

### Permission matrix

What the intended agent identity can read/write and what requires another
team.

### Current troubleshooting workflow

Reconstruct several real human investigation paths.

### Entity vocabulary

What nouns actually matter internally.

### Relationship vocabulary

What dependency types actually matter.

### Identifier problems

Aliases, inconsistent names, missing environment identifiers, etc.

### Existing metadata

What relationships can already be obtained without AI.

### Existing search/index capabilities

Avoid rebuilding tools that already exist.

### Historical incident availability

Whether useful backtesting is possible.

### Sandbox capability

What can realistically be reproduced outside production.

### Knowledge freshness model

How important sources can remain current.

### Token-cost model

Estimate retrieval and model consumption for several representative
investigations.

### Architecture comparison

Compare candidate designs against the actual environment.

### Recommended narrow prototype

Select one useful incident/problem class.

Choose the minimum set of connectors needed.

Define success/failure criteria.

### Evaluation plan

Define historical cases and metrics before broad rollout.

### Risks and unknowns

Explicitly state what has not yet been established.

## Evidence requirements for this discovery

Every important conclusion must cite internal evidence where possible:

    repository path
    code symbol
    configuration path
    documentation page
    API/resource
    deployment metadata
    dashboard
    query
    incident
    historical message
    runbook

Separate:

    VERIFIED FACT
    INFERENCE
    RECOMMENDATION
    UNKNOWN

Do not present guesses as architecture facts.

## Final constraint

Do not optimize for impressive autonomy.

Optimize for:

    correctness
    evidence
    inspectability
    low token consumption
    low query volume
    freshness
    permission safety
    usefulness to a human responder

The first successful system should make a human investigation substantially
faster and easier to understand.

Autonomy can be earned later.
