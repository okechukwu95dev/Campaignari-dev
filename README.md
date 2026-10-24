# Operational Intelligence & AI Incident Investigation
**Public, anonymized research workspace | Separate research branch**

This branch holds the comprehensive research report and a ready-to-read discovery assignment for an authorized internal agent. It does **not** describe any named company's actual systems. It contains no internal code, customer data, private operational details or deployment instructions tied to a real environment.

## Start here

1. **[Full Deep Research report](docs/DEEP_RESEARCH_REPORT.md)** — The most recent long-form research report (October 24, 2026), including the AI SRE landscape, four architecture approaches, knowledge indexing, identity and dependency mapping, labeling, freshness, hypothesis-based investigation, live explainable UI, permissions, isolated reproduction, model token costs, and implementation feasibility.
2. **[Internal-agent handoff](AGENT_HANDOFF.md)** — The actual discovery assignment extracted from that report. It asks an authorized internal agent to discover the real system, existing tools and limits before making any architecture choices.
3. **[Multi-agent operations workbench](docs/MULTI_AGENT_OPERATIONS_WORKBENCH.md)** — Newer requirement: a Claude-first specialist-agent lobby, sharing evidence rather than full transcripts, VS Code/Claude/Copilot interface options, corporate permissions and token-cost controls. Read this **in addition to** the initial research and handoff; its design is not yet verified internally.
4. **[Knowledge model notes](docs/01-knowledge-model.md)** — Supplementary preliminary design notes for entity/relationship/evidence schemas. These were drafted **before** the latest Deep Research report and should not override it.

## Core objective

Build toward an **evidence-grounded operational investigation platform**: identify what a reported entity means; determine supported upstream/downstream dependencies and business impact; selectively investigate code, runtime events, changes, and prior incidents with low token usage; display the developing evidence graph, hypotheses and tool activity in a human-visible workspace; and learn only verified reusable facts and procedures.

The research does not assume a particular company, business domain, technology stack, permission model or code structure. Architecture options are **alternatives to evaluate**, not mandates.

## First assignment

The authorized internal agent should:
- **Read the two main documents** and inventory what sources, code, runtime evidence and permissions actually exist.
- Use only approved, read-only discovery and **one narrow, safely authorized** operational case or synthetic example.
- Produce a source-backed entity definition and upstream/downstream map, distinguishing observed, verified, inferred, stale, contradictory and unknown relationships.
- Compare architecture approaches and quantify tokens / queries / execution cost on representative incidents.
- Assess historical labeling and evaluation, incremental freshness, safe lower-environment testing and human-visible progress.
- Recommend the smallest realistic prototype **after** discovery; do not select a graph database, vector database, LLM framework, model, or cloud infrastructure in advance.

## Security and separation

**This repository is a public, read-only reference for the internal agent.** The agent must never publish or commit private findings, internal identifiers, documents, source code, logs, secrets, architecture, incident traces, screenshots or proprietary summaries here. It must comply with its organization's data handling policies and keep all generated work in an approved internal destination.

Production access must respect existing permissions; no production writes or destructive tests are authorized by these documents. Test environments are optional and must have separate explicit authorization, resource boundaries, synthetic/sanitized data and isolated side effects.

## Quality and limitations

- The full report is a **research synthesis**, not a verification of a particular internal environment.
- External product examples, announced capabilities and prices are time-sensitive and should be validated against primary sources before procurement or production use.
- Prefer evidence, reliability, inspectability and **useful diagnosis per token**, rather than autonomous spectacle or indexing everything.
- The repository's default **main** branch is not part of this work.
