# Fast Incident Investigation: Parallel Hypothesis Paths, Correlation, and Precomputed Operational Context

**Research and design extension — 26 October 2026**  
**Status:** Public, technology-neutral proposal; not verified against any private environment.  
**Read with:** [Initial internal-agent handoff](../AGENT_HANDOFF.md), [multi-agent workbench](MULTI_AGENT_OPERATIONS_WORKBENCH.md), [contrarian/explorer roles](CONTRARIAN_AND_EXPLORER_ROLES.md), and [deep research](DEEP_RESEARCH_REPORT.md).

> **Security:** This public repository is reference material only. Never publish internal system names, code, logs, identifiers, topology, credentials, tickets, traces, or investigation results here. The authorized internal agent must store all findings in an approved **internal** workspace. No production writes or new scans are authorized by this document. Only run approved bounded read-only checks within existing access.

## 1. Problem statement, from the request

The user observes that an investigation can take **around two hours**, with the agent seemingly solving small discovery problems serially: reading code to work out the system, deciding what might need to be fetched, calling a tool, reading the result, then discovering the next tool. This is **a reported symptom to benchmark**, not a measured performance result or evidence that a particular implementation has these flaws.

The requested improvement:

- On incident arrival, form **up to three genuinely different possible failure pathways**.
- Launch small, independent, parallel evidence retrieval tasks **before** the agent commits to one pathway.
- Reuse already-known system relationships, code locations, query templates, and earlier verified investigations, instead of re-deriving them.
- As evidence accumulates, **prune or park less promising investigation branches** and cancel unnecessary outstanding queries; preserve contradictory evidence and provenance.
- Defer deep reading / code exploration until a cheap discriminator shows it is needed.
- Ask whether pathways should be **cached, indexed, or represented in another form**.

**Answer:** Use all of these, but for *different purposes*. Cache stable context and query plans; index searchable entities, relations, symbols, and time-scoped observations; encode repeatable troubleshooting paths as **versioned, conditional procedures**; retrieve genuinely live state from authoritative systems. A cache of old root-cause guesses is not a substitute for current evidence.

## 2. External research: what fast systems actually do

The table distinguishes **documented product behavior** from **design inference**. Vendor descriptions are not proof of comparable results in this organization's environment.

| Evidence / system | Documented capability | Transferable implementation idea |
| --- | --- | --- |
| **incident.io Investigations** (engineering post, 1 Oct 2026) | Begins investigations with **parallel fan-out** across telemetry, deploys, messages, docs, and code. Compresses data into evidence-backed findings, updates them **in waves** rather than recomputing everything, and runs adversarial hypothesis checks. It maintains **Nexus**, a living model of the environment, and periodically revalidates useful query memories. | Parallel *I/O*, structured fact deltas, continuous knowledge maintenance, falsification before declaring cause. [Source](https://incident.io/blog/building-investigations-what-it-takes-to-build-an-ai-sre) |
| **Grafana Knowledge Graph / RCA Workbench** | Discovers entities and relations from existing metrics/labels, **indexes the graph**, attaches and indexes condensed health insights, and correlates by entity and time. Its telemetry-correlation configuration maps entity properties to log/trace labels so investigators get pre-filtered queries. | Precompute entity and relationship resolution, health summaries, and query-join mappings, rather than deriving them via ad-hoc prompts. [How data is processed](https://grafana.com/docs/grafana-cloud/platform/knowledge-graph/reference/how-knowledge-graph-processes-data/), [Telemetry correlation](https://grafana.com/docs/grafana-cloud/platform/knowledge-graph/configure/telemetry-correlation/), [RCA Workbench](https://grafana.com/docs/learning-paths/rca-workbench-investigation/) |
| **Datadog Bits AI SRE** | Documents a hypothesis -> targeted telemetry query -> validate/invalidate -> revise investigation loop across traces, logs, metrics, change events, and other products. Explicit service tags/scoping improve correlation. | Query only the relevant entity, interval and signal; do not scan everything. The public docs **do not establish** that every step is parallel or precomputed. [Source](https://docs.datadoghq.com/bits_ai/bits_investigation/investigate_issues/) |
| **Sentry Seer** | Uses error/issue context, traces, logs, profiles, and linked code; issue scans enrich incoming issues before manual deep debugging. | Attach incident-specific context to trigger events so investigation starts further along. [Source](https://docs.sentry.io/product/ai-in-sentry/seer/) |
| **OpenTelemetry** | Shared context / trace and span IDs can connect work across services; messaging conventions describe producers/consumers, conversation IDs, links, and async boundaries. | High-quality correlation **keys** are often more valuable than extra embeddings. For async/batch flows, missing producer-to-consumer context must be acknowledged, not hallucinated. [Context propagation](https://opentelemetry.io/docs/concepts/context-propagation/), [Messaging](https://opentelemetry.io/docs/specs/semconv/messaging/messaging-spans/) |
| **GitHub Code Navigation** | Symbol definitions/references are extracted using parsers (tree-sitter for supported languages), allowing targeted navigation without repeatedly reading entire repos. | Reuse existing repository code navigation and deployed-revision mapping; do not build a new embedding index before proving symbol/lexical search inadequate. [Source](https://docs.github.com/en/repositories/working-with-files/using-files/navigating-code-on-github) |

**Research judgment:** The apparent speed advantage is a **systems architecture**, not a special prompt or simply adding features: persistent environment knowledge, stable IDs, normalizing and indexing useful signals in advance, cheap targeted queries, parallel I/O, incremental evidence synthesis, and strict stopping conditions. Vendor latency claims must not be treated as internal benchmarks.

## 3. Separate the four mechanisms (do not conflate them)

| Mechanism | Store/compute | Benefit | Invalidates when |
| --- | --- | --- | --- |
| **Entity/relationship index** | Canonical ID, aliases, producer, consumer, owner, scheduler, resource links, graph edges with provenance, environment and time validity; reverse dependency lookup. | Instantly answer "what is this?" / "what depends on it?" | Relevant topology, config, ownership, deployment or confirmed fact changes. |
| **Code/search index** | Repository+revision -> symbols, definitions, references, entry points, config fields, job handlers, query templates; map *deployed revision* to source, not only HEAD. | Navigate straight to a relevant handler or config without repo-wide rereads. | Commits/deployments/config changes. |
| **Procedural/diagnostic memory** | Versioned *decision trees*: incident signature -> initial checks -> branch conditions -> follow-ups; last verification, applicability, exceptions. | No rediscovery of repeated investigative procedures; avoid unnecessary LLM deliberation. | Query fails, schema/log format changes, procedure becomes stale, or counterexample appears. |
| **Bounded hot cache / event summaries** | Source pointers, pre-aggregated recent anomalies, health states, small query result summaries, tool schemas, incident-specific memoization. | Lower initial latency and avoid duplicate queries. | Short TTL, new event, changed incident window, permissions or source revision; live statuses are always rechecked when material. |

An index answers **where** to look. A correlation key answers **which records belong together**. A procedure answers **which question to ask next**. A cache saves repeated work. The LLM should primarily handle ambiguity, exceptions, cross-source explanation, and evidence evaluation.

A knowledge graph can be backed by tables/JSON/relational indexes at first. A dedicated graph database is an **implementation option**, not a prerequisite.

## 4. Correlation: the likely missing prerequisite

Correlation should be established in this priority order, as availability permits:

1. **Strong exact joins:** shared trace/span, execution/run, event/message, transaction, batch, transfer, and artifact IDs. For async messaging, propagate message creation context; where traces do not span the boundary, build sanctioned links from producer/event to consumer via exact IDs and time.
2. **Scoped structural joins:** environment + canonical entity + topic/queue + scheduler/job + deployed revision + ownership + relationship.
3. **Temporal candidates:** changes, missing expected signals, upstream errors, downstream failure within a bounded time window. Correlation by timestamp alone is **not causation**.
4. **Fuzzy candidates:** aliases, text search, similar historical patterns, embeddings. Never silently promote a fuzzy match to a verified dependency.

A small **correlation manifest** is more useful than a pile of raw tool output. Example generic fields (not claims that they already exist):

```yaml
incident_ref: <opaque-id>
environment: <env>
time_window: <start..end>
canonical_entity_ref: <catalog-ref>
execution_ref: <optional>
event_or_message_ref: <optional>
trace_ref: <optional>
deployed_revision_ref: <optional>
artifact_ref: <optional>
relationship_refs: []
evidence_refs: []
missing_join_keys: []
```

Important failure modes: absent propagation across queues and batch jobs; retries creating new execution IDs; clock skew/time-zone issues; aliases mis-resolving; log sampling/retention; different environments sharing names; stale ownership; business-date versus clock-date confusion. Log presence with a trace ID does not guarantee the trace was retained. See [Datadog log-trace correlation](https://docs.datadoghq.com/tracing/other_telemetry/connect_logs_and_traces/).

## 5. Three-path speculative investigation with progressive pruning

For a **hypothetical** missing expected output (not a claim about any system), start with three alternatives:

- **Path A: Not expected or never triggered.** Is this a valid run date? Were conditions, inputs, scheduler trigger, or preconditions satisfied?
- **Path B: Triggered, generation failed.** Is there an execution/run? Did an upstream input, processing step, error, or dependency prevent completion?
- **Path C: Generated, but not delivered/recognized.** Was the artifact/event produced? Was it staged, transferred, renamed, filtered, delayed, or accepted by the consumer?

### Phase 0: deterministic intake (no deep LLM research)

Extract symptom, time window, environment, possible entity/alias, and explicit missing information. Resolve canonical entity + top 1-2 hop dependencies + approved tool manifests from the index. Load only a small *orientation packet* and relevant decision-tree candidates. If entity/time/scope are unresolved, do not run broad speculative scans.

### Phase 1: parallel **cheap discriminator queries**

Fan out **independent read-only lookups** using a scheduler, not necessarily three large LLMs:

| Path | Initial cheap read | Decision-changing result |
| --- | --- | --- |
| A | expected-state / schedule rule + latest trigger status | Not expected, not triggered, or triggered |
| B | run state / bounded failure summary + upstream prerequisite status | No run, run failed, or run succeeded |
| C | existence/metadata of expected output + transfer/consumer receipt | Never created, created but not handed off, delivered/received |

These should share exact scope and request ID. A **single-flight deduplicator** merges identical requests from multiple paths. Enforce per-source concurrency and rate limits; a fan-out that overwhelms logging/telemetry is not an improvement.

### Phase 2: revise and prune

As each query completes, write a **small source-backed evidence delta** immediately. If evidence strongly rules out one path, mark it **parked** (do not delete it). Cancel pending *low-value* requests where safe; keep conflicting findings in the ledger. Continue one or at most two credible branches. Path selection is based on evidence, not on which agent speaks most confidently or answers first.

When two paths remain equally plausible, select the next query with highest **expected discrimination per cost**. Cost includes latency, query limits, model tokens, security risk and potential operational impact. Use configurable heuristics rather than claiming calibrated probabilities without data.

### Phase 3: targeted deep investigation

Only now retrieve exact code symbols, a bounded log slice, deployment diff, detailed traces or further historical cases justified by the remaining hypothesis. Query the code at the **deployed revision** where possible. A coding specialist can investigate a bounded question using an existing code index; it should not independently reconstruct the entire architecture.

### Phase 4: conclude, abstain, or escalate

Emit a compact causal narrative with source references, counterevidence, uncertainty, and next permitted check. The Contrarian is invoked only at meaningful claim/risk gates; the Explorer only if discriminating progress stalls. No looping debate. Reusable verified procedure changes can be reviewed into memory.

## 6. Proposed execution model (illustrative pseudocode)

```text
onIncident(incident):
  scope = parse_and_validate_scope(incident)
  orientation = catalog.lookup(scope.entity, scope.environment)
  plans = procedure_registry.select(scope, orientation, max_paths=3)

  tasks = []
  for plan in plans:
      tasks += plan.first_cheap_read_only_checks

  # Bounded concurrency, shared tool budgets, request deduplication.
  in_flight = scheduler.submit_deduplicated(tasks, scope, permissions)

  while in_flight and within_budget:
      for result in scheduler.next_completed_wave():
          finding = validate_and_reduce(result)       # tiny summary + source link
          ledger.upsert(finding)                       # no raw log dumps to LLM
          hypotheses.update_with_provenance(finding)

      park_paths_with_specific_refuting_evidence()
      scheduler.cancel_only_truly_unneeded_work()
      next_checks = rank_by_expected_discrimination_and_cost()
      scheduler.submit_deduplicated(next_checks, scope, permissions)

      if evidence_sufficient_for_claim(): break
      if no_useful_safe_check_remains(): break

  review_causal_claim_and_counterevidence()
  emit_status_with_evidence_and_unknowns()
  queue_only_approved_memory_candidates_for_review()
```

A **wave** is arrival of enough new findings to change a decision; do not block until the slowest parallel tool returns. Do not restart the whole prompt/workflow on every wave. Share **typed evidence references**, not entire multi-agent chat histories.

### Agent organization

- **Coordinator:** path budget, permissions, common scope, hypothesis table, deduplication, early stop.
- **Specialists:** optional read-only tool-specific adapters, preferably deterministic code for simple queries; invoke LLM specialists only for complex synthesis.
- **Contrarian / Explorer:** event-triggered, short evaluations, *not* permanently reasoning.
- **Shared finding store:** atomic updates, deduplication, source attribution, historical version and invalidation.

The user asked for subagents working in parallel; **parallel tool calls can often deliver most of the latency improvement with fewer model tokens**. Benchmark orchestration before adding more LLM agents.

## 7. What to precompute *before* an incident

**Continuous / event-driven (if permitted):**

- Canonical entity/alias lookup, known dependency edges and reverse dependencies.
- Runbook and procedure index (branch predicates and cheap tool calls).
- Existing observability labels, query templates and which labels join to which entities.
- Deployed revision -> symbols / config / source locations.
- Prior incident signatures with *verified* failure mechanisms and source pointers.
- Recent bounded summaries of change events, health trends, missed expected signals, and last successful run **where permitted and already available**.
- Freshness metadata and invalidation hooks on deployment/config/relationship changes.

**At incident trigger:**

- Resolve scope; begin small **path-independent** reads (current entity health, recent relevant changes, expected status and alert context).
- Fan out the first cheap checks for up to three paths. Cache resulting evidence for this incident using a scoped key that includes identity, environment, time window, auth, query parameters, and source watermark.

**Only when a path survives:**

- Pull deep logs, large traces, code diffs, object detail, incident conversations or payload-adjacent records.
- No bulk background harvesting of sensitive data, whole-code embedding, or storing full raw telemetry in an external cache.

### Suggested invalidation policy to validate internally

- Static metadata: versioned; refresh on repository/catalog changes, plus periodic verification.
- Query templates: revalidate on schema changes and observable failures.
- Runbook procedures: owner/version/reverification date; expire after disproving counterexamples.
- Dynamic health, scheduler state and file existence: short-lived / request-scoped; **do not treat cached state as proof of a current condition**.
- Historical incident evidence: immutable historical records, but labels/conclusions can be corrected.
- Access scope: cache segregated by tenant, environment and authorization; permission revocation invalidates reuse.

## 8. Latency budget and diagnostic instrumentation

Before designing extra infrastructure, measure a single representative incident's **critical path**:

```
time_to_first_useful_context
  = parse/resolve + longest initial essential read + small synthesis

time_to_supported_diagnosis
  = orientation + critical-path query waves + hypothesis updates
    + evidence validation and claim review
```

Record a timestamped span for:

- Model setup/classification and context size.
- Entity resolution, graph lookup and cache hit/miss.
- Query planning and authorization checks.
- Each tool request: queue wait, rate limit, service latency, result size and reduction.
- Sequential dependence versus parallel overlap.
- Code navigation time and what step forced deep code reading.
- Repeated or duplicate queries, retries, idle waits, handoffs and full-context replays.
- First meaningful finding, first ruled-out path, first defensible conclusion.
- Accuracy, contradictions found, missing join keys, wasted queries, tokens and tool/observability cost.

**Illustrative math only, not a forecast:** Three independent 25-second reads done serially cost ~75 seconds of waiting. If submitted concurrently with adequate tool capacity, their wall-clock waiting can approach ~25 seconds. Real end-to-end savings are reduced by coordinator overhead, quotas, contention, dependencies and further waves.

Set *trial targets* only after recording baseline (e.g. reduce p50 time-to-first-relevant-evidence, p95 diagnosis time, duplicate fetches and input tokens; hold correctness fixed or improve it). **Do not promise a five-minute RCA or a percentage speedup without a time-correct benchmark.**

## 9. Experiments to distinguish indexing, parallelism, and correlation

For a small permitted synthetic or historical **time-frozen** set of incident types, compare:

1. **Baseline:** current serial agent and current tools; instrument fully.
2. **Bounded fan-out only:** same indexes/tools, parallel independent cheap reads.
3. **Catalog + exact correlation joins:** add canonical IDs, pre-resolved dependencies and query label mappings; keep fan-out.
4. **Procedures and cache:** add versioned query plans, change-based invalidation and per-incident deduplication.
5. **Triggered specialists (optional):** only where the preceding variants demonstrably leave difficult ambiguity.

Compare correctness *and* time to evidence, time to supported diagnosis, p50/p95, duplicate fetches, unresolved alias rate, cache staleness, missed root causes, telemetry load, tokens, model cost and investigator review time. Randomize case order or replay deterministically. Reconstruct only evidence genuinely available at the incident timestamp to avoid future-data leakage. Include no-incident and misleading-recent-deploy counterexamples.

**Possible outcome:** If (2) wins most of the time, a new knowledge store is premature. If (3) wins, correlation/identity is the bottleneck. If (4) wins, recurring decision-path rediscovery is expensive. If none wins, measure permission delays, source query slowness and insufficient instrumentation before adding agents.

## 10. Assignment for the authorized internal agent

**Do not jump to implementation.** Produce an internally stored, source-backed **latency and correlation assessment** for a narrowly authorized sample:

1. Break down the observed long investigation into a wall-clock critical-path flamegraph/table. Distinguish LLM reasoning, tool latency, waiting, code traversal and repeat work.
2. Inventory authoritative entity IDs, alias maps, execution IDs, batch/message/trace IDs, deployment versions and timestamps. Identify where exact cross-system joins work or fail.
3. Inspect existing search/indexing features (code symbols, observability facets, catalog, saved queries, dependency topology). Explain what **already exists** that could replace new infrastructure.
4. Draw a single approved example as an evidence-backed graph from symptom -> expected action -> producer -> output -> delivery -> consumer, marking verified/inferred/unknown/stale. Do not disclose the real graph in this public repository.
5. Write a 3-path hypothesis plan for that example, with 1-2 **bounded read-only** discriminator checks per path, execution dependencies, cost estimates, and explicit pruning conditions.
6. Identify which data belongs in persistent indexes, short-term caches, executable procedures, or direct live queries. Define invalidation and permission isolation.
7. Compare serial versus parallel execution on the same approved evidence using the experiments above. Include false-cause and incomplete-trace cases.
8. Return the **smallest demonstrably effective change** (e.g. shared ID + three parallel query templates + evidence ledger, possibly no new database) and exactly what additional data/permissions would be needed to make more improvements.

**Deliverables to keep internal:** critical-path timings, existing-capability matrix, join-key gap report, compact runbook schema, three-path query DAG, benchmark design/results, safety gates, incremental rollout plan. **Nothing from the internal investigation gets pushed to this public repo.**

## 11. Final engineering position

This is not primarily an “agent thinks faster” problem. It is the interaction of:

**(a) unknown system identity + (b) missing correlation keys + (c) repeated topology/code discovery + (d) serial tool execution + (e) large repeated model contexts.**

Optimize in that order **only if measurements support that ordering**. Treat each incident as an evidence-discrimination problem: resolve the entity once, launch a small fan-out of cheap checks, reuse verified knowledge, follow the surviving paths, and stop when further checks will not change the conclusion.
