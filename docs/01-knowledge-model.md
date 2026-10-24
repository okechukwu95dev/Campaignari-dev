# 01 — A living knowledge model, not a pile of embeddings
## Why this matters
A human might say "Output X is missing." The agent must understand **what X is** before doing diagnostics. The core problem is knowledge organization across code, documentation, operational data, conversations, traces, schedules, ownership, and repeated incidents.

The aim is to establish an **operational ontology**: agreed names and types for the things that matter, and evidence-backed relations between them. Start tiny. Do not build an enterprise-wide ontology in advance.

## 1. Separate facts from prose
There are at least four different stores of useful information, which do not need four databases:

- **Sources:** original code, docs, log events, alerts, chat threads, change records and audit evidence. They are authoritative only within their scope.
- **Canonical facts:** narrow claims such as "A triggers B", "C consumes D", "E owns F" with source references, observation window and verification status.
- **Operational cases:** past symptoms, relevant context, checks, observations, tested hypotheses, outcome and review.
- **Procedures / skills:** approved steps or reusable diagnostic queries, including when to use them, expected observations, safety class, inputs, owners and last validation.

A vector index is a search accelerator, not the canonical truth. A graph is a relationship representation, not by itself proof that a relation is real.

## 2. The minimum useful types
Do **not** assume these types exist in the target system. Discover them, then add only what evidence demands.

Possible entity categories:
- business concept / user-facing feature / operational process;
- artifact / file / logical dataset / message / report;
- scheduled definition and individual execution/run;
- software component / service / function / worker;
- endpoint / interface / protocol contract;
- queue, topic, stream or event type;
- storage location / logical table / schema;
- environment / deployment / version / change;
- owner / escalation group / handbook;
- incident / symptom / evidence / hypothesis / test / remedy.

Possible typed relations:
- **produces**, **consumes**, **requires**, **blocks**, **triggers**, **scheduled-by**;
- **reads-from**, **writes-to**, **publishes**, **subscribes**, **calls**, **handles**;
- **deployed-as**, **configured-by**, **changed-by**, **owned-by**;
- **observed-in**, **reported-in**, **investigated-with**, **resolved-by**.

A code import is not equivalent to a runtime call. A code comment describing ownership is not proof of current ownership. A single log line mentioning a file is not proof of a producer/consumer relationship.

## 3. Entity record: illustrative fields
Every field is optional until evidenced. Do not fabricate missing values.

- ID: internal stable reference; not human-readable raw secrets.
- Type, canonical display label, known aliases, namespace/scope.
- Short business meaning (one sentence) and technical meaning (one sentence).
- Owner, lifecycle, environment and known versions, where applicable.
- Evidence pointers with access-control labels.
- First observed, last verified, valid-from/valid-to (when applicable).
- Confidence class: observed/verified/inferred/conflicted/unknown/stale.
- Reviewer/source-of-truth, not just LLM-generated assertion.
- Sensitivity and retention rules.

## 4. Relationship record: illustrative fields
- Subject ID, relation type, object ID.
- Evidence identifier and precise source anchor; source time; method of extraction.
- Observed runtime / static code / reviewed documentation / human report / hypothesis.
- Time interval and environment applicability.
- Review status, reviewer, contradictions, last checked, expiry/revalidation trigger.
- Optional condition: a dependency may exist only under a certain configuration.

**An edge without traceable evidence should normally remain a candidate edge.**

## 5. Incident case record
A useful case need not contain a long essay:

- Case ID, scope, symptom, observed timeframe, entity IDs and change window.
- Confirmed observations; relevant missing observations; false leads.
- Hypothesis list and discriminating test for each.
- Tool/action event references and their safe outputs.
- Actual resolution, verification, rollback/recovery result if any.
- Impacted graph path, with observed versus potential impact distinguished.
- Human verification state and reusability conditions.
- Links to canonical procedures or diagnostic queries that were validated.
- Sensitivity and expiration/review date.

Two incidents with similar messages may have unrelated causes. Match by precise conditions and evidence, not by keywords alone.

## 6. Do we need data labeling and annotation?
**Yes, selectively. No, not as a massive project before proof of value.**

Five levels of labeling, in order:
1. **Deterministic metadata labels:** source type, timestamp, environment, repo path, message thread, trace ID, log level, deploy revision, permission/sensitivity label. Often extracted without an LLM.
2. **Entity recognition and aliases:** candidate service, workflow, artifact, schedule, process, API or business term. Use code symbols, structured metadata, patterns and targeted model extraction.
3. **Relations:** typed producer/consumer/call/trigger/owner edges linked to evidence; prefer observed runtime or reviewed code before inference.
4. **Incident cases:** symptom, checks performed, observations, hypothesis, resolution, verification.
5. **Human-reviewed skills:** approved reliable procedures with inputs, preconditions, expected evidence, failure modes, owner and safety classification.

Useful dimensions to tag when they genuinely change retrieval: source, time, service/domain, entity type, lifecycle stage, environment, protocol category, impact class, state, approval class, evidence quality, owner. Resist hundreds of one-off tags. A taxonomy that nobody can maintain is worse than narrow accurate labels.

### Recommended approach
- Auto-label cheap, explicit metadata first.
- For ambiguous claims, collect candidates and ask an authorized engineer to confirm the small high-value subset.
- Keep a human correction queue: merge duplicate names, split ambiguous terms, reject invented edges, mark stale explanations.
- Use actual incident reviews as annotations so labeling produces direct operational value.
- Retain provenance and uncertainty rather than forcing every record into an artificial ground-truth category.
- Build a small high-quality **gold set** of reviewed examples for evaluation. The remaining corpus may stay unreviewed.

## 7. Extracting knowledge from human incidents
Collect **observable actions**, not hidden mental processes:
1. Incident reported.
2. Engineer chose a source and a scoped query.
3. Query returned a result.
4. Engineer investigated a second source to test a theory.
5. Evidence ruled something in or out.
6. Engineer requested an infrastructure action, ran an approved test, or escalated.
7. Outcome and verification were recorded.

Retrospectively summarize these actions into a concise case; link each action to evidence. Distinguish "we ran command Q" from "because of X we reasoned Y": reconstruct the latter as an explicit, reviewable operational hypothesis, never claim to possess someone's private chain of thought.

A useful skill record should say:
- **When:** observed conditions that trigger the check.
- **Read:** permitted sources and narrowly scoped lookups.
- **Interpret:** result patterns and alternatives.
- **Stop:** when evidence is insufficient or permissions fail.
- **Escalate:** if action needs another owner.
- **Verify:** what confirms actual recovery.
- **Cost and risk:** expected tokens, calls and side effects.
- **Validity:** supported versions, date and human reviewer.

## 8. Fresh knowledge, low cost
Do not frequently re-embed or re-summarize everything.

**Initial crawl:** respect source ACLs, gather metadata and a small scoped sample, hash content for deduplication; defer embeddings.

**Incremental refresh:** prefer official webhooks/events for code commits, changes and incident state; polling with checkpoints where no push integration exists; process only changed records.

**Runtime data:** usually query at incident time with strict time bounds instead of indexing all raw logs. Keep pointers, normal event metadata and important summaries, not a duplicate log lake.

**Change detection:** invalidate facts linked to changed code paths, schemas, configs, deleted docs or renamed entities; refresh only affected graph neighborhoods.

**Conflict handling:** surface disagreements with timestamps, source rank and version. Never silently let a newer chat message override a verified runtime relation.

**Lifecycle:** fresh -> needs review -> stale -> retired; define triggers per source and entity type. Production facts may need near-real-time checks; historical procedures need review on underlying change.

**Permissions:** enforce access controls before retrieval and after joining records. Do not leak an inaccessible incident through a derived summary, graph edge or embedding.

## 9. Retrieval strategy — cheap before expensive
In increasing cost order:
1. Exact entity name, alias and metadata filter.
2. Scoped keyword/full-text search.
3. Verified 1–2 hop relationship expansion.
4. Targeted source query (e.g. a time-bounded event lookup).
5. Semantic/vector retrieval for ambiguous descriptions or past-case similarity.
6. Reranking a small candidate set.
7. Model reasoning with a compact evidence packet.
8. Expensive multi-source or graph-wide exploration only if earlier steps fail.

A cache key must include source version, access scope, query scope, freshness window and model/prompt version when relevant.

## 10. Why not mandate a graph database?
A simple relational edge table can power the first dependency explorer. Search indexes can power definitions. A graph database becomes more attractive if:
- queries repeatedly need long transitive paths;
- dependency changes need complex graph operations;
- multi-hop ownership/impact queries dominate;
- the team can support graph versioning, security and maintenance.

Measure the graph benefit against simpler methods on real approved questions. General-purpose GraphRAG extraction can be expensive and hallucinated edges are particularly damaging here.

## 11. A worked generic example — "expected artifact X missing"
Unknowns before looking: what is X? Which clock and deadline? Was it ever expected today? Is X a file or merely an event name?

Agent finds an approved glossary entry for X; links an artifact ID; locates a versioned producer definition; finds a scheduler event that allegedly triggers the producer; checks actual run state; retrieves runtime evidence that output was or was not emitted; explores consumers where evidence supports them.

Graph might show:
Prerequisite P -> (requires) -> Job J -> (produces) -> Artifact X -> (consumed by) -> Consumer C.

**This is hypothetical until source evidence exists.** One edge can remain inferred while others are verified. Show impact as potential unless there is observable evidence that Consumer C was blocked.

## 12. Questions to evaluate this layer
- Can an engineer define X from a single search result?
- How many false edges does automated extraction introduce?
- Does a stale edge survive a code change?
- Can a reviewer locate the exact evidence behind each edge?
- Does exact plus keyword retrieval beat embedding for code symbols?
- Can metadata/index changes stay current without model calls?
- Does retrieval respect each user's source permissions?

Do not measure success by the number of nodes or embeddings. Measure **accuracy, evidence coverage, and avoided investigation effort per dollar**.
